# Curso: Essential Google Cloud Infrastructure: Core Services

> **Duração:** 8h 15min · **Status:** 🟡 em andamento  
> **Seções da prova cobertas:** 1.1 (IAM), 3.4 (Storage/DB), 4.1 (GCE mgmt), 4.4 (Storage mgmt), 4.6 (Monitoring)

---

## Módulo 1 — Cloud IAM

### 1.1 — Introdução ao IAM

**IAM (Identity and Access Management)** é o sistema do Google Cloud para controlar *quem* pode fazer *o quê* em *quais recursos*.

**Componentes do IAM:**

| Componente | Descrição |
|---|---|
| **Organizations** | Nó raiz da hierarquia de recursos |
| **Roles** | Coleção de permissões agrupadas por "tipo de trabalho" |
| **Members** | Identidades que recebem roles (usuários, grupos, service accounts) |
| **Service Accounts** | Identidades para aplicações/VMs, não para humanos |

**Destaques do IAM no GCP:**
- Identidades usam formato de **endereço de e-mail**
- Roles são baseadas em **tipo de função** (job-type roles), não permissões avulsas
- Permissões são **granulares** — maior controle com menor superfície de ataque
- Inclui o recurso **Organization Restrictions** para limitar escopo de acesso

> **Tópicos do módulo:** Organizations → Roles → Members → Service Accounts → Organization Restrictions → Best Practices → Lab

---

### 1.2 — O que é IAM? + Resource Hierarchy

**Definição central:**

> **Quem** pode fazer **o quê** em **qual recurso**

| Elemento | Descrição | Exemplo |
|---|---|---|
| **Quem** (Who) | Pessoa, grupo ou aplicação | `user@example.com` |
| **O quê** (What) | Privilégios ou ações específicas | Role: `Compute Viewer` |
| **Qual recurso** (Which) | Qualquer serviço do GCP | Compute Engine, Cloud Storage... |

**Exemplo prático:** Role `Compute Viewer` → acesso read-only para get/list de recursos Compute Engine, **sem** acesso aos dados armazenados neles.

---

#### Hierarquia de Recursos do IAM

```
Organization  (example.com)          ← nó raiz
    └── Folders  (Dept X, Dept Y)     ← representa departamentos
            └── Projects              ← trust boundary
                    └── Resources     ← instâncias, buckets, topics...
```

**Regras críticas:**

| Regra | Detalhe |
|---|---|
| Cada recurso tem **exatamente 1 pai** | Sem herança múltipla |
| Policies são **herdadas para baixo** | Role no org → vale para tudo abaixo |
| **Organization** = sua empresa | Roles aqui valem para todos os recursos |
| **Folder** = departamento | Roles aqui valem para tudo dentro da pasta |
| **Project** = trust boundary | Serviços no mesmo projeto têm o mesmo nível de confiança padrão |

> **Pegadinha de prova:** herança é sempre **top-down** — um role concedido num nível superior não pode ser negado num nível inferior.

---

### 1.3 — Organization Node e Folders

#### Organization Node

O nó raiz da hierarquia. É provisionado automaticamente quando um usuário com conta **Google Workspace** ou **Cloud Identity** cria um projeto GCP.

**Roles principais na Organization:**

| Role | Quem recebe | Responsabilidades |
|---|---|---|
| **Org Admin** | Bob (admin de TI) | Define IAM policies, estrutura da hierarquia, delega billing/networking via roles |
| **Project Creator** | Alice | Cria projetos dentro da org (pode ser aplicado no nível da org e herdado) |
| **Org Viewer** | Auditores | View-only em todos os recursos da org |

**Dois perfis críticos na criação da org:**

| Perfil | Quem é | Responsabilidades |
|---|---|---|
| **Workspace/Cloud Identity Super Admin** | Admin do domínio | Atribui o Org Admin role; ponto de contato para recovery; controla lifecycle da conta |
| **GCP Organization Admin** | Admin do GCP | Define IAM policies; determina estrutura da hierarquia; delega roles críticos |

> **Importante:** Os dois roles são geralmente atribuídos a **pessoas diferentes**. O Org Admin, por princípio de least privilege, **não inclui** permissão para criar folders — precisa atribuir roles adicionais à própria conta.

---

#### Folders

Folders são **sub-organizações** dentro da org — mecanismo de agrupamento e isolamento entre projetos.

**Usos típicos:**

| Nível de Folder | Representa |
|---|---|
| 1º nível | Departamentos (Dept X, Dept Y, Shared Infrastructure) |
| 2º nível | Times dentro do departamento (Team A, Team B) |
| 3º nível | Produtos/aplicações (Product 1, Product 2) |

**Roles de Folder:**

| Role | O que faz |
|---|---|
| **Folder Admin** | Controle total sobre folders |
| **Folder Creator** | Navega na hierarquia e cria folders |
| **Folder Viewer** | Visualiza folders e projetos abaixo do recurso |

**Roles de Project (nível resource manager):**

| Role | O que faz |
|---|---|
| **Project Creator** | Cria projetos → torna-se automaticamente **Owner** do projeto |
| **Project Deleter** | Permissão para deletar projetos |

> **Delegação via Folders:** o chefe de um departamento pode receber full ownership de todos os recursos dentro da sua folder. Usuários de um departamento só acessam recursos dentro da sua folder — isolamento por design.

---

## Módulo 2 — Storage and Database Services

<!-- colar conteúdo aqui -->

---

## Módulo 3 — Resource Management

> **Status:** 🟡 em andamento · **Iniciado em:** 2026-05-14
> **Fontes:** Vídeos "Resource Manager: Projects, Folders, and Organizations" + "Quotas" + "Labels" + "Billing"
> **Subseções da prova tocadas:** 1.1 (resource hierarchy), 1.2 (billing/labels/budgets), 4.x (quotas operacionais)

### 3.1 — Resource Manager: visão geral

O **Resource Manager** é o serviço que permite **gerenciar recursos hierarquicamente** por organização → folder → projeto → recurso. Já vimos a hierarquia no Módulo 1 (IAM); aqui o foco é entender como **policies fluem para baixo** e como **billing flui para cima** ao mesmo tempo — dois fluxos opostos passando pela mesma árvore.

#### Dois fluxos opostos na hierarquia

```
                    Organization
                         │
                    ┌────┴────┐
                    │ Folder  │
                    └────┬────┘
                         │
                    ┌────┴────┐
                    │ Project │   ← projeto = ponto de junção
                    └────┬────┘     dos dois fluxos
                         │
                    ┌────┴────┐
                    │Resource │
                    └─────────┘

  Policies IAM  ↓ (herdam de cima pra baixo)
  Billing       ↑ (acumula de baixo pra cima)
```

**Por que importa.** O projeto é a fronteira de tudo: ele **herda** policies da org/folder acima e **acumula** o consumo de todos os recursos abaixo. Quando alguém pergunta "onde isso é cobrado?", a resposta é sempre o projeto onde o recurso vive. Quando alguém pergunta "quem pode mexer nisso?", você sobe a árvore até onde a role foi concedida.

#### IAM Allow Policy vs IAM Deny Policy (novidade)

Já sabemos que policies têm **roles + members** atribuídos a recursos, e que recursos herdam policies do pai. O detalhe novo dos dois vídeos:

| Tipo de policy | O que faz | Como combina com hierarquia |
|----------------|-----------|-----------------------------|
| **Allow policy** | Concede roles a principais sobre recurso | **União** com policies herdadas — aditiva. Se a org dá `roles/storage.admin` ao grupo X, o projeto recebe essa role também. |
| **Deny policy** | Bloqueia permissões específicas a principais | **Prevalece** sobre allow — bloqueia mesmo se o principal tem a role. |

**Exemplo concreto.** Você quer dar `roles/owner` à Alice em todo o projeto, mas **nunca** quer que ela delete buckets de Cloud Storage. Fluxo:
1. Allow policy no projeto: `Alice → roles/owner`
2. Deny policy no projeto: bloqueia permissão `storage.buckets.delete` para Alice

Ela mantém todo o resto do Owner mas perde aquele dedo específico. Sem deny policy, você teria que sair do `roles/owner` e construir uma role customizada sem essa permissão — muito mais trabalho.

**Armadilha de prova.** Em pergunta de "como impedir um Owner de fazer X sem tirar o Owner role?", **deny policy** é a resposta. É mais granular que mexer no allow.

#### Billing acumula de baixo pra cima

Vimos no Módulo 1 que uma billing account está ligada a um projeto. O detalhe importante aqui é o **fluxo**:

```
Resource (VM, bucket, query)  →  cobra consumo
        ↓
Project (acumula consumo de TODOS seus resources)  →  envia para billing
        ↓
Billing Account (vinculada ao projeto)  →  fatura
```

**O consumo é medido em.**
- **Quantidades** (GB armazenados, número de requests)
- **Rate of use** (CPU-hours, requests/s)
- **Tempo** (segundos de execução de função, horas de VM ligada)
- **Número de itens** (queries no BigQuery, mensagens no Pub/Sub)
- **Feature use** (uso de feature paga específica, como Cloud Spanner com backups)

**Regras críticas.**
- Um **recurso pertence a exatamente 1 projeto**
- Um **projeto pertence a exatamente 1 billing account**
- Logo, uma **organization contém todas as billing accounts** dela
- Você **NÃO consegue dividir o custo de UM recurso entre dois projetos** — se quer cobrança separada, separe os projetos

**Por que importa.** Pra atribuir custo (chargeback), você desenha a hierarquia primeiro. Cymbal Superstore separa `ecommerce-dev`, `ecommerce-staging`, `ecommerce-prod` justamente pra cada ambiente ter seu próprio rollup de billing — sem isso, time de finanças não consegue separar "quanto custa staging" sem labels e queries no BigQuery.

#### Para que serve um projeto (recap consolidado)

Um projeto é o que permite:
- **Enable billing** (linkar a uma billing account)
- **Gerenciar permissions e credentials** (IAM)
- **Habilitar services e APIs** (Cloud Storage, BigQuery, etc.)
- **Rastrear consumo e quotas**

Toda interação com a API do GCP exige identificar um projeto. Os 3 identificadores (já vistos):
- **Project Name** — humano, mutável, não usado pelas APIs
- **Project Number** — gerado pelo Google, imutável, único globalmente
- **Project ID** — derivado do Name na criação, imutável, único globalmente, é o que vai em URLs e configs

Você acha esses 3 no dashboard do Console ou via `gcloud projects describe PROJECT_ID`.

#### Classificação física dos recursos: Global, Regional, Zonal

**Conceito novo importante.** Independente da hierarquia lógica (org/folder/project), cada recurso tem um **escopo físico** que afeta onde ele vive e como você o referencia:

| Escopo | O que significa | Exemplos | Implicação prática |
|--------|-----------------|----------|---------------------|
| **Global** | Recurso visível em qualquer região, sem afinidade geográfica | **Images**, **snapshots**, **VPC networks**, custom roles, IAM policies | Você cria 1 vez, usa de qualquer lugar. Snapshot tirado em `us-east1` pode restaurar em `europe-west1`. |
| **Regional** | Recurso vive em 1 região (várias zonas dentro), sobrevive à queda de 1 zona | **External IP addresses (estáticos)**, regional persistent disks, subnets, regional managed instance groups, Cloud SQL regional | Acessível por qualquer recurso na região. Falha de zona não derruba. |
| **Zonal** | Recurso vive em 1 zona específica, morre se a zona cair | **Instances (VMs)**, **persistent disks zonal**, local SSDs, GKE nodes (em cluster zonal) | Acessível apenas naquela zona. Fundamental considerar pra HA. |

**Por que importa.**
- Recursos zonais → você sozinho é responsável por replicar entre zonas pra HA (MIG, regional disk, regional Cloud SQL)
- Recursos globais → como snapshots e imagens, são acessíveis de qualquer lugar — facilita criar VM em outra região a partir de snapshot
- Recursos regionais → "no meio do caminho" — alta disponibilidade local, mas se a região cair (raro mas acontece), você precisou já ter cross-region

**Exemplos de comandos pra reforçar o escopo:**
```bash
# Recurso global — sem flag de região/zona
gcloud compute snapshots list
gcloud compute images list
gcloud compute networks list

# Recurso regional — precisa --region
gcloud compute addresses list --filter="region:us-central1"
gcloud compute addresses create my-ip --region=us-central1

# Recurso zonal — precisa --zone
gcloud compute instances list --filter="zone:us-central1-a"
gcloud compute instances create my-vm --zone=us-central1-a
```

**Armadilha de prova.**
- "External IP address" — costuma confundir: o **estático/reservado** é **regional**, o **ephemeral** vem com a VM e é **zonal** (some quando a VM morre)
- Persistent disk pode ser **zonal** (padrão) ou **regional** (HA cross-zone, mais caro)
- VPC network é **global** (subnets é que são regionais)

---

### 3.2 — Quotas (tópico novo)

**O quê.** Toda criação de recurso no GCP está sujeita a **quotas/limites por projeto**. Quotas existem como guarda-rails: protegem você (de gastar demais por erro) e protegem o Google (de um projeto consumir recursos infinitos).

#### As 3 categorias de quota

| Categoria | O que limita | Exemplo |
|-----------|--------------|---------|
| **Quantos recursos por projeto** | Quantidade máxima de objetos criados | **15 VPC networks por projeto** (default) |
| **Rate limit de API** | Quão rápido você pode chamar uma API no projeto | **5 ações administrativas/segundo** via API do Spanner |
| **Quota regional** | Recursos disponíveis dentro de uma região específica | **24 CPUs por região** (default em região nova) |

**Por que três categorias.** Cada uma protege contra um padrão de abuso diferente:
- **Por projeto** → evita projeto fazendo "explosão" de objetos (10000 firewalls)
- **Rate limit** → evita DDoS interno no plano de controle (criar/destruir 1000 VMs/s)
- **Regional** → evita um projeto monopolizar capacidade física de uma região

#### Por que quotas existem (lista canônica da prova)

1. **Prevenir runaway consumption** em caso de **erro ou ataque malicioso**
   - Exemplo do vídeo: você roda `gcloud` em loop e em vez de 10 VMs cria 100 sem perceber → quota corta antes
2. **Prevenir spikes/surpresas de billing**
   - Mesmo mecanismo: limita até onde a conta pode crescer rapidamente
3. **Forçar sizing consideration e revisão periódica**
   - "Você precisa MESMO de uma VM de 96 cores ou um n2-standard-4 resolve?"

#### Como aumentar quota

Como ACE, você normalmente **não tem permissão pra aprovar aumentos** — esse fluxo é do Org Admin ou de quem tem `roles/serviceusage.quotaAdmin`. Mas você **identifica** quando precisa subir e **submete o request**.

**Fluxo padrão:**
1. Vá em **IAM & Admin → Quotas** no Console
2. Filtre por serviço/região/métrica
3. Selecione a quota e clique **Edit Quotas**
4. Justifique (usuário previsto, motivo do crescimento, prazo)
5. Aguarda aprovação do Google Cloud (pode levar 1-2 dias úteis)

**Via gcloud:** atualmente o aumento manual é por Console + API; não tem comando `gcloud` simples e direto pra "raise quota" (você consulta com `gcloud compute project-info describe` e usa a API de quotas pra requisitar).

**Comandos pra inspecionar quota:**
```bash
# Quotas de projeto inteiro (CPUs globais, IPs, networks, etc.)
gcloud compute project-info describe --project=PROJECT_ID

# Quotas por região (CPUs regionais, IPs regionais, etc.)
gcloud compute regions describe us-central1

# Quotas específicas via API
gcloud alpha services quota list --service=compute.googleapis.com \
  --consumer=projects/PROJECT_ID
```

#### Armadilhas de prova críticas

1. **Quota é MÁXIMO, não GARANTIA.**
   Tem quota de 100 local SSDs em `us-central1`? Ótimo — mas se a região estiver sem capacidade física de local SSD nesse momento, **você não consegue criar** mesmo dentro da quota. Quota = teto autorizado; capacidade = teto físico. Os dois precisam estar atendidos.

2. **Quotas são por PROJETO, não por usuário.**
   Quotas vivem no nível do projeto. Não importa se 1 usuário ou 50 estão criando recursos no mesmo projeto — o teto é o mesmo.

3. **Aumento de quota não é instantâneo.**
   Em pergunta de prova tipo "preciso de 200 CPUs amanhã, o que faço?" — a resposta inclui **abrir o request com antecedência** porque a aprovação leva tempo.

4. **Quota ≠ Limit de billing.**
   Quota limita **quantos recursos** você pode ter. Budget alert/cap limita **quanto você pode gastar**. São controles diferentes que se complementam — quota previne o caso "1000 VMs", budget previne o caso "10 VMs caras esquecidas ligadas o mês inteiro".

5. **Default quotas variam por idade da conta.**
   Conta nova (trial) tem quotas baixas (8 CPUs em alguns lugares). Conta paga estabelecida sobe automaticamente com o tempo conforme uso real.

#### Cenário real

Você lança uma promoção sexta-feira que prevê 10x o tráfego normal. Sua aplicação roda em GKE com autoscaler configurado pra subir até 50 nodes. **Antes de sexta, você deve:**
- Conferir quota de CPUs na região (cabem 50 nodes × 4 CPUs = 200 CPUs?)
- Conferir quota de IPs externos se cada node tem IP público
- Conferir quota de persistent disks
- Se faltar quota, abrir request com 2-3 dias de antecedência

Sem essa checagem, autoscaler tenta subir node, bate em quota, GKE para de escalar, app degrada na promoção.

---

### 3.3 — Labels (e a diferença para Network Tags)

**O quê.** Labels são pares **key:value** definidos pelo usuário que você anexa a recursos GCP para **organizar, agrupar e filtrar**. Diferente da hierarquia (Org/Folder/Project) que é estrutural e única, labels são **livres, sobrepostas e múltiplas** — um mesmo recurso pode ter vários labels simultaneamente.

```
VM "ecommerce-app-vm-7":
   environment: production
   team:        marketing
   component:   frontend
   owner:       gaurav
   state:       inuse
```

**Formato.**
- Key e value são strings (key tem regras mais estritas: lowercase, números, hífen, underline, max 63 chars; deve começar com letra)
- Até **64 labels por recurso** (limite GCP)
- Labels **não** carregam permissões — não são mecanismo de segurança

#### Por que importa (4 casos de uso canônicos)

1. **Inventário e filtros** — `gcloud compute instances list --filter="labels.environment=production"` lista só as VMs de prod
2. **Cost accounting / chargeback** — labels **propagam para o billing export do BigQuery**, permitindo agrupar custo por `team`, `component`, `environment`. Esse é o **diferencial mais importante** das labels.
3. **Bulk operations em scripts** — script que faz snapshot de "tudo com `state:inuse`", ou que deleta "tudo com `state:readyfordeletion`"
4. **Automation hooks** — pipelines de CI/CD podem agir sobre recursos com `environment:staging` mas não em `production`

#### Casos típicos de label (lista do vídeo, decorada)

| Padrão | Exemplo de key:value | Para que serve |
|--------|----------------------|----------------|
| **Time ou cost center** | `team:marketing`, `team:research` | Chargeback entre departamentos |
| **Componente** | `component:redis`, `component:frontend` | Análise de custo por tier da aplicação |
| **Ambiente/estágio** | `environment:production`, `environment:test` | Separar dev/staging/prod sem ter projetos separados |
| **Owner / contato** | `owner:gaurav`, `contact:opm` | Quem é dono / quem chamar quando der pau |
| **Estado** | `state:inuse`, `state:readyfordeletion` | Gating de ações automáticas (snapshot, delete) |

**Decisão chave: project vs label para separar ambientes.**
- Projeto separado por ambiente → isolamento **total** (IAM, billing account, quotas, blast radius)
- Label `environment:` → isolamento **lógico** apenas (mesma billing/IAM/quotas)
- Best practice: para `dev/staging/prod`, use **projetos separados**. Labels servem pra dimensões transversais (team, component, owner) que não justificam projeto separado.

#### Comandos práticos

```bash
# Adicionar labels a uma VM
gcloud compute instances add-labels VM_NAME \
  --labels=environment=production,team=marketing,component=frontend \
  --zone=us-central1-a

# Remover labels
gcloud compute instances remove-labels VM_NAME \
  --labels=state \
  --zone=us-central1-a

# Listar VMs filtrando por label
gcloud compute instances list \
  --filter="labels.environment=production AND labels.team=marketing"

# Atualizar labels (formato chave=valor)
gcloud compute instances update VM_NAME \
  --update-labels=state=inuse \
  --zone=us-central1-a

# Labels em buckets Cloud Storage
gcloud storage buckets update gs://my-bucket --update-labels=team=marketing

# Labels em datasets BigQuery
bq update --set_label team:marketing PROJECT_ID:DATASET
```

**No billing export do BigQuery** as labels aparecem como uma coluna `labels` (array de structs key/value). Query típica de chargeback:

```sql
SELECT
  l.value AS team,
  SUM(cost) AS total_cost
FROM `billing_export.gcp_billing_export_v1_XXXXX`,
  UNNEST(labels) AS l
WHERE l.key = 'team'
  AND invoice.month = '202605'
GROUP BY team
ORDER BY total_cost DESC;
```

#### Labels vs Network Tags — **a confusão clássica de prova**

| Aspecto | **Labels** | **Network Tags** |
|---------|-----------|------------------|
| Formato | **key:value** | **string só** (sem value) |
| Aplicáveis a | Quase **todo recurso GCP** (VMs, disks, buckets, datasets, snapshots...) | **Apenas instâncias** (Compute Engine VMs) |
| Função | **Organizar / filtrar / billing rollup** | **Aplicar firewall rules e routes** com base no tag |
| Propaga em billing? | **SIM** (aparece no export do BigQuery) | **NÃO** |
| Limite | Até **64 labels** por recurso | Até **64 network tags** por instância |
| Exemplo | `environment:production` | `web-server`, `db`, `allow-ssh` |

**Por que existe a distinção.**
- Labels resolvem o problema **organizacional/financeiro** (quem é dono, quanto custou)
- Network tags resolvem o problema **de rede** (que tráfego permitir/rotear)
- Mesmo conceito de "tag livre", mas mecanismos em planos completamente diferentes do GCP

**Exemplo prático de network tag** (pra fixar a diferença):
```bash
# Firewall rule que se aplica a VMs com tag "web-server"
gcloud compute firewall-rules create allow-http-web \
  --network=default \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:80 \
  --target-tags=web-server \
  --source-ranges=0.0.0.0/0

# Aplicar a tag na VM
gcloud compute instances add-tags my-vm \
  --tags=web-server \
  --zone=us-central1-a
```

A partir daí, qualquer VM com a network tag `web-server` permite ingress na porta 80. **Nenhum** desses comandos teria efeito de firewall se usássemos `labels` no lugar de `tags`.

#### Armadilhas de prova críticas

1. **"Como separar custo por time/projeto/componente sem criar projetos diferentes?"** → **Labels** (`team:X`). Network tag NÃO propaga em billing.
2. **"Permitir tráfego HTTP só em servidores web?"** → **Network tags** (`web-server`) + firewall rule com `--target-tags=web-server`. Label NÃO se aplica a firewall.
3. **"Posso aplicar network tag a um bucket de Cloud Storage?"** → **NÃO.** Network tags são **só de instances**. Bucket aceita label, não tag.
4. **Label NÃO é mecanismo de segurança.** Recurso com `environment:production` continua acessível pra quem tem IAM permission — não confundir label com "tag de classificação que restringe acesso".
5. **Key da label é case-sensitive na busca, mas convenção é tudo lowercase.** `Team:marketing` ≠ `team:marketing` em filtros.
6. **Cobrança aparece com atraso no export.** Labels novas só aparecem no BigQuery billing export a partir da próxima atualização do export (geralmente algumas horas). Cuidado em pergunta que sugira efeito imediato.
7. **Mudar label depois do recurso criado é OK.** Labels são metadados mutáveis. Diferente do Project ID/Number, que são imutáveis.

#### Cenário real

Projeto compartilhado entre 3 times (marketing, vendas, eng). Cada time tem suas VMs e buckets, mas tudo paga pela mesma billing account. No fim do mês finanças quer saber "quanto cada time gastou?". Solução:
- Aplicar `team:marketing` / `team:vendas` / `team:eng` em **todo** recurso na criação (mandatório via policy ou pipeline de IaC)
- Habilitar billing export para BigQuery
- Query mensal agrupando por `labels.team`
- Resultado: planilha mensal com custo por time, sem precisar quebrar em 3 projetos

Sem labels → finanças tem que conferir nome do recurso, dono do recurso, ou recorrer a engenharia pra mapear. Insustentável em escala.

---

### 3.4 — Billing, Budgets e o Loop de Otimização de Custo

> **Conteúdo principal vive em [`../01-setup-environment/1.2-billing.md`](../01-setup-environment/1.2-billing.md)** (seções 3, 4 e 6). Aqui ficam só os pontos novos cobertos por este vídeo do curso.

**Novidades cobertas pelo vídeo "Billing" deste módulo:**

1. **Forecasted spend alert** — além do alerta tradicional de gasto real (50%/90%/100%), existe um alerta de **projeção**: dispara quando o GCP estima que, no ritmo atual, o gasto vai estourar o budget **até o fim do período**, mesmo que ainda não tenha estourado. Permite reagir cedo. Configurado junto com threshold rules; é um checkbox separado no Console.

2. **O que o email de alert contém (e o que NÃO contém):** o email tem **project name**, **percent of budget exceeded**, **budget amount**. Ele **NÃO** detalha por SKU/serviço — pra saber "o quê" estourou, ou você vai no Console → Billing → Reports, ou no **BigQuery export**. Pergunta de prova esperta usa essa diferença.

3. **Stack canônica de análise de custo:**
   ```
   Labels → Cloud Billing → BigQuery export → Looker Studio → ação
   ```
   - **Labels** dão as dimensões além de project/SKU
   - **BigQuery export** persiste o dado bruto, viabiliza SQL
   - **Looker Studio** visualiza, compartilha como Google Docs (sem precisar dar IAM no projeto)
   - **Ação** = decisão humana baseada no dashboard

4. **Exemplo concreto do vídeo — egress caro:**
   VMs em `us-central1` enviando tráfego majoritário pra Europa → custo de **inter-region egress** dispara. Solução: relocar VMs pra `europe-west1` OU ligar **Cloud CDN** com cache na borda (Europa). A descoberta vem de cruzar `labels.customer-region:eu` no BigQuery, não do email do alerta.

**Por que não duplico aqui o conteúdo de budgets/billing accounts:** já está coberto em `1.2-billing.md`, e duplicar = arquivos divergindo no tempo. Ficou nesse arquivo só o que este vídeo do curso adicionou.

---

## Módulo 4 — Resource Monitoring

> **Status:** 🟡 em andamento · **Iniciado em:** 2026-05-14
> **Fontes:** vídeos da seção *Resource Monitoring* (Google Cloud Skills Boost)
> **Subseção da prova:** 4.6 (Monitoring and Logging)
> **Arquivo canônico:** [`4.6-monitoring.md`](4.6-monitoring.md)

### 4.1 — Visão geral do Google Cloud Observability

**O quê.** **Google Cloud Observability** é o **guarda-chuva unificado** de telemetria do GCP. Não é um produto único: é uma **suíte integrada** que reúne, sob um mesmo plano de controle, os serviços de **métricas (Cloud Monitoring)**, **logs (Cloud Logging)**, **erros de aplicação (Error Reporting)**, **rastreamento distribuído (Cloud Trace)** e **profiling de performance (Cloud Profiler)**.

> Nota histórica: a marca antiga era **Stackdriver**. Material mais velho ainda usa o nome — é o mesmo serviço, rebatizado para "Google Cloud Observability" em 2020.

#### Os 3 pilares de observability (e onde cada um vive na suíte)

Toda discussão moderna de observability gira em torno de três sinais. Importante entender porque a prova pode cobrar "qual ferramenta para tal sinal":

| Pilar (sinal) | O que mede | Ferramenta na suíte | Pergunta típica que responde |
|---------------|-----------|---------------------|------------------------------|
| **Metrics** | Números agregados ao longo do tempo (CPU%, req/s, latência p99) | **Cloud Monitoring** | "O sistema está saudável **agora**?" |
| **Logs** | Eventos timestampados com texto livre/estruturado | **Cloud Logging** | "**O que aconteceu** naquele momento específico?" |
| **Traces** | Caminho de uma request atravessando múltiplos serviços | **Cloud Trace** | "**Onde** está o gargalo dessa request distribuída?" |

E dois "primos" especializados que completam a suíte:
- **Error Reporting** — agrupa e agrega exceptions de aplicação (deduplicação inteligente, alerta quando uma nova classe de erro aparece em produção)
- **Cloud Profiler** — profiling contínuo de CPU/heap das suas aplicações (overhead baixo, roda em prod)

#### Os 4 diferenciais vendidos pelo Google (e o que cada um significa)

O vídeo de abertura lista quatro características da suíte. Vale destrinchar o que significa cada uma na prática.

**1. Integração profunda com GCP — e suporte a AWS.**
Cada serviço gerenciado do GCP **já emite métricas e logs no Observability sem você configurar nada**: GKE, Cloud Run, Cloud SQL, Pub/Sub, Load Balancers — todos aparecem automaticamente. Para VMs em outras nuvens (ou on-prem), você instala o **Ops Agent** e a telemetria chega no mesmo painel.

A **integração multicloud com AWS** é o ponto curioso: você pode criar um **monitoring workspace** (hoje chamado de "metrics scope") que **inclui projetos GCP e contas AWS**. Útil para empresas em transição ou genuinamente multicloud — um único dashboard mostra "saúde do meu sistema" sem importar onde ele roda.

> **Armadilha de prova.** "Posso monitorar instâncias EC2 no Cloud Monitoring?" → **Sim, com o agente instalado e a conta AWS conectada via metrics scope.** Não confunda com Pub/Sub ou Storage (esses ainda exigem export).

**2. Smart defaults (configurações padrão inteligentes).**
"Visibilidade completa em minutos" significa: você cria um projeto, sobe uma VM/Cloud Run, e **dashboards básicos já existem sem você desenhar nada**. CPU, memória de instância, requests, latência — tudo aparece pronto. Você só precisa investir tempo customizando quando o caso é específico (alertas customizados, métricas de aplicação, SLO).

Por que isso importa em prova: pergunta tipo "qual a forma mais rápida de ver métricas básicas das minhas VMs?" → **nenhuma configuração**, é nativo. Não precisa criar dashboard, não precisa instalar agent (para métricas básicas como CPU host-level).

**3. Ferramentas de dados/análise + colaboração com terceiros.**
A suíte exporta para **BigQuery** (análise SQL de logs), **Looker Studio** (dashboards compartilháveis), **Pub/Sub** (streaming para ferramentas externas como Splunk, Datadog, PagerDuty). Não é um silo: dá pra plugar a stack que sua empresa já usa.

**4. Pay-per-use com free tier.**
Modelo de cobrança canônico do GCP: você paga pelo volume de **logs ingeridos** e **métricas customizadas** que cria; métricas padrão dos serviços gerenciados são incluídas sem custo extra. Existe **tier gratuito mensal** (suficiente pra dev/pequenos projetos não pagarem nada). Comparado a SaaS de monitoring (Datadog, New Relic) que cobram por host, o modelo do GCP costuma ser mais barato em escala média.

#### O argumento central: por que "tudo numa suíte só" importa

O vídeo termina batendo nesse ponto e ele é didaticamente importante. Em arquiteturas tradicionais você teria:

```
Logs → ELK stack (Elasticsearch + Logstash + Kibana)
Metrics → Prometheus + Grafana
Traces → Jaeger ou Zipkin
Errors → Sentry
Profiling → pyspy/perf rodando manualmente
```

Cinco produtos, cinco UIs, cinco contas, cinco modelos de cobrança, integração frágil entre eles ("por que esse trace não tem log associado?"). **Cada produto fala um dialeto diferente** e correlacionar um problema de produção entre eles é trabalho manual de SRE.

No Cloud Observability, **uma request com erro** te leva: log no Logging → métrica no Monitoring → trace no Trace → exception agrupada no Error Reporting — **com link direto entre eles**. Isso é o "abrangente e integrado" do vídeo: a **correlação entre os sinais** é nativa, não um plugin que alguém configurou.

**Por que importa na feira.** "Como vocês fazem observability?" — falar "usamos a suíte do Cloud Observability porque correlaciona logs, métricas e traces nativamente" mostra que você entende o problema que essa integração resolve, não só os nomes dos produtos.

#### Cenário real

Aplicação de e-commerce em Cloud Run. Cliente reporta lentidão no checkout. Sem observability integrada, você abriria 4 abas e tentaria casar timestamps na mão. Com Cloud Observability:

1. Abre **Cloud Monitoring** → vê pico de latência p99 em checkout-service às 14:32
2. Clica no gráfico → "View logs for this resource and time range" → cai no **Cloud Logging** filtrado naquela janela
3. Acha logs de timeout para o pagamento → clica no Trace ID → cai no **Cloud Trace** mostrando que a chamada para o gateway externo demorou 8s
4. **Error Reporting** já agrupou todas as exceptions desse tipo e mostra quantos usuários foram afetados

Tempo do problema reportado até causa-raiz: minutos, não horas. Esse é o pitch.

#### Armadilhas de prova

| Pegadinha | Resposta |
|-----------|----------|
| "Qual ferramenta para latência distribuída entre serviços?" | **Cloud Trace** (não Monitoring, não Logging) |
| "Onde agrupar exceptions/stack traces da aplicação?" | **Error Reporting** (Logging guarda os logs brutos; Error Reporting agrupa por padrão) |
| "Como ver métricas de VMs em AWS no GCP?" | Cloud Monitoring + **Ops Agent** + conexão de conta no **metrics scope** |
| "Stackdriver ainda existe?" | É o nome **antigo** — hoje a marca é **Google Cloud Observability**. Material desatualizado pode usar o termo |
| "Métricas básicas de uma VM precisam de agente?" | **NÃO** para métricas host-level vistas pela hypervisor (CPU%, rede, disco I/O). **SIM** para métricas internas do SO (memória usada, processos, logs de syslog) → precisa **Ops Agent** |

---

### 4.2 — Cloud Monitoring em profundidade

#### Por que o Google investe pesado em monitoring

O vídeo começa amarrando Cloud Monitoring à **Site Reliability Engineering (SRE)** — a disciplina criada pelo Google que **aplica práticas de engenharia de software a operações**. SRE existe porque o Google precisou rodar Search, Gmail, YouTube em escala planetária e descobriu que **"jogar mais gente no problema" não escala** — precisava-se de automação, observabilidade e error budgets.

Em prova/feira, vale saber que Cloud Monitoring **não é só "uma ferramenta de gráficos"** — é a base do modelo SRE do Google, e cada feature reflete uma lição aprendida operando sistemas gigantes.

#### O que Cloud Monitoring **ingere** (e o que faz com isso)

Três tipos de dado entram no Monitoring:

| Tipo | Exemplo |
|------|---------|
| **Métricas** | Valores numéricos com timestamp (CPU%, requests/s, latência p99) |
| **Eventos** | "Deploy aconteceu", "auto-scaling escalou" — marcadores no tempo |
| **Metadados** | Labels do recurso, região, projeto, instance group |

A partir disso a ferramenta gera:
- **Dashboards** (gráficos persistentes)
- **Charts** (gráficos ad-hoc)
- **Alertas** (regras que disparam notificações)
- **Uptime checks** (verificações ativas externas)

#### Metrics scope — o conceito **mais cobrado em prova** desta seção

**O quê.** **Metrics scope** (antigo "monitoring workspace") é a **entidade raiz** que agrupa configurações de monitoring no Cloud Monitoring. Você não cria alertas/dashboards "no projeto" — você cria **em um metrics scope** que **referencia** um ou mais projetos.

```
Metrics Scope "prod-observability"           ← onde vivem dashboards, alertas,
   │                                            uptime checks, notification channels
   ├── Scoping project: prod-ecommerce       ← 1º projeto adicionado; vira o nome do scope
   ├── Monitored project: prod-payments
   ├── Monitored project: prod-inventory
   ├── Monitored project: aws-connector-proj ← projeto que hospeda o connector da AWS
   └── ... (até 375 projetos)
```

**Regras críticas (decorar):**

| Regra | Detalhe |
|-------|---------|
| **1 a 375 projetos** por scope | Limite hard |
| **Dados ficam no projeto de origem** | O scope é uma **visão**, não copia dados. Logs e métricas continuam vivendo onde foram emitidos |
| **Scoping project = 1º projeto** | O 1º projeto adicionado dá nome ao scope e tem papel especial (host de configs) |
| **AWS connector mora num GCP project** | Pra monitorar AWS, você cria um projeto GCP que hospeda o conector da AWS — esse projeto entra no scope |
| **Acesso = "tudo ou nada"** | Role concedida no scope vê **TODOS** os projetos monitorados |

**A pegadinha de acesso (cenário de prova clássico).**

> "Time de pagamentos não pode ver métricas do time de inventory. Como configurar?"

❌ **NÃO** funciona: adicionar os dois projetos no mesmo scope e tentar restringir IAM por projeto.
**Razão:** o IAM do scope é único e se aplica uniformemente a todos os projetos monitorados. Você dá role no scope → o usuário vê tudo.

✅ **Funciona:** **scopes separados** — um scope para pagamentos, outro para inventory, IAM diferente em cada.

**Por que essa decisão de design.** Metrics scope foi pensado pra "war room central" — SRE quer **uma tela** mostrando saúde de toda a frota. Compartimentalização por projeto seria um produto diferente. Quem precisa de isolamento real cria múltiplos scopes.

**Exemplo concreto.**
```
Empresa Cymbal Superstore:
   metrics-scope: cymbal-sre        ← time SRE central vê tudo
        ├── ecommerce-prod
        ├── payments-prod
        ├── inventory-prod
        └── aws-warehouse-conn

   metrics-scope: payments-isolated ← compliance PCI exige isolamento
        └── payments-prod          ← mesmo projeto, scope separado

   IAM no cymbal-sre: time SRE (10 pessoas)
   IAM no payments-isolated: 2 SREs de pagamentos + auditor PCI
```

Note que **um mesmo projeto pode estar em vários scopes simultaneamente** — não é exclusividade. O scope é só uma "saved view".

#### Dashboards e charts — refinamento que vale na prova

Charts customizados aceitam três operações para virar úteis:

| Operação | O que faz | Exemplo |
|----------|-----------|---------|
| **Filter** | Remove ruído / restringe o conjunto | "só VMs com label `environment=production`" |
| **Group** | Agrupa séries temporais para reduzir cardinalidade | "agrupar por zona" → 3 linhas no gráfico em vez de 50 |
| **Aggregation** | Combina múltiplas séries | mean / sum / max / p99 |

Sem essas três operações, dashboards viram "ondas no mar" — milhares de linhas indecifráveis.

#### Alertas — anatomia + práticas recomendadas

Já cobrimos a estrutura em [4.6-monitoring.md](4.6-monitoring.md). O vídeo adiciona **4 best practices SRE** que são prova certa e ouro de feira:

**1. Alertar em sintomas, não em causas.**
- ❌ Errado: "alertar quando o DB está down"
- ✅ Certo: "alertar quando as queries da aplicação estão falhando"

**Por quê.** A causa pode mudar (DB down hoje, rede caída amanhã, certificado expirado depois). O **sintoma é o que importa pro usuário** — "minhas queries falharam" continua relevante independente da causa. Alertar em sintoma também evita alert duplicado quando uma causa única gera vários efeitos (DB down dispararia o alerta dele E os de cada serviço que depende dele).

**2. Use múltiplos canais de notificação.**
Email é fácil, mas: caixa lotada, filtro do Outlook, internet do escritório caiu. Combine email + SMS + Slack + PagerDuty. Você **não quer ter um único ponto de falha na sua estratégia de alertas**.

**3. Personalize por audiência.**
Alerta pra um dev tem **stack trace + link pro log**. Alerta pro NOC tem **runbook resumido + número do plantão**. Mesma condição, mensagem diferente — o campo `documentation` da alert policy serve exatamente pra isso.

**4. Evite ruído (alert fatigue).**
Alerta que dispara toda hora e ninguém investiga **vira ignorado**. Quando o alerta realmente importante chegar, vai junto no ruído. Regra: **alertar somente para coisas acionáveis** — se ao receber o alerta a pessoa não tem ação clara a tomar, ele não devia existir.

Existe também um alerta especial: monitorar o **próprio uso do Cloud Observability** pra avisar antes de estourar o billing. É meta-alerta — usar Observability pra evitar surpresa na fatura do Observability.

#### Uptime checks — verificação ativa do mundo de fora

**O quê.** Cloud Monitoring dispara probes a partir de **locais distribuídos no mundo** (não dos seus servidores) que tentam acessar seu endpoint. Se a resposta não chega dentro do timeout, é considerado **falha** — e isso vira alerta.

| Campo | Opções | Observação |
|-------|--------|------------|
| **Tipo** | HTTP, HTTPS, TCP | HTTPS valida certificado também |
| **Alvo** | App Engine app, Compute Engine instance, URL pública, AWS instance, AWS Load Balancer | Funciona em endpoint privado? Só com configuração extra |
| **Frequência** | 1, 5, 10, 15 minutos | Padrão 1 min |
| **Timeout** | até 60s | Padrão 10s no exemplo do curso |
| **Locais** | regiões globais (selecionáveis) | Para evitar falso positivo de uma região, exige falha em **múltiplos locais** antes de alertar |

**Por que importa.** Métrica interna (CPU baixa, requests OK) **não detecta** problemas de DNS, certificado expirado, balanceador mal configurado ou queda de uma região inteira. Uptime check é a única forma de saber "meu site está acessível **pra um cliente real fora da minha rede**".

**Armadilha de prova.** "Como detectar que o site está fora do ar do ponto de vista do cliente?" → **Uptime check**, não métrica de CPU/requests do servidor. O servidor pode estar saudável e o site inacessível (problema de rede a montante).

#### Fontes de dados — onde a métrica nasce

O vídeo enfatiza um ponto sutil sobre VMs:

> Como as VMs rodam em hardware do Google, **o hipervisor não enxerga métricas internas da VM** — não consegue ver memória usada pelo SO, processos rodando, conteúdo de syslog.

Isso explica a divisão de métricas em duas classes:

| Classe | Fonte | Exemplos | Precisa de agente? |
|--------|-------|----------|---------------------|
| **Host-level** (hypervisor) | Google enxerga "de fora" da VM | CPU%, network bytes in/out, disk I/O, instance uptime | **Não** |
| **Guest-level** (dentro do SO) | Só visível pra quem roda dentro da VM | Memory usage, swap, processos, logs do syslog, métricas de app | **Sim** — Ops Agent |

#### Ops Agent — recap consolidado

- Agente unificado (substitui o Stackdriver Agent legado, que era 2 binários separados)
- Roda **dentro da VM** e envia para Cloud Monitoring + Cloud Logging
- Suporta CentOS, Ubuntu, Windows e outras
- Pode monitorar **aplicações de terceiros**: nginx, Apache, MySQL, Redis, MongoDB, RabbitMQ, etc. — basta habilitar o receiver correspondente no `config.yaml`
- Configuração principal: `/etc/google-cloud-ops-agent/config.yaml`

```bash
# Instalação canônica (em uma VM existente)
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
sudo bash add-google-cloud-ops-agent-repo.sh --also-install

# Verificar status
sudo systemctl status google-cloud-ops-agent

# Logs do próprio agent (pra debug)
sudo journalctl -u google-cloud-ops-agent
```

> **Em prova:** "Como coletar métrica de **memória** de uma VM?" → **Ops Agent**. Memory não é host-level, hypervisor não enxerga.

#### Métricas customizadas — quando as padrão não bastam

Cenário canônico do vídeo: **servidor de jogos com capacidade para 50 usuários simultâneos**. Você quer autoscaling. O que usar como trigger?

- **CPU%** — proxy ruim. Servidor de jogo pode estar com 30% de CPU e 49 usuários — quase no limite.
- **Network in/out** — proxy ruim. Idle players geram tráfego mínimo.
- **Active users** — **métrica perfeita**, mas não existe nativamente.

**Solução:** a aplicação envia diretamente pro Cloud Monitoring uma métrica customizada `users/active`. O autoscaler usa essa métrica como sinal de escalonamento.

```python
# Pseudo-código de envio de métrica customizada (Python)
from google.cloud import monitoring_v3

client = monitoring_v3.MetricServiceClient()
project_name = f"projects/{PROJECT_ID}"

series = monitoring_v3.TimeSeries()
series.metric.type = "custom.googleapis.com/game/active_users"
series.resource.type = "gce_instance"
series.resource.labels["instance_id"] = INSTANCE_ID
series.resource.labels["zone"] = ZONE

point = monitoring_v3.Point({
    "interval": {"end_time": {"seconds": now}},
    "value": {"int64_value": current_user_count}
})
series.points = [point]
client.create_time_series(name=project_name, time_series=[series])
```

A métrica aparece em **`custom.googleapis.com/game/active_users`** no Metrics Explorer e pode ser referenciada em dashboards, alertas e **políticas de autoscaling de MIG**.

#### Autoscaling com métrica customizada — a regra do "average vs group"

Detalhe técnico do vídeo que vale na prova:

| Origem da métrica | Como o autoscaler usa |
|-------------------|------------------------|
| Métrica vem **de cada VM** do MIG | Autoscaler calcula a **média** entre todas as VMs e compara com o target |
| Métrica é **de grupo** (não vem das VMs) | Autoscaler compara o valor direto da métrica de grupo com o target |
| Métrica tem **múltiplos valores** (várias séries) | Aplique um **filter** para o autoscaler usar uma série específica |

**Exemplo prático.**
```
Game server MIG com 5 VMs, cada uma emite "active_users":
   vm-1: 40  vm-2: 45  vm-3: 38  vm-4: 42  vm-5: 41
   Média = 41.2
   Target = 40 (utilização-alvo)
   Como 41.2 > 40 → autoscaler adiciona VM
```

#### Armadilhas de prova críticas (4.2)

| Pegadinha | Resposta |
|-----------|----------|
| "Time A não pode ver métricas do time B" | **Metrics scopes separados** — não tente IAM-por-projeto dentro do mesmo scope |
| "Quantos projetos cabem em um metrics scope?" | **1 a 375** |
| "Quem dá o nome ao scope?" | O **scoping project** (1º projeto adicionado) |
| "Como monitorar AWS no Cloud Monitoring?" | Projeto GCP hospeda o **AWS connector**; entra no metrics scope |
| "Alertar em causa ou em sintoma?" | **Sintoma** (queries falhando), não causa (DB down) |
| "Como saber se o site está fora pra cliente externo?" | **Uptime check** (probe externa), não métrica interna |
| "Métrica de memória da VM precisa de agente?" | **Sim** — Ops Agent (memory é guest-level, hypervisor não vê) |
| "Autoscaler de game server com 50 usuários máx?" | **Métrica customizada** (active users), não CPU/network |

#### Cenário real

Cymbal Superstore tem 4 projetos: `web-prod`, `payments-prod`, `inventory-prod`, `analytics-prod`. SRE quer:
1. **Uma tela** mostrando saúde geral
2. Time de pagamentos com **acesso isolado** para compliance PCI
3. Alertas em **sintomas** (5xx no LB, latência p99 alta)
4. **Uptime checks** globais no checkout (HTTPS, 1 min, 5 regiões)
5. Notificações em **email + PagerDuty + Slack**

Configuração:
- Scope `cymbal-sre` → contém os 4 projetos, IAM = time SRE
- Scope `payments-pci` → contém só `payments-prod`, IAM = 2 SREs + auditor
- Alert policies criadas no scope `cymbal-sre` (sintomas do LB, latência, erros do Cloud Run)
- Uptime check apontando para `https://shop.cymbal.com/checkout`, 5 locations
- 3 notification channels conectados a cada alert policy

Resultado: a SRE tem **single pane of glass**, pagamentos tem **isolamento auditável**, alertas são **acionáveis** e não há **SPOF no canal de aviso**.

---

### 4.3 — Cloud Logging

#### O quê

**Cloud Logging** é o serviço de **log centralizado e totalmente gerenciado** do GCP. "Totalmente gerenciado" significa: você não precisa instalar ELK, não precisa dimensionar Elasticsearch, não precisa configurar retenção de índice — o Google cuida de tudo isso até milhares de VMs emitindo logs simultaneamente.

**Três componentes do serviço:**

| Componente | O que é |
|------------|---------|
| **Log storage** | Armazenamento gerenciado, retenção padrão de 30 dias |
| **Logs Explorer** | Interface web para busca, filtro e análise de logs |
| **API** | Permite ler/escrever entradas de log programaticamente |

**O que você pode fazer:**
- Ler e escrever log entries (via API, gcloud, SDKs)
- Buscar e filtrar logs (linguagem de filtro própria, muito usada em prova)
- Criar **log-based metrics** — transformar padrão de log em métrica numérica para alertar

#### Retenção e os destinos de exportação (sinks)

**Retenção padrão: 30 dias.** Depois disso os logs somem. Se você precisar de mais, **export via sink**.

```
Cloud Logging ──sink──► Cloud Storage       (retenção longa, barato)
              ──sink──► BigQuery            (análise SQL + Looker Studio)
              ──sink──► Pub/Sub             (streaming para sistemas externos)
              ──sink──► Log Buckets (outro) (roteamento interno dentro do GCP)
```

**Por que cada destino? (pergunta clássica de prova):**

| Destino | Caso de uso canônico |
|---------|----------------------|
| **Cloud Storage** | Logs > 30 dias por compliance. Formato: arquivo comprimido em bucket. Barato, mas não é queryable diretamente. |
| **BigQuery** | Análise — SQL em gigabytes a petabytes de logs. Integra com **Looker Studio** para dashboards. Caso do vídeo: top IPs que trocaram tráfego com servidor web, análise de crescimento de tráfego, forense de rede. |
| **Pub/Sub** | Streaming para sistemas **externos** (Splunk, Datadog, SIEM on-premises). Pub/Sub é o "conector universal" — qualquer sistema que consome Pub/Sub recebe os logs. |

**Exemplo de análise com BigQuery** (mencionado no vídeo):
```sql
-- Top 10 IPs que mais acessaram o web server
SELECT
  jsonPayload.remoteIp AS ip,
  COUNT(*) AS requests
FROM `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE resource.type = 'gce_instance'
GROUP BY ip
ORDER BY requests DESC
LIMIT 10;
```

A partir disso você decide: realocar infra para reduzir egress de uma região cara, ou bloquear IPs suspeitos via firewall rule.

#### Armadilhas de prova

| Pegadinha | Resposta |
|-----------|----------|
| "Por quanto tempo logs ficam no Cloud Logging?" | **30 dias** (padrão). Para mais, sink para Cloud Storage. |
| "Como analisar logs com SQL?" | Sink para **BigQuery** |
| "Como enviar logs para o Splunk?" | **Pub/Sub** → Dataflow → Splunk (não é direto) |
| "Como visualizar logs em dashboard?" | BigQuery + **Looker Studio** |
| "Log-based metrics serve pra quê?" | Criar métricas de Monitoring a partir de padrões nos logs → alertar em ocorrências de texto específico |

---

### 4.4 — Error Reporting

#### O quê

**Error Reporting** é o serviço de **agregação inteligente de exceções**. Ele **não é o Cloud Logging** — enquanto Logging guarda linhas de log brutas, Error Reporting **agrupa stack traces similares** em uma única "classe de erro", conta ocorrências, e alerta quando um **novo** tipo de erro aparece.

Analogia: imagine que sua aplicação lança `NullPointerException` 5.000 vezes em 10 minutos. No Logging você veria 5.000 linhas. No Error Reporting você veria **1 card** com "NullPointerException, 5.000 ocorrências, afetou X usuários, primeira vez: 14:32".

**Funcionalidades:**

- **Contagem e agregação** de erros por tipo/stack trace
- **Dashboard centralizado** com sorting (mais frequente, mais recente, novo) e filtering
- **Notificações em tempo real** quando um **novo** tipo de erro é detectado (armadilha: não alerta em erro já conhecido que piorou, só em novo)

#### Plataformas suportadas (lista de prova)

| Plataforma | Suportado? |
|------------|------------|
| App Engine Standard | ✅ |
| App Engine Flexible | ✅ |
| Apps Script | ✅ |
| Compute Engine | ✅ |
| Cloud Run | ✅ |
| Cloud Run functions | ✅ |
| Google Kubernetes Engine | ✅ |
| **Amazon EC2** | ✅ (surpresa! multicloud) |

> **Armadilha de prova.** EC2 (AWS) é suportado pelo Error Reporting. Pergunta "quais plataformas são suportadas?" que inclui EC2 como distrator — mas EC2 **é** uma resposta correta.

#### Linguagens suportadas (stack trace parser)

Go, Java, .NET, Node.js, PHP, Python, Ruby.

#### Armadilhas de prova

| Pegadinha | Resposta |
|-----------|----------|
| "Onde ver erros **agrupados** da aplicação?" | **Error Reporting** (Logging tem linhas brutas, não agrupa) |
| "Alerta de erro **novo** em produção?" | Error Reporting (notificação em tempo real para **novos** tipos de erro) |
| "Error Reporting funciona com EC2?" | **Sim** — plataforma AWS suportada |
| "Error Reporting vs Cloud Logging?" | Reporting = agregação/deduplication de exceptions; Logging = armazenamento bruto de tudo |

---

### 4.5 — Cloud Trace

#### O quê

**Cloud Trace** é o sistema de **distributed tracing** — ele responde à pergunta: *"por onde passou essa request e quanto tempo cada parte demorou?"*

Em arquiteturas de microserviços, uma request do usuário pode passar por 10 serviços diferentes antes de retornar. Se a latência total é 3 segundos, **onde estão esses 3 segundos?** É exatamente isso que o Trace mostra.

```
Usuário ──► API Gateway (50ms)
                └──► Auth Service (20ms)
                └──► Product Service (80ms)
                         └──► Database (2500ms) ← GARGALO
                └──► Cart Service (30ms)
Total: ~3s
```

Com o Trace, essa visualização (chamada **waterfall diagram** ou **flame graph**) aparece no console com timestamps de cada "span".

#### Funcionalidades

- **Coleta automática** de latência de apps no App Engine, global external Application LBs e apps instrumentados com a Cloud Trace API
- **Near real-time** — você vê traces em segundos
- **Análise automática** — o Trace detecta **performance degradations** automaticamente comparando latência atual vs histórico (sem você configurar threshold)
- **Latency reports** — relatórios de distribuição de latência (p50, p95, p99)

#### Quando usar vs outras ferramentas

| Ferramenta | Quando usar |
|------------|------------|
| **Cloud Monitoring** | "Minha latência p99 está alta" (métrica agregada) |
| **Cloud Trace** | "Qual serviço específico está causando essa latência?" (trace individual) |
| **Cloud Logging** | "O que exatamente aconteceu nessa request?" (logs linha a linha) |

O fluxo típico de debugging: Monitoring mostra pico de latência → Trace mostra qual serviço é lento → Logging mostra os logs daquele serviço naquele instante.

> **Contexto histórico do vídeo:** Cloud Trace é baseado nas ferramentas internas que o Google usa para operar seus próprios serviços em escala extrema (o sistema interno se chama "Dapper").

#### Armadilhas de prova

| Pegadinha | Resposta |
|-----------|----------|
| "Onde ver latência distribuída entre microserviços?" | **Cloud Trace** (não Monitoring, que mostra métrica agregada) |
| "Error Reporting vs Cloud Trace?" | Error Reporting = exceções/erros; Trace = **latência** do caminho da request |
| "Cloud Trace detecta degradações automaticamente?" | **Sim** — analisa e compara com histórico automaticamente |

---

### 4.6 — Cloud Profiler

#### O quê

**Cloud Profiler** resolve um problema diferente dos outros: não é "o sistema está lento" (Trace) nem "o sistema tem erros" (Error Reporting) — é *"qual função específica no código está consumindo mais CPU ou memória?"*

É como ter um **profiler de desenvolvimento** (py-spy, async-profiler, pprof) rodando **continuamente em produção**, sem impacto perceptível.

#### O problema que resolve

```
Código em desenvolvimento: profiler mostra função X é lenta.
Mesmo código em produção: função X pode ser rápida (dados menores) 
ou mais lenta (dados reais, concorrência real, JIT diferente).
```

Profilers de produção tradicionais têm dois problemas:
1. **Sampling intrusivo** — pausa threads, degrada performance
2. **Coverage parcial** — você só consegue perfilar um subset do código

**Cloud Profiler usa técnicas estatísticas** com instrumentação de baixíssimo impacto e roda em **todas as instâncias de produção** simultaneamente — você vê o perfil médio de toda a frota, não de uma amostra.

#### Funcionalidades

- **CPU profiling** — quais funções consomem mais tempo de CPU?
- **Memory/heap profiling** — quais funções alocam mais memória? (útil pra detectar memory leaks)
- **Flame graph** — visualização hierárquica de call stack com tamanho proporcional ao custo
- **Comparação entre versões** — "o deploy de hoje foi mais lento que o de ontem em qual função?"

#### Plataformas e linguagens

Roda em qualquer lugar: **GCP, outras nuvens, on-premises**.
Linguagens: **Java, Go, Node.js, Python**.

#### Quando usar

Você usa Profiler quando:
- Custo de CPU/memória está alto e não sabe **por quê** no nível de código
- Quer otimizar antes de escalar (escalar código ineficiente só escala o custo)
- Acabou de fazer um deploy e quer comparar perfil antes/depois

#### Armadilhas de prova

| Pegadinha | Resposta |
|-----------|----------|
| "Qual ferramenta para otimizar funções custosas em produção?" | **Cloud Profiler** |
| "Cloud Profiler impacta performance de produção?" | **Não** — overhead baixíssimo por design |
| "Profiler funciona só no GCP?" | **Não** — GCP, outras nuvens, on-prem |
| "Quais linguagens?" | Java, Go, Node.js, Python |

---

### 4.7 — Ecossistema de parceiros: BindPlane e Splunk

#### BindPlane (Blue Medora) — ingesta de fontes externas

**Problema:** Cloud Logging aceita logs nativamente de serviços GCP e AWS via Ops Agent. Mas e logs de um banco de dados on-premises? Um appliance de rede? Um sistema legado?

**BindPlane** é o conector que resolve isso: ele ingesta logs e métricas de **fontes não-GCP** (sistemas legados, appliances, on-prem) e os empurra para as **APIs abertas** do Cloud Logging e Cloud Monitoring.

```
Fontes externas ──► BindPlane ──► Cloud Logging API  ──► todos os recursos do Logging
                             └──► Cloud Monitoring API ──► dashboards, alertas, etc.
```

Após ingestão, esses logs se comportam **exatamente como logs GCP nativos** — você pode criar log-based metrics, alertar, exportar via sink, visualizar no Logs Explorer.

#### Integração com Splunk — o fluxo canônico

**Splunk** é uma plataforma de SIEM (Security Information and Event Management) amplamente usada em enterprises. O fluxo de integração mencionado no vídeo é o **padrão de mercado** para "enviar logs GCP para Splunk":

```
Cloud Logging
      │
      ▼ (sink)
   Pub/Sub  ◄── buffer temporário, desacopla produtores de consumidores
      │
      ▼
  Dataflow Pipeline (template "Pub/Sub to Splunk")
   ├── Pipeline primário ──► Splunk (entrega normal)
   └── Pipeline secundário ──► re-envia se falhar (failsafe)
      │
      ▼
    Splunk
   (on-prem, GCP, SaaS, ou híbrido)
```

**Por que Pub/Sub no meio?** Pub/Sub é o **buffer que desacopla** Cloud Logging de Splunk. Se o Splunk ficar temporariamente indisponível, as mensagens ficam no Pub/Sub (por até 7 dias no padrão) sem perda. Quando volta, o Dataflow processa a fila acumulada.

**Dataflow template:** `Pub/Sub to Splunk` — template gerenciado pelo Google, você preenche configs de conexão e pronto. Qualquer mensagem que chegar ao Pub/Sub topic configurado vai para o Splunk.

#### Por que esse fluxo importa na prova

Questão típica: "Como exportar logs do GCP para um SIEM externo (como Splunk) de forma confiável?"

Resposta: **Cloud Logging → sink para Pub/Sub → Dataflow (template Pub/Sub to Splunk) → Splunk**. Não é uma exportação direta — o Pub/Sub é essencial para durabilidade e desacoplamento.

#### Armadilhas de prova

| Pegadinha | Resposta |
|-----------|----------|
| "Logs para Splunk via conexão direta?" | **Não** — precisa de Pub/Sub + Dataflow como intermediários |
| "Para que serve BindPlane?" | Ingestar logs/métricas de **fontes não-GCP** no Cloud Logging/Monitoring |
| "Por que Pub/Sub entre Logging e Splunk?" | **Durabilidade e desacoplamento** — se Splunk cair, mensagens ficam no Pub/Sub sem perda |
| "O pipeline Dataflow tem failsafe?" | **Sim** — pipeline secundário reprocessa em caso de falha |

---

## Labs completados

<!-- documentar labs aqui -->

---

## DQs cobertas por este curso

<!-- listar quais DQs este curso esclarece -->

---

## Conexões com arquivos de subseção

<!-- ao terminar o curso, listar quais arquivos foram atualizados/criados com base neste conteúdo -->
