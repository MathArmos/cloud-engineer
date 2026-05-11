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
