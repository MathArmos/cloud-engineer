# Seção 5 — Configuring access and security

> **Domínio 5 da prova ACE.** Quem pode fazer o quê em qual recurso: IAM aprofundado + service accounts.

---

## Subseções

### 5.1 — Managing Identity and Access Management (IAM)
**Workbook ACE — Diagnostic Questions:** 01 (GKE→Spanner, tipos de identidade), 02 (folder set-iam-policy, least privilege), 03 (custom role update)

**Tópicos principais:**
- **Tipos de identidade:**
  - **Google Account** — humano com email Google
  - **Google Workspace account** — humano em domínio gerenciado por Workspace
  - **Cloud Identity** — Workspace sem o Workspace (só identidade)
  - **Service Account** — identidade para aplicações/recursos (NÃO humano)
- **Hierarquia de IAM bindings:** Org → Folder → Project → Resource (herança descendente)
- **Roles necessárias para gerenciar IAM em diferentes níveis:**
  - Editar IAM de um projeto: `resourcemanager.projects.setIamPolicy`
  - Editar IAM de uma pasta: `roles/resourcemanager.folderIamAdmin`
  - Editar IAM da organização: `roles/resourcemanager.organizationAdmin`
- **Custom roles:**
  - Update local (definição YAML/JSON) + comando de update = forma recomendada
  - NÃO deletar e recriar (perde IAM bindings)

**Arquivos documentados:**
- [5.1-iam-roles.md](5.1-iam-roles.md) — Roles administrativas por nível (organizationAdmin/folderIamAdmin/projectIamAdmin), permission vs role, diagnóstico de erro `set-iam-policy`, princípio "desça na hierarquia até o menor nó que cobre o necessário".

---

### 5.2 — Managing service accounts
**Workbook ACE — Diagnostic Questions:** 04 (qual cenário usar SA — GKE pods), 05 (mobile app + Pub/Sub privado — OAuth)

**Tópicos principais:**
- **Quando usar Service Account:** para **aplicações** acessarem recursos (não para usuários humanos, não para análise interativa, não para devs)
- **Autenticação client-side (apps mobile/browser):**
  - **OAuth 2.0 user credentials** = padrão recomendado para apps que acessam dados privados em nome do usuário
  - **Service account keys** = perigoso em cliente (chave vaza)
  - **API keys** = só para APIs públicas/anônimas (Maps, etc.), NÃO para dados privados
- **Workload Identity (GKE):** vincula Kubernetes Service Account a Google Service Account — forma segura, sem chaves expostas

**Arquivos documentados:**
- [5.2-service-accounts.md](5.2-service-accounts.md) — Criar SA, conceder roles, anexar à VM (cenário Cymbal Superstore: LAMP + Cloud SQL). Identity vs managed resource, ADC via metadata server, default SA pitfalls.

---

## Como vamos documentar

Cada arquivo: conceito → por que → quando usar → armadilha de prova → exemplo prático.
Erros de DQ → [../../flashcards.md](../../flashcards.md).

---

## 📘 Sample study plan — recomendação oficial do Google (6 semanas)

> Plano sugerido pelo próprio Google ao final do módulo de revisão da ACE. Cobre toda a trilha do Skills Boost + skill badges + revisão. Serve como **mapa de referência cruzada** com nosso cronograma de 14 dias.

### Semana 1 — Fundação
- **Google Cloud Fundamentals: Core Infrastructure** — visão geral da plataforma (regions/zones, projects, billing, IAM básico, principais serviços compute/storage/data/ML).
- **Essential Google Cloud Infrastructure: Foundation** — primeiros passos com IAM, Cloud Storage, VMs (gcloud SDK, Cloud Shell, Marketplace).

### Semana 2 — Compute & Networking
- **Essential Google Cloud Infrastructure: Core Services** — VPC, firewall, Cloud Storage avançado, monitoring/logging básico.
- **Elastic Google Cloud Infrastructure: Scaling and Automation** — Cloud Interconnect, load balancing (todos os tipos), autoscaling (MIG), infra como código.

### Semana 3 — Dados, GKE & Load Balancing
- **Select a Google Cloud Database for Your Applications** — Cloud SQL, Spanner, Firestore, Bigtable, BigQuery — quando escolher cada um.
- **Getting Started with Google Kubernetes Engine** — clusters, pods, deployments, services, autopilot vs standard.
- 🏅 **Skill badge:** *Implementing Cloud Load Balancing for Compute Engine* — labs práticos de LB HTTP(S) + MIG.

### Semana 4 — Cloud Run, App Dev & Network
- **Developing Applications with Cloud Run on Google Cloud: Fundamentals** — serverless containers, deploy, scaling to zero.
- 🏅 **Skill badge:** *Set Up an App Dev Environment on Google Cloud* — Cloud Build, Artifact Registry, Cloud Source Repositories.
- 🏅 **Skill badge:** *Develop Your Google Cloud Network* — VPC peering, Shared VPC, Cloud NAT, VPN.

### Semana 5 — Observabilidade & IaC
- **Logging and Monitoring in Google Cloud** — Cloud Logging, Cloud Monitoring, alerting, SLOs.
- **Observability in Google Cloud** — dashboards, métricas customizadas, Cloud Trace, Profiler.
- 🏅 **Skill badge:** *Build Infrastructure with Terraform on Google Cloud* — providers, state, modules, deploy real.

### Semana 6 — Revisão & Simulado
- **Sample questions** — banco oficial de questões de exemplo (formato e dificuldade reais da prova).
- **Review documentation** — releitura da [exam guide](https://cloud.google.com/learn/certification/cloud-engineer) + docs dos serviços fracos identificados nos simulados.

---

### Como esse plano se relaciona com nosso cronograma de 14 dias

| Nosso módulo | Cobre semanas Google |
|---|---|
| 01 — Setup environment | Semana 1 (Foundations) |
| 02 — Planning | Semana 2 (Core/Elastic Infra) + parte da Semana 3 (Databases) |
| 03 — Deploy | Semanas 3 e 4 (GKE, LB, Cloud Run, App Dev, Network) |
| 04 — Operations | Semana 5 (Logging, Monitoring, Terraform) |
| 05 — Access & Security | transversal (IAM aparece desde a Semana 1) |
| **Revisão final (dias 13-14)** | **Semana 6 (sample questions + docs)** |

**Conclusão prática:** o plano do Google é mais espaçado (6 semanas) e dá mais peso a skill badges hands-on. Nosso plano de 14 dias prioriza domínios da prova + Diagnostic Questions do Workbook ACE. Os dois são complementares: use as semanas 3, 4 e 5 do Google como **fonte de labs adicionais** caso sobre tempo, especialmente os 4 skill badges (LB, App Dev, Network, Terraform) — eles são os melhores simuladores práticos do que cai na prova real.

---

## 🧭 Framework oficial do Google — *Create a weekly study plan*

> Antes de montar (ou ajustar) qualquer cronograma semanal, o Google recomenda responder estas **5 perguntas** para garantir que o plano cobre os 4 pilares: teoria, prática, documentação e simulado.

### As 5 perguntas para cada semana de estudo

1. **What exam guide section(s) or topic area(s) will you focus on?**
   → *Qual(is) domínio(s) da exam guide você vai atacar?*
   Os 5 domínios oficiais são: (1) Set up environment, (2) Planning, (3) Deploy & implement, (4) Operations, (5) Access & security. Escolher 1-2 por semana evita pulverizar foco.

2. **What courses (or specific modules) will help you learn more?**
   → *Quais cursos ou módulos específicos te ensinam esse tópico?*
   Mapeia para os cursos do Skills Boost (Core Infrastructure, Essential Infra, Elastic Infra, etc.). Pode ser **curso inteiro** ou **módulo específico** dentro de um curso — não precisa fazer linear.

3. **What Skill Badges or labs will you work on for hands-on practice?**
   → *Quais skill badges ou labs vão dar prática real (hands-on)?*
   Teoria sem mãos no console = reprova. Cada domínio tem skill badges associados — fazer no mínimo 1 lab por semana fixa o conhecimento muscular do `gcloud` e do console.

4. **What documentation links will you review?**
   → *Quais páginas da documentação oficial você vai revisar?*
   Cada serviço (IAM, VPC, GKE, Cloud SQL...) tem páginas de **"best practices"** e **"quotas & limits"** que costumam aparecer textualmente nas alternativas da prova. Salvar 3-5 links por semana.

5. **What additional resources will you use - such as sample questions?**
   → *Que recursos extras você vai usar — ex.: questões de exemplo?*
   Sample questions oficiais, Diagnostic Questions do Workbook ACE, flashcards, simulados de terceiros (ExamTopics, Whizlabs). Sem auto-teste, você só *acha* que sabe.

---

### Aplicando o framework ao nosso cronograma

| Pergunta | Como já aplicamos no nosso plano de 14 dias |
|---|---|
| **1. Exam guide section** | Cada módulo (`01` a `05`) = 1 domínio da exam guide. |
| **2. Cursos/módulos** | [curriculum_and_plan.md](../../) mapeia cada dia a cursos do Skills Boost. |
| **3. Skill Badges/labs** | Documentamos labs feitos em cada módulo + lista de skill badges complementares na seção anterior. |
| **4. Documentation** | Cada `.md` de tópico lista os links oficiais consultados. |
| **5. Recursos extras** | Workbook ACE (DQs) → [flashcards.md](../../flashcards.md) + sample questions na revisão final (Semana 6 Google / Dias 13-14 nosso). |

**Conclusão:** se em algum ponto do estudo bater dúvida "estou fazendo o suficiente?", rode mentalmente as 5 perguntas para a semana atual. Se alguma resposta for *"nenhum"* (ex.: "qual lab? nenhum essa semana"), o plano está incompleto naquele eixo.

---

## 🎯 Calibração antes de planejar — as 4 perguntas-âncora do Google

> Antes das 5 perguntas semanais, o Google manda calibrar o **tamanho** do plano. São 4 perguntas que definem se o cronograma é viável ou fantasia.

| # | Pergunta | Por que importa |
|---|---|---|
| 1 | **When will you take the exam?** | Data marcada vira deadline real. Sem data, o plano vira "estudar até me sentir pronto" = nunca. |
| 2 | **How many weeks does that give you to prepare?** | Define a granularidade — 2 semanas exige foco brutal; 6+ permite cobrir tudo com folga. |
| 3 | **How many hours can you realistically spend each week?** | "Realisticamente" é a palavra-chave. Plano de 20h/semana que vira 5h/semana = atraso garantido. |
| 4 | **How many total hours will you prepare?** | Horas/semana × semanas. Serve para sanity-check vs. carga real do Skills Boost (~40-60h só de vídeos + labs). |

> **Regra do Google:** sempre reservar tempo **no final** do plano para refazer Diagnostic Questions + Sample Questions e tapar lacunas remanescentes. Não é opcional — é onde você descobre o que **acha** que sabe vs. o que **sabe**.

### Aplicando ao nosso caso

- **Exam date:** alinhada com deadline 25/05 (feira de trabalho).
- **Semanas disponíveis:** 14 dias (~2 semanas) — cronograma compactado, sem margem para refazer cursos inteiros.
- **Horas/semana realistas:** alto, dado deadline + dedicação atual.
- **Total horas:** dias 13-14 reservados para revisão = ✅ regra do Google atendida.

---

## 🧩 Dois modelos de estruturação semanal

Google sugere **duas estratégias** legítimas para definir o foco de cada semana — escolha por contexto, não por dogma.

### Modelo A — Foco por curso/skill badge
Cada semana = um curso completo OU um skill badge inteiro.
- ✅ **Quando usar:** começo da preparação, quando você ainda está construindo base. Conhecimento se acumula em blocos coerentes.
- ❌ **Risco:** se um tópico crítico estiver disperso entre cursos, você pode passar semanas sem tocá-lo.

### Modelo B — Foco por tópico
Cada semana = um tema da exam guide (ex.: "configurar VPCs", "IAM avançado").
- ✅ **Quando usar:** reta final, quando você já tem base e precisa caçar **lacunas específicas** identificadas nas Diagnostic Questions.
- ❌ **Risco:** pular pré-requisitos. Não dá para estudar "Load Balancing" sem antes ter VPC firme.

### Padrões dentro de uma semana

Google mostra dois **mix patterns** dentro de uma semana:

1. **Mix híbrido (recomendado para a maioria):** módulos teóricos pontuais + 1 skill badge prático + 2-3 páginas de documentação. Cobre os 4 eixos (teoria, prática, docs, revisão).
2. **Semana monotemática:** uma semana inteira só em curso teórico, próxima semana só em skill badge. Útil para quem precisa de imersão profunda em um eixo de cada vez.

**Nosso plano de 14 dias usa Modelo B + mix híbrido:** cada dia tem 1 tópico da exam guide + leitura oficial + DQs do Workbook + (quando possível) lab. Isso bate com o que Google considera "approach that fits your existing skillset" — assumindo base prévia de desenvolvimento e cloud.

### Princípio iterativo

> *"Remember, you may need to adjust your plans based on the areas where you need to learn more."*

O plano **não é estático**. Após cada bloco de Diagnostic Questions, se uma área aparecer fraca (ex.: erra 3/5 em IAM), **realocar** dias futuros para reforço — não seguir o plano original cegamente. Plano serve à preparação, não o contrário.
