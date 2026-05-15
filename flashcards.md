# Flashcards — Erros do Auto-teste

> Quando errar uma pergunta do auto-teste, registrar aqui. Revisar todo dia no warm-up (15min).

**Formato:**
```
### [Dia X — Tópico] Pergunta resumida
- **Errei porque:** <minha confusão>
- **Correto:** <resposta certa + por quê>
- **Por que as outras estão erradas:** <breve>
- **Mnemônico/dica:** <opcional>
```

---

### [1.1 — Hierarquia] Q1 do quiz Skills Boost / DQ 02 workbook: ordem da hierarquia GCP
- **Errei porque:** escolhi "Projeto, Organização, Pasta, Recurso" — coloquei Project no topo
- **Correto:** **Organização → Pasta → Projeto → Recurso**
- **Por que as outras estão erradas:** qualquer ordem que não comece em Organization e desça contendo (Folder → Project → Resource) está errada. Recursos moram em Projects, Projects em Folders/Orgs, Folders em Orgs
- **Mnemônico:** "Tudo mora dentro de algo. Org é o teto, Resource é o chão."

---

### [1.2 — Billing] Q2 do quiz / DQ 07 workbook: relação billing accounts ↔ projetos (pick two)
- **Errei porque:** pulei — não sabia a cardinalidade
- **Correto:** "**1 projeto → 1 billing account ativa**" + "**1 billing account → N projetos**" (relação N:1)
- **Por que as outras estão erradas:** "1 projeto pode ter várias billings" é falso direto. "Cloud Billing paga GCP+Workspace" é verdadeiro mas não responde "como aplicado a projetos" — descreve o que o serviço é
- **Mnemônico:** "Many projects share one bill, never the other way around."

---

### [1.2 — Billing] Q4 do quiz / DQ 08 workbook: dar visibilidade de budget alert sem dar billing access
- **Errei porque:** pulei — não conhecia o campo "custom email recipients"
- **Correto:** adicionar a pessoa em **custom email recipients** do budget (até 5 emails, sem precisar de role billing)
- **Por que as outras estão erradas:** mudar threshold rules → não muda QUEM recebe; Cloud Monitoring channels → sistema separado de Billing; Pub/Sub → overkill, é para automação programática
- **Padrão geral:** visibilidade sem acesso admin = mecanismo de notificação dedicado do recurso, nunca promover a role

---

### [1.1 — IAM] Q5 do quiz / DQ 01 workbook: Stella monitorando VMs (group + role predefinida)
- **Errei porque:** escolhi "atribua a política de visualizador à Stella" — confundi "viewer" solto com a role granular correta, e atribuí direto à pessoa em vez de grupo
- **Correto:** **adicionar Stella a um Google Group + vincular o grupo a `roles/compute.viewer`** (predefinida específica de Compute, não a basic `roles/viewer`)
- **Por que as outras estão erradas:** "roles/compute.viewer direto à pessoa" → não-escalável; "compute.instances.get" → permission isolada não é role; "viewer policy" → basic role ampla demais
- **Mnemônico:** "Group > User. Predefinida específica > basic genérica."

---

### [1.1 — IAM] Q7 do quiz / DQ 04 workbook: Jane gerenciando objetos em todos os projetos
- **Errei porque:** pulei — não sabia que dava pra atribuir role no nível organizacional via grupo
- **Correto:** **grupo + `roles/storage.objectAdmin` no nível organizacional** (herda pra todos os projetos automaticamente)
- **Por que as outras estão erradas:** `objectCreator` só cria, não gerencia; `Editor` no nível Org é poder absurdo; replicar setting por projeto/bucket não escala
- **Padrão de prova:** "em todos os projetos da org" = grupo + role predefinida + nível organizacional

---

### [1.1 — IAM] Q8 do quiz / DQ 05 workbook: least privilege para novos grupos
- **Errei porque:** pulei — não tinha clareza sobre a hierarquia de preferência das roles
- **Correto:** **predefinidas e custom como padrão, basic só em exceção**
- **Por que as outras estão erradas:** Owner ou Editor como default = poder demais; "Viewer mais restritiva" ainda é basic, abrange tudo do projeto; custom para usuários individuais viola "grupo > usuário"
- **Mnemônico:** "Predefinida > Custom > Basic. Group > User. Menor escopo possível."

---

### [1.1 — IAM] Knowledge Check Q2 (2026-05-11): basic role mais baixa que altera estado
- **Errei porque:** escolhi "Administrator", que **não existe** como basic role no GCP
- **Correto:** **Editor** (`roles/editor`)
- **Por que as outras estão erradas:**
  - "Administrator" — não é basic role; só aparece em nome de predefinidas tipo `storage.objectAdmin`
  - **Viewer** — só lê, não altera estado
  - **Owner** — altera estado, mas é a mais ALTA (Editor é a mais baixa que altera)
- **Mnemônico:** **Viewer (Vê) → Editor (Edita) → Owner (Onipotente).** Apenas três basic roles. Em ordem.

---

### [1.1 — Interfaces GCP] Knowledge Check Q1 (2026-05-11): interface para scripting CLI [acertei, mas registro para fixar]
- **Acertei.** Resposta correta: **Cloud Shell**
- **Conceito-chave:** "set of command line executables" = `gcloud`, `gsutil`, `bq`, `kubectl`. Eles vivem no Cloud SDK / gcloud CLI (instalado localmente) ou no Cloud Shell (browser, pré-instalado e autenticado)
- **Distinção importante:** Cloud Shell ≠ gcloud CLI. Cloud Shell é o AMBIENTE; gcloud CLI é o conjunto de EXECUTÁVEIS. O Cloud Shell tem o gcloud CLI pré-instalado
- **Outras 3 interfaces (não-CLI):** Cloud Console (web UI), REST API / Client Libraries (programático), Cloud Mobile App (Android/iOS)

---

### [2.x — Networking] Quiz (2026-05-11): Load balancing regional low-cost para app de 3 camadas
- **Resposta correta:** **Standard tier regional external Application Load Balancer** (frontend, internet → web tier) + **regional internal Application Load Balancer** (backend, web tier → backend)
- **Por que:** tráfego local (Minneapolis/us-central1) → não precisa de Premium tier (global). Standard = mais barato. External ALB para tráfego da internet; Internal ALB para comunicação interna entre camadas
- **Por que as outras estão erradas:**
  - Opção B: internal proxy NLB como frontend → frontend precisa ser **external**, não internal
  - Opção C: global external + premium tier → caro e desnecessário para tráfego regional
  - Opção D: premium tier global → caro, tráfego é local
- **Mnemônico:** "Local traffic = Standard tier regional. Internet-facing = External. Between tiers = Internal."

---

### [2.x — Data] Quiz (2026-05-11): análise de projeção de vendas trimestrais com SQL
- **Resposta correta:** **BigQuery**
- **Por que:** workload **analítico** em larga escala + analistas familiarizados com SQL. BigQuery é data warehouse SQL-native, projetado para analytical queries
- **Por que as outras estão erradas:**
  - Cloud SQL → OLTP (transacional), não escala para analytics massivos
  - Spanner → banco relacional global distribuído, foco em consistência transacional
  - Firestore → NoSQL, sem suporte nativo a SQL analítico
- **Mnemônico:** "Análise + SQL + grande escala = BigQuery. Sempre."

---

### [2.x — Networking] Quiz (2026-05-11): qual load balancer opera em Layer 7?
- **Resposta correta:** **Global Application Load Balancer**
- **Por que:** Layer 7 = camada de aplicação (HTTP/HTTPS). "Application" no nome = Layer 7
- **Por que as outras estão erradas:** Network Load Balancers (regional passthrough, global proxy, regional internal proxy) operam em **Layer 4** (TCP/UDP)
- **Mnemônico:** "**Application** LB = Layer **7**. **Network** LB = Layer **4**."

---

### [2.x — Compute] Quiz (2026-05-11): foco em código, desenvolvimento rápido, portabilidade
- **Resposta correta:** **Package to container image → Cloud Run**
- **Por que:** palavra-chave **"portable"** → containers são portáveis por natureza. Cloud Run = serverless, deploy rápido, sem gerenciar infra
- **Por que as outras estão erradas:**
  - Cloud Run functions → serverless mas menos portável (vendor-specific)
  - Compute Engine VM → exige gerenciar infra, não é rápido
  - GKE + kubectl → mais complexo, não é "foco em código rapidamente"
- **Mnemônico:** "Portável = Container. Rápido sem infra = Cloud Run."

---

### [2.x — Compute] Quiz (2026-05-11): microservices, controle total de containers, sem gerenciar control plane
- **Resposta correta:** **Google Kubernetes Engine (GKE)**
- **Por que:** GKE gerencia o control plane automaticamente. Você tem controle total sobre containers, autoscaling e confiabilidade. Projetado para microservices
- **Por que as outras estão erradas:**
  - Compute Engine → você gerencia tudo (inclusive infra de SO)
  - App Engine → PaaS, pouco controle sobre containers
  - Cloud Run → serverless, menor controle sobre orquestração de containers
- **Mnemônico:** "Controle de container + sem control plane = GKE."

---

### [2.x — Storage] Quiz (2026-05-11): dashboards históricos, time-based, analítico (escolha 2)
- **Resposta correta:** **BigQuery + Bigtable**
- **Por que:**
  - **BigQuery** → data warehouse SQL, perfeito para analytics e dashboards visuais
  - **Bigtable** → wide-column NoSQL, projetado para dados de **séries temporais** (time-series) em alta escala
- **Por que as outras estão erradas:**
  - Firestore → NoSQL documentos, não é analítico
  - Cloud SQL → OLTP, não escala para analytics histórico massivo
  - Cloud Storage → armazenamento de objetos, não é banco de dados analítico
- **Mnemônico:** "Histórico + analítico = BigQuery. Time-series em escala = Bigtable."

---

### [2.x — Compute] Quiz (2026-05-11): dependências específicas de SO
- **Resposta correta:** **Compute Engine (VMs)**
- **Por que:** único serviço que dá controle **total** sobre o sistema operacional. Você escolhe, configura e personaliza o SO completamente
- **Por que as outras estão erradas:**
  - GKE / Cloud Run → containers compartilham o kernel do host, sem controle total do SO
  - App Engine → PaaS, zero acesso ao OS
- **Mnemônico:** "Controle de OS = Compute Engine. Sempre."

---

### [2.x — Compute] Quiz (2026-05-11): Ubuntu customizado, menor tempo, mínimas mudanças de código
- **Resposta correta:** **Compute Engine VMs** (lift-and-shift)
- **Por que:** lift-and-shift puro = zero mudança de código. Custom machine images suportam qualquer customização de Ubuntu
- **Por que as outras estão erradas:**
  - Kubernetes → exige containerização + orquestração, alto esforço
  - App Engine → exige adaptar o app ao modelo App Engine, possíveis mudanças de código
  - Cloud Run → exige containerização (Dockerfile), etapa extra
- **Mnemônico:** "Ubuntu customizado + zero mudança de código = Compute Engine lift-and-shift."

---

### [2.x — Storage] Quiz (2026-05-11): storage class para análise frequente de grandes volumes de dados
- **Acertei.** Resposta correta: **Standard**
- **Ordem das classes (mais → menos acessado):**
  | Classe | Frequência | Obs |
  |---|---|---|
  | Standard | Frequente/contínuo | Sem taxa de recuperação |
  | Nearline | ~1x/mês | Taxa por acesso |
  | Coldline | ~1x/trimestre | Taxa maior |
  | Archive | ~1x/ano | Taxa bem alta |
- **Mnemônico:** "**S**empre → **N**ovamente → **C**om → **A**traso = Standard → Nearline → Coldline → Archive"

---

### [5.2 — Service Accounts] Quiz (2026-05-12): cenário correto para usar Service Account
- **Acertei, mas hesitei entre A (individual GKE pods) e C (interactive analysis).**
- **Resposta correta:** **A — individual GKE pods** (mecanismo: Workload Identity vinculando KSA ↔ GSA, sem chaves expostas)
- **Por que C é armadilha:** "interactive" = HUMANO no teclado rodando queries / abrindo console. Humano usa **suas próprias credenciais** (`gcloud auth login`), nunca SA
- **Por que B erra:** dados de usuário em nome do usuário → **OAuth 2.0 user credentials**, não SA
- **Por que D erra:** devs locais → `gcloud auth application-default login` com a conta deles; distribuir chave de SA para dev é anti-padrão (vaza no Git, perde rastreabilidade)
- **Regra-mãe:** **humano no loop → credencial humana; código/container/VM/batch → Service Account**
- **Mnemônico:** "**I**nteractive = **I**ndivíduo (humano). **P**od = **P**rocesso (máquina)."

---

### [5.2 — Auth client-side] Quiz Q4 (2026-05-12): mobile app lendo Pub/Sub privado
- **Errei (não tinha ideia, precisei de eliminação guiada).**
- **Resposta correta:** **OAuth 2.0 user credentials** — usuário loga, consente, app recebe access token curto (1h) emitido em nome do usuário
- **Por que as outras erram:**
  - **API key** → só identifica projeto chamador para APIs PÚBLICAS (Maps, Translate). Não autoriza dado privado, não autentica usuário
  - **Service account key** → embarcar chave JSON no APK = desastre. Decompilação extrai a chave, vaza para sempre
  - **Environment provided service account** → só funciona dentro de **ambiente Google Cloud** (VM/GKE/Cloud Run via metadata server). Celular do usuário não tem metadata server
- **Matriz mestre — onde o código roda determina a credencial:**
  | Onde roda | Credencial |
  |---|---|
  | Celular/browser + dado do usuário | OAuth 2.0 user credentials |
  | Celular/browser + API pública | API key |
  | VM/GKE/Cloud Run/Function | Environment-provided SA |
  | Servidor on-premises | Workload Identity Federation |
- **Regra de ouro:** client-side **nunca** embarca secret de longa duração. Sempre token curto via OAuth
- **Mnemônico:** "Cliente = OAuth. Servidor GCP = SA do ambiente. Nunca chave JSON em APK."
- **Nuance arquitetural (do PDF oficial):** o app mobile **não** chama Pub/Sub diretamente. O fluxo é: Mobile → OAuth (token do usuário) → **backend seguro** → backend usa **própria SA** → Pub/Sub. A resposta da DQ continua OAuth, mas saiba que existe um backend no meio na arquitetura completa

---

### [5.2 — Auth interna GCP] Quiz (2026-05-12): app DENTRO do GCP autenticando em service APIs do GCP
- **Errei porque:** escolhi "Locally stored keys" interpretando "internal to Google Cloud environment" como "minhas chaves ficam guardadas internamente no ambiente". O enunciado fala da **localização da APLICAÇÃO** (roda dentro do GCP), não da localização da chave
- **Resposta correta:** **Temporary credentials** — tokens OAuth2 curtos (~1h) entregues pelo metadata server quando uma SA está anexada ao recurso (GCE/GKE/Cloud Run/Cloud Functions/App Engine). Rotação automática, zero secret em disco, ADC busca sozinho
- **Por que as outras estão erradas:**
  - **Locally stored keys** = chave JSON de SA no disco. O próprio Google diz "avoid service account keys whenever possible" — é fallback para apps **fora** do GCP que não têm metadata server. Usar dentro do GCP é trocar o seguro automático por arquivo estático sem rotação
  - **API keys** = só **identificam** o projeto (quota/billing), não **autenticam** principal. Maioria das APIs Cloud rejeita. Não dá pra anexar role IAM a uma API key
  - **User account credentials** = identidade de **humano** (`gcloud auth login`). Nunca pra app em produção: atrelada a pessoa, herda permissions amplas, exige fluxo interativo
- **Sinônimo importante:** "Temporary credentials" = "Environment-provided service account" (metadata server) — terminologia varia, conceito é o mesmo
- **Mnemônico:** "App dentro do GCP → SA anexada → metadata server → token temporário. Chave em disco só quando NÃO dá pra usar metadata server."
- **Matriz pra decorar:**
  | Quem? | Onde roda? | Credencial |
  |---|---|---|
  | Humano | Laptop | User account |
  | App | **Dentro GCP** | **Temporary credentials (metadata server)** ✅ |
  | App | Fora GCP | Workload Identity Federation > SA key |
  | App público | Qualquer | API Key (só APIs que aceitam) |

---

### [3.5 — Networking] Quiz "Virtual Networks" Q2 (2026-05-12): mínimo de IPs de uma VM
- **Errei porque:** marquei "Two: one internal + one external", assumindo que toda VM precisa de IP externo para funcionar/acessar internet
- **Correto:** **One: only an internal IP address** — externo é **opcional**
- **Por que externo não é obrigatório:**
  - Saída para internet (apt, docker pull, APIs) → roteada via **Cloud NAT** sem precisar de IP externo na VM
  - Acesso admin → **IAP TCP Forwarding** (túnel autenticado) em vez de SSH/RDP no IP público
  - Comunicação entre VMs internas → IP interno via VPC
- **Por que as outras erram:**
  - Two (interno + externo) → senso comum errado; externo é opcional
  - Three (interno + externo + alias) → alias é específico (ex.: GKE pods), opcional também
- **Padrão de produção:** VM criada com `--no-address` é o **default**. IP externo é exceção justificada.
- **Mnemônico:** "VM precisa de endereço **interno para existir**, externo só se for **conversar com fora** — e mesmo assim NAT + IAP cobrem 90% dos casos."

---

### [5.1 — Custom Roles] Quiz Q5 (2026-05-12): adicionar permissions a custom role existente
- **Errei porque:** escolhi "Delete the custom role and recreate" — pensei que recriar com novas permissions seria limpo
- **Resposta correta:** **Editar a definição da role localmente e rodar `gcloud iam roles update`**
- **Por que deletar/recriar quebra tudo:** todos os IAM bindings que apontavam para a role viram inválidos imediatamente. Todo principal (user/group/SA) que tinha a role **perde o acesso**. E o ID da role fica em quarentena 7-37 dias após delete — não pode reusar de cara
- **Por que as outras erram:**
  - "Criar nova role + migrar usuários" → trabalho duplicado, polui catálogo, janela de inconsistência
  - "Copiar + adicionar + deletar antiga" → o passo "deletar antiga" tem o MESMO problema do B (quebra bindings)
- **Fluxo correto:**
  ```bash
  gcloud iam roles describe ROLE_ID --project=PROJECT_ID --format=yaml > role.yaml
  # editar role.yaml adicionando as novas permissions
  gcloud iam roles update ROLE_ID --project=PROJECT_ID --file=role.yaml
  ```
- **Mnemônico:** "Custom role = **update**, nunca delete. Delete quebra bindings; update preserva tudo."

---

### [1.2 — Billing] Quiz Core Services Q3 (2026-05-14): o que acontece quando budget de $500 atinge 100%
- **Errei porque:** marquei "Todas as atividades do projeto serão suspensas". Intuição de que "se eu defini um teto, o teto é enforced". Outras clouds reforçam essa intuição errada (AWS Budgets actions, Azure Cost Management).
- **Correto:** **um email é enviado ao Billing Admin.** Budget alert é **apenas notificação**, **NUNCA** suspende serviço. A fatura continua subindo se ninguém agir.
- **Por que as outras estão erradas:**
  - "Nada acontece" → falso, o email É enviado
  - "Suspende todas as atividades" → ❌ ARMADILHA — alert nunca corta serviço
  - "Período de cortesia de 4 horas" → **inventado**, não existe
- **Como de fato cortar gasto (pra prova):** cadeia **Budget alert → Pub/Sub topic → Cloud Run function → desabilitar billing/deletar recurso**. Você (engenheiro) escreve a lógica; o GCP não faz por padrão.
- **Mnemônico:** "**No GCP, budget é régua, não disjuntor.** Alert = email. Cortar = você programa via Pub/Sub + Function."

---

### [4.6 — Cloud Trace] Quiz Observability Q1 (2026-05-14): propósito do Cloud Trace
- **Errei porque:** escolhi "reporting on resource consumption" — confundi com Cloud Monitoring (que mede CPU, memória, I/O)
- **Correto:** Cloud Trace = **reporting on latency** (quanto tempo cada parte de uma request leva, entre serviços)
- **Por que as outras estão erradas:**
  - Resource consumption → **Cloud Monitoring** (métricas de infraestrutura)
  - System errors → **Cloud Logging** + **Error Reporting**
  - Application errors → **Error Reporting** (agrega stack traces)
- **Mnemônico:** "**Trace = tempo** (latência). Nunca erros, nunca CPU. Trace rastreia o *caminho* e o *tempo* da request."

---

### [4.6 — SRE] Quiz Observability Q2 (2026-05-14): fundamento do SRE do Google
- **Errei porque:** escolhi "testing and release procedures" — associei SRE a DevOps/CI-CD automaticamente
- **Correto:** o fundamento do SRE é **monitoring** — sem visibilidade do sistema você não tem SLO, não tem error budget, não tem nada
- **Por que as outras estão erradas:**
  - Testing/release → práticas de engenharia/DevOps, não o fundamento de *operar* o sistema
  - Capacity planning → consequência do monitoring, não o fundamento
  - Root cause analysis → você só faz RCA *depois* que o monitoring alertou
- **Sequência mental SRE:** Monitoring (vejo) → Alerting (aviso) → Incident response (ajo) → RCA (aprendo) → Capacity planning (prevejo)
- **Mnemônico:** "SRE começa com **olhos abertos** (monitoring). Tudo mais vem depois."

---

## ⚠️ Labs pendentes para revisitar

### [Lab pendente — 2.2 Cloud Storage CSEK] Qwiklabs "Cloud Storage features", Tasks 3 e 4 (2026-05-14)
- **Status:** parcialmente completo — Tasks 1, 2, 5, 6, 7 OK; **Tasks 3 e 4 bloqueadas pelo ambiente Qwiklabs**.
- **O que foi bloqueado:** upload com Customer-Supplied Encryption Key (CSEK) e rotação via `gsutil rewrite -k`.
- **Por que bloqueou:** Organization Policy do projeto de aula enforcing Google-managed encryption only. Erro retornado:
  ```
  PreconditionException: 412 Requested encryption type for object is not
  compliant with the bucket's encryption enforcement configuration.
  ```
- **Diagnóstico confirmado:** `gcloud storage buckets describe` retornou `encryption: null` (bucket limpo) — bloqueio veio de policy mais alta, fora do alcance do aluno.
- **Lição que importa pra feira:** CSEK está sendo aposentado em ambientes managed. Perder a chave = perder o dado pra sempre. **Indústria prefere CMEK (Cloud KMS)** pelo mesmo controle com rotação gerenciada e recovery possível. Quando o enunciado da DQ disser "we manage our own keys outside GCP" → CSEK; "we want to control rotation and revocation" → CMEK.
- **Bugs operacionais que cometi (não esquecer):**
  1. Digitei `x` por engano no nano e salvei → `.boto` corrompeu → `gsutil` inteiro quebrou. Recovery: `rm .boto && gsutil config -n`.
  2. Ctrl+W do nano fecha aba do navegador. **Solução:** usar `sed -i` direto no shell, ou abrir Cloud Shell em janela própria.
  3. `$variavel` ≠ `nome-literal`. `gs://$qwiklabs-...` expande `$qwiklabs` (vazio) → vira `gs:///-...` → erro `Invalid bucket name`.
  4. Cloud Shell perde variáveis entre sessões → sempre re-exportar e confirmar com `echo $VAR`.
- **Como completar antes da feira:** re-executar o lab fora do Qwiklabs (conta pessoal com créditos GCP). Documentação completa do que **seria** feito está em `modulos/02-planning/2.2-cloud-storage.md` na seção `## Labs > Task 3 e Task 4`.
- **Comando-mestre da rotação (decorar mesmo sem ter feito na prática):**
  ```bash
  gsutil rewrite -k gs://<bucket>/<objeto>
  ```
  Lê o objeto com `decryption_key1` (chave antiga), escreve in-place com `encryption_key` (chave nova), sem download nem mudança de URL/generation.
