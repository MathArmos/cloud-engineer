# Agent WhatsApp com regras rigorosas — arquitetura de storage no GCP

> **Contexto profissional:** agent de WhatsApp que atende clientes em nome de uma empresa, precisa seguir **regras de negócio rigorosas** (não pode passar informação errada, precisa respeitar políticas internas, precisa logar tudo pra auditoria).
> **Por que esse cenário é bom para a feira:** toca **vários serviços de storage do GCP de uma vez** (Memorystore, Firestore, Cloud SQL/AlloyDB, BigQuery, Cloud Storage, Bigtable, Vector Search). Defender essa arquitetura em voz alta = mostrar fluência na árvore de decisão do exame.

---

## 1. Anatomia do sistema

Um agent WhatsApp com regras rigorosas tem, no mínimo, estes componentes que precisam de **algum tipo de storage**:

| # | Componente                            | O que é                                                                                  |
|---|---------------------------------------|------------------------------------------------------------------------------------------|
| 1 | **System prompts / instruções base**  | Regras fixas que governam o agent ("não fale sobre X", "sempre confirme Y antes de Z")   |
| 2 | **Knowledge base (regras detalhadas)**| Manual de procedimentos, FAQs, políticas — texto longo que o agent consulta via RAG      |
| 3 | **Histórico de conversa (curto prazo)** | Últimas N mensagens da conversa ativa (contexto pro próximo turno do LLM)              |
| 4 | **Histórico de conversa (longo prazo)** | Todas as conversas já fechadas — para auditoria, métricas, treino futuro               |
| 5 | **Mídia recebida**                    | Imagens, áudios, PDFs que o cliente envia                                                |
| 6 | **Cache de respostas / embeddings**   | Respostas frequentes, embeddings já computados — pra reduzir latência e custo de LLM     |
| 7 | **Traces / eventos do agent**         | Cada tool call, cada decisão, cada flag de qualidade — para debug e auditoria forense    |
| 8 | **Dados operacionais (clientes, sessões, billing)** | Quem é o cliente, qual plano, quantas conversas no mês                       |

Cada um tem padrão de acesso, volume e latência **diferentes**. Forçar tudo no mesmo storage = ou caro demais, ou lento demais, ou as duas coisas.

---

## 2. Mapeamento por componente

### 2.1 — System prompts / instruções base

**Características:**
- Volume pequeno (dezenas a poucas centenas de prompts).
- Mudança rara (toda vez que evolui o produto ou descobre um bug).
- Cada request do agent **lê** o prompt aplicável → leitura **altíssima frequência**, escrita rara.
- Precisa ser rápido (toda mensagem do cliente espera esse lookup antes do LLM rodar).

**Storage recomendado:** **Memorystore Redis** com fonte de verdade em **Firestore** ou **Cloud SQL**.

**Por quê:**
- Memorystore devolve em sub-ms → não vira gargalo no caminho crítico.
- Firestore/Cloud SQL guarda a versão "oficial" (com versionamento, metadados, multi-tenant).
- No deploy, o app puxa de Firestore → carrega em Redis. Quando o prompt muda, publica em Pub/Sub → todos os pods recarregam.

**Por que NÃO Bigtable:** Bigtable cobra ~$0.65/hora por node mesmo ocioso, mínimo 1 node em dev e 3 em prod. Pra guardar 50 prompts? Caro e overkill.

**Por que NÃO só Cloud Storage:** funciona como source of truth versionado (prompts/v3/atendimento.md), mas latência de GET é dezenas de ms — não é cache. Combinaria bem **com** Memorystore.

**Exemplo de comandos:**
```bash
# Memorystore Redis (Basic tier, 1 GB, region us-central1)
gcloud redis instances create prompt-cache \
  --size=1 \
  --region=us-central1 \
  --tier=basic \
  --redis-version=redis_7_0

# Firestore — habilita native mode no projeto (uma vez por projeto)
gcloud firestore databases create --location=us-central1
```

---

### 2.2 — Knowledge base (regras detalhadas, FAQs, políticas)

**Características:**
- Volume médio (centenas a milhares de chunks de texto, podendo crescer).
- Padrão de busca **semântico** (cliente pergunta "posso cancelar?", precisa achar o procedimento de cancelamento mesmo que o texto use "rescisão").
- Latência aceitável: dezenas a centenas de ms (é só uma etapa do RAG).

**Storage recomendado:** **Vector database**. As opções no GCP:

| Opção                                | Quando faz sentido                                              |
|--------------------------------------|----------------------------------------------------------------|
| **AlloyDB com `pgvector`**           | Se já vai usar AlloyDB para dados operacionais (HTAP)          |
| **Cloud SQL Postgres com `pgvector`**| Stack simples, volume baixo-médio, custo controlado            |
| **Vertex AI Vector Search**          | Volume grande (milhões de vetores), latência muito baixa, gerenciado puro |

**Por que NÃO Bigtable ou Firestore puros:** eles não fazem **similarity search** (cosine distance / dot product) nativamente. Você teria que implementar o índice à mão (HNSW, IVF) — antipattern.

**Por que NÃO BigQuery:** BigQuery tem funções de vetor, mas a latência (segundos) não cabe no caminho crítico de um WhatsApp em tempo real. Serve pra **construir o índice offline**, não pra servir.

**Exemplo conceitual:**
```sql
-- AlloyDB / Cloud SQL Postgres com pgvector
CREATE EXTENSION vector;

CREATE TABLE kb_chunks (
  id        bigserial PRIMARY KEY,
  source    text,
  content   text,
  embedding vector(768)
);

-- Lookup do top-5 mais parecido com a pergunta do cliente
SELECT content
FROM kb_chunks
ORDER BY embedding <=> $1   -- $1 = embedding da pergunta
LIMIT 5;
```

---

### 2.3 — Histórico de conversa (curto prazo)

**Características:**
- Por sessão / por número de telefone.
- Volume da janela ativa: 5 a 50 mensagens.
- Read e write a cada turno da conversa.
- Latência crítica (caminho quente do LLM).

**Storage recomendado:** **Memorystore Redis** (com TTL) ou **Firestore** (subcoleção por sessão).

**Quando preferir Redis:**
- Conversas curtas, alto volume, latência máxima.
- Você tolera perder o histórico se o Redis cair (idealmente não, mas como cache de janela ele é ok).

**Quando preferir Firestore:**
- Quer durabilidade garantida sem se preocupar com cache invalidation.
- Sessões podem durar horas/dias.
- Quer queries por outros campos (ex.: "conversas abertas com flag urgente").

**Anti-padrão:** Bigtable para isso. Bigtable é incrível para histórico **longo**, não para janela ativa.

---

### 2.4 — Histórico de conversa (longo prazo)

**Características:**
- Append-only depois que a conversa fecha.
- Volume crescente, indefinido (todas as conversas de todos os clientes).
- Padrão de acesso:
  - **Operacional / forense:** "me mostre a conversa do número X em D dia" → lookup por chave.
  - **Analítico:** "quantas conversas violaram a regra Y este mês?" → SQL ad-hoc em milhões de linhas.

**Storage recomendado:** **duas camadas**:

1. **Bigtable** para acesso operacional rápido por `phone_number#timestamp_invertido`.
2. **BigQuery** para analytics — sincronizado a partir do Bigtable (ou direto via Pub/Sub → BigQuery).

**Quando você ainda NÃO precisa de Bigtable:**
- Volume baixo (até dezenas de milhares de mensagens/dia): **Cloud SQL Postgres** com índice em `(phone_number, created_at)` resolve.
- A regra prática: só pula pra Bigtable quando Postgres começar a sofrer com índices grandes ou storage.

**Por que BigQuery aqui é decisivo:**
- Analytics de qualidade do agent ("% de respostas que violaram regra X").
- Dataset de avaliação / fine-tuning ("me dá 10k conversas onde o agent foi corrigido").
- Dashboards de operação (Looker Studio em cima do BigQuery).

**Pipeline típico:**
```
WhatsApp → app → Pub/Sub → Dataflow → {Bigtable (operacional), BigQuery (analytics)}
                                  ↘ Cloud Storage (raw archive)
```

---

### 2.5 — Mídia recebida (imagens, áudios, PDFs)

**Características:**
- Blob binário.
- Imutável depois de gravado.
- Acesso esporádico (operador abre pra ver, ou modelo de visão processa).

**Storage recomendado:** **Cloud Storage** — sem discussão. É o caso textbook.

**Storage class:**
- **Standard** nos primeiros 30 dias (cliente / operador ainda pode revisar).
- **Lifecycle policy** movendo pra **Nearline** após 30 dias e **Coldline / Archive** após 90/365 dias.
- Custo despenca depois do hot period.

**Exemplo:**
```bash
gcloud storage buckets create gs://wpp-media-prod \
  --location=us-central1 \
  --default-storage-class=STANDARD \
  --uniform-bucket-level-access

# Lifecycle: depois de 30 dias → Nearline, 90 → Coldline, 365 → Archive
gcloud storage buckets update gs://wpp-media-prod \
  --lifecycle-file=lifecycle.json
```

**Por que NÃO Filestore:** o app acessa por API HTTP, não precisa de POSIX. Filestore aqui seria pagar caro por feature que ninguém usa.

---

### 2.6 — Cache de respostas / embeddings

**Características:**
- Embeddings de perguntas frequentes (custa caro recomputar a cada request).
- Respostas pré-computadas para FAQs ("aceita Pix?" → texto pronto).
- Lookups sub-ms.

**Storage recomendado:** **Memorystore Redis**.

**Padrão de implementação:**
- Chave: `embedding:<hash_da_pergunta>` → valor: vetor serializado.
- TTL longo (dias) — embedding de "como cancelo?" não muda toda hora.

**Ganho concreto:** 1 chamada à API de embeddings custa ~$0.0001 e leva ~50ms. Cache em Redis = grátis e <1ms. Em escala, é a diferença entre uma conta de US$ 500 e US$ 50.000 por mês.

---

### 2.7 — Traces / eventos do agent

**Características:**
- Cada interação gera N eventos (tool calls, decisões, flags de qualidade, custos do LLM).
- Volume **muito** maior que o de mensagens (10-50x).
- Padrão de acesso:
  - **Debug agora:** "me dá o trace completo da sessão X" — lookup por chave.
  - **Analytics:** "qual % de sessões usou a tool Y este mês?" — SQL.

**Storage recomendado:**
- **Cloud Logging** primeiro (vem de graça, fácil de buscar com Log Explorer) — bom até certa escala.
- Acima de um milhão de eventos/dia → exportar pra **BigQuery** (sink do Logging) e usar **Bigtable** se o lookup por session_id virar gargalo.

**Não invente:** comece com Cloud Logging. Só monte Bigtable+BigQuery quando o Logging começar a doer no bolso ou na busca.

---

### 2.8 — Dados operacionais (clientes, planos, billing, sessões abertas)

**Características:**
- Relacional clássico (clientes, planos, faturas, integridade referencial).
- Transacional (criar cliente, atualizar plano, registrar pagamento — precisa de ACID).
- Volume moderado (mesmo grandes operações têm "só" milhões de clientes, não bilhões).

**Storage recomendado:** **Cloud SQL Postgres** (default), **AlloyDB** (se quiser BI em tempo real sobre operações).

**Por que NÃO Spanner:** só se a operação for **global** com requisitos de consistência transcontinental — não é o caso típico de uma empresa brasileira atendendo WhatsApp.

**Por que NÃO Firestore:** funciona, mas você perde joins, integridade referencial e SQL — coisas que um sistema operacional realmente usa.

---

## 3. Arquitetura consolidada

```
                          +--------------------+
                          |  WhatsApp / Twilio |
                          +---------+----------+
                                    |
                                    v
                          +-------------------+
                          |   Backend (Run /  |
                          |   GKE / GCE)      |
                          +---+----+----+-----+
                              |    |    |
       Cache (sub-ms)  <------+    |    +------> Source of truth
       Memorystore Redis           |             Cloud SQL Postgres
       - system prompts            |             - clientes / planos
       - janela conversa           |             - sessões abertas
       - embeddings cache          |             - billing
                                   |
                                   v
                          +-----------------+
                          |  Vector DB      |
                          |  pgvector ou    |
                          |  Vector Search  |  ← RAG da knowledge base
                          +-----------------+
                                   |
                                   v
                       Pub/Sub (eventos do agent)
                                   |
                  +----------------+----------------+
                  |                |                |
                  v                v                v
          +-------------+   +-------------+   +-------------+
          |  Bigtable   |   |  BigQuery   |   | Cloud       |
          |  (operac.)  |   |  (analytics)|   | Storage     |
          |  histórico  |   |  histórico  |   | (mídia +    |
          |  por phone  |   |  + traces   |   | raw archive)|
          +-------------+   +-------------+   +-------------+
```

---

## 4. Evolução por escala (tier 1 → tier 3)

A pior coisa que se faz num projeto novo é montar essa arquitetura toda no dia 1. Use **tiers**:

### Tier 1 — MVP (até ~10k mensagens/dia)

**Stack mínima:**
- **Cloud SQL Postgres** (dados operacionais + histórico de conversa + system prompts numa tabelinha)
- **Cloud Storage** (mídia)
- **Cloud Logging** (traces)
- **Vertex AI** (LLM)

**Por que funciona:** Postgres aguenta tudo isso fácil. Você ainda não tem volume nem latência crítica que justifique mais de um banco.

### Tier 2 — Crescimento (10k – 1M mensagens/dia)

**Adiciona:**
- **Memorystore Redis** (cache de prompts, janela de conversa, embeddings)
- **`pgvector` no Postgres** (RAG da knowledge base)
- **BigQuery** (analytics — sink do Logging + dump do Postgres)

**Por que essa ordem:** Redis dá ganho imediato de latência e custo (cache de LLM). BigQuery dá insight de qualidade. Vector search resolve o RAG sem virar gerenciamento extra.

### Tier 3 — Escala alta (>1M mensagens/dia, multi-cliente, multi-região)

**Adiciona / migra:**
- **Bigtable** (histórico operacional — Postgres já sofreu).
- **AlloyDB** (substitui Cloud SQL se quiser HTAP forte sem ETL).
- **Vertex AI Vector Search** (substitui pgvector quando os índices passam de alguns milhões).
- **Spanner** apenas se a operação cruzar fronteiras e exigir SLA de consistência multi-região.

---

## 5. Armadilhas comuns nesse design

1. **Pôr tudo em Postgres "porque é fácil".** Funciona no Tier 1, quebra no Tier 2. Cache em Redis é low-hanging fruit antes de qualquer otimização.
2. **Pôr system prompts em Bigtable.** Overkill — Bigtable é petabyte-scale. Use Firestore + Memorystore.
3. **Usar BigQuery como source of truth do agent.** BigQuery tem DML, mas o cache embutido + latência de segundos fazem dele péssimo para o caminho quente.
4. **Esquecer lifecycle policy no Cloud Storage.** Mídia velha em Standard custa 10x o necessário.
5. **Pular Cloud Logging direto pra Bigtable.** Logging é mais barato e gerenciado pro volume inicial. Bigtable só quando o Logging doer.
6. **Confiar em Memorystore como source of truth.** Redis sem persistência é volátil. Cache nunca é a verdade.
7. **Pôr embeddings em Firestore.** Sem suporte nativo a similarity search → você reimplementa um vector DB ruim. Use pgvector ou Vector Search.
8. **Sincronizar Postgres ↔ BigQuery na mão.** Use Datastream (CDC gerenciado) ou Dataflow templates. ETL artesanal envelhece mal.

---

## 6. Perguntas de feira + respostas prontas

### "Como você arquitetaria um agent multi-cliente no GCP?"

> Hoje no MVP eu rodaria em Postgres + Cloud Storage + Vertex AI. Conforme cresce, separo cache pra Memorystore Redis, knowledge base pra `pgvector`, e analytics pra BigQuery via Datastream. Bigtable só entra quando o histórico operacional em Postgres começar a sofrer — geralmente acima de algumas dezenas de milhões de mensagens. Spanner eu não usaria a menos que precisasse de consistência global.

### "Por que não tudo em BigQuery?"

> BigQuery é warehouse analítico — funciona ótimo com dado que não muda muito e queries SQL ad-hoc. Mas tem cache embutido e latência de segundos, então não serve pro caminho crítico de uma mensagem em tempo real. Eu uso BigQuery pra avaliação do agent e construção de dataset de treino, não pra servir.

### "Por que pgvector ao invés de Vertex AI Vector Search?"

> Custo e simplicidade no início. Se eu já tenho Postgres no stack, adicionar `pgvector` é uma extension — sem mais um serviço pra operar. Migro pra Vector Search quando o volume de embeddings ou a latência de busca justificar.

### "Onde guardaria os prompts do agent?"

> Source of truth em Firestore (versionado, multi-tenant, baixa latência por documento). Cache de leitura em Memorystore Redis pra zerar latência no caminho quente. Bigtable ou BigQuery aqui seriam exagero — o volume é pequeno e o padrão de acesso é lookup por chave única.

### "Como auditaria uma decisão errada do agent?"

> Cada tool call e decisão vai pro Cloud Logging com `session_id` e `trace_id`. Logs são exportados pro BigQuery via sink. Pra debug operacional rápido eu busco no Log Explorer; pra análise histórica (% de sessões com erro X) eu rodo SQL no BigQuery. Em escala muito alta eu colocaria também um índice no Bigtable por `session_id#timestamp` pra reconstrução de trace sub-segundo.

---

## 7. Conexão com a árvore de decisão do exame

Cada componente acima exercita um ramo da árvore:

| Componente            | Ramo da árvore ACE                                                              |
|-----------------------|--------------------------------------------------------------------------------|
| System prompts        | Estruturado → não analítico → não relacional → **precisa de cache** → Memorystore |
| Knowledge base (RAG)  | Estruturado → analítico operacional → **relacional + extensão** → AlloyDB/Cloud SQL pgvector |
| Histórico curto       | Estruturado → não analítico → não relacional → cache OK → **Memorystore ou Firestore** |
| Histórico longo (op.) | Estruturado → analítico → **baixa latência + alto throughput** → Bigtable      |
| Histórico longo (BI)  | Estruturado → analítico → **SQL ad-hoc + dado estável** → BigQuery             |
| Mídia                 | **Não-estruturado** → não precisa de NFS → Cloud Storage                       |
| Cache de embedding    | Estruturado → não analítico → não relacional → **caching** → Memorystore       |
| Traces do agent       | Estruturado → analítico → SQL ad-hoc → BigQuery (sink do Logging)              |
| Dados operacionais    | Estruturado → não analítico → relacional → **sem HTAP, sem global** → Cloud SQL |

Sabendo defender essa tabela em voz alta, você cobre **toda** a árvore do M3 do Core Services e fica fluente para qualquer DQ de planejamento que cair.
