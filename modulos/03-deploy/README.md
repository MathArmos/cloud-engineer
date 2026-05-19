# Seção 3 — Deploying and implementing a cloud solution

> **Domínio 3 da prova ACE.** Botar a mão na massa: criar e configurar os recursos via `gcloud`, console ou IaC.

---

## Subseções

### 3.1 — Deploying and implementing Compute Engine resources
**Workbook ACE — Diagnostic Questions:** 01, 02

**Tópicos principais:**
- Criação de instâncias GCE (machine types, images, disks)
- **Cloud SQL como alternativa a VMs** quando o objetivo é apenas hospedar DB
- **Managed Instance Groups (MIGs)** + instance templates
- **Rolling updates de SO:**
  - **Opportunistic** — só atualiza quando há outra operação (poupa recursos)
  - **Proactive** — força atualização imediata

**Arquivos documentados:**
- [3.1-compute-engine.md](3.1-compute-engine.md) — overview do módulo Compute Engine (Essential GCP Infra Foundation): o que é CE, anatomia de uma instância (5 dimensões), famílias de machine type, imagem ≠ disco ≠ instância, bloco "VM access and lifecycle" completo (acesso SSH/RDP, estados RUNNING/STOPPED/SUSPENDED/REPAIRING, reset, live migration, availability policy, shutdown timing 90s vs 30s preemptible, OS patch management, o que pode/não pode em TERMINATED), bloco "Compute Engine Pricing" completo (cobrança 1min mínimo + segundos, resource-based pricing, SUD até 30% automático, CUD 1/3 anos até 57%/70%, Preemptible vs Spot VMs, VM sizing recommendations, Free Tier). **Lab documentado:** Creating Virtual Machines (3 tasks — utility VM sem IP externo, Windows VM com RDP, custom machine type 2 vCPU + 4 GB com Debian).

---

### 3.2 — Deploying and Implementing GKE resources
**Workbook ACE — Diagnostic Questions:** 03

**Tópicos principais:**
- **Tipos de cluster:**
  - **Standard** vs **Autopilot** (Autopilot = Google gerencia nodes)
  - **Zonal** (1 zona) vs **Regional** (multi-zona, alta disponibilidade)
  - **Private** vs **Public**
- Node pools (separação de workloads)
- Image types (Container-Optimized OS vs Ubuntu)

**Arquivos documentados:**
- [3.2-gke.md](3.2-gke.md) — Autopilot vs Standard, Zonal vs Regional, VPC-native, kubectl básico. DQ 03 resolvida. **Curso "Getting Started with Google Kubernetes Engine":** bloco Module 1 (Introduction to Google Cloud) documentado em 6 sub-blocos — (1) 5 traços do cloud computing (NIST-style), (2) comparativo das 5 opções de compute do GCP com lente "containers" (CE/GKE/AE/Cloud Run/Cloud Run functions, incluindo cap de VM 416 vCPUs/12+TB, Persistent Disks até 257 TB, Knative em Cloud Run, free tier perpétuo em CRf), (3) infra global (40% do tráfego mundial, 100+ edge POPs, 7 macro-regiões, 41 regions/124 zones à época do curso), (4) hierarquia de recursos + IAM + Shared Responsibility Model ("if you configure or store it, you're responsible"), (5) billing no nível Project + 4 ferramentas de controle (Budget/Alert/Reports/Quota) + 2 tipos de quota (rate × allocation), (6) 4 formas de interagir com GCP (Console/SDK+Cloud Shell/APIs/Mobile app). Quiz Module 1 (2026-05-17) — 3 ✅ + 1 ⚠️ (Q4 hierarchy pegadinha, flashcard registrado). **Bloco Module 2 (Introduction to Containers and Kubernetes)** documentado em 3 sub-blocos: (7) Vídeo "What is a container?" — escada deploy físico → VM → container, problema multi-app em VM (starvation, dep hell), por que container venceu VM-por-app (kernel compartilhado vs kernels duplicados), definição user-space, 3 benefícios canônicos (code-centric, kernel-reliability, microservices); (8) Vídeo "Containers and container images" — image vs container (receita × bolo), 4 tecnologias Linux (process / namespace=visão / cgroup=consumo / union FS), Linux ≠ K8s namespace, anatomia de layers read-only + container layer efêmera, Dockerfile (FROM/COPY/RUN/CMD) com best practice de ordenação (estável topo, volátil base), multi-stage build, Artifact Registry (pkg.dev) substituindo Container Registry, Cloud Build (build steps em containers Docker, integrado com Source Repos/GitHub/Bitbucket, deploy multi-destino); (9) Vídeo "What is Kubernetes?" — a dor que K8s resolve (descoberta + scheduling + lifecycle em escala), definição canônica (open-source platform para workloads containerizados, 3 capacidades-âncora: orquestrar, escalar microservices, rollout/rollback), arquitetura control plane + nodes (no GKE control plane é gerenciado pelo Google), **armadilha de terminologia "node"** (K8s node = computing instance; GCP node = VM; no GKE coincidem), **declarativo × imperativo** (declarativo = describe estado desejado, auto-healing, estado documentado; imperativo só pra fixes rápidos/exploração), 4 workload types (Deployment/StatefulSet/Job-CronJob/DaemonSet), 3 autoscalers (HPA réplicas / VPA tamanho do pod / Cluster Autoscaler nodes), resource requests (mínimo, usado pelo scheduler) × limits (máximo, runtime mata se excede), CRDs como mecanismo de extensão, portabilidade no-vendor-lock-in (mesmo YAML em GKE/EKS/AKS/on-prem). Tabela consolidada com 14 armadilhas de prova do Module 2. **Sub-bloco 10 do Module 2** (transição → Module 3): vídeo "What is GKE?" — definição operacional de GKE managed K8s (COS, auto-upgrade, node auto-repair, cluster autoscaler), 4 features automáticas com armadilhas (auto-upgrade vs node auto-repair vs cluster autoscaler vs HPA), intro Autopilot (cross-ref pra seção detalhada), 5 integrações nativas (Cloud Build/Artifact Registry/IAM/Cloud Observability/VPC). **Quiz Module 2 (2026-05-18)** — 3 questões, 3 ✅ após reconsideração: Q1 (declarative inicialmente "Virtualization" → flashcard), Q2 (nodes inicialmente "Containers" → flashcard), Q3 (layered filesystem resolvida após explicação). **Bloco Module 3 (Kubernetes Architecture)** em 10 sub-blocos (2026-05-18): (1) 2 conceitos centrais (object model + declarative management, recap aprofundado), (2) anatomia de Object com spec/status (você escreve spec; control plane preenche status — pegadinha), (3) Watch Loop com diagrama de fluxo + imperativo vs declarativo, (4) Pod como building block (1 IP único, network namespace compartilhado, localhost intra-pod, volumes compartilháveis, diagrama de pod com 2 containers), (5) anatomia control plane vs nodes (hub-and-spoke pelo apiserver, diagrama ASCII), (6) **5 componentes do control plane** detalhados — kube-apiserver (auth+authz+admission, única porta), etcd (key-value Raft, só apiserver toca), kube-scheduler (decide node mas NÃO lança pod, considera recursos/constraints/affinity/anti-affinity/taints+tolerations), kube-controller-manager (workload controllers Deployment/ReplicaSet/StatefulSet/DaemonSet + system Node Controller + outros Job/CronJob), kube-cloud-manager (LBs e PDs no GCP), (7) **3 componentes do node** — kubelet (agente, executa probes liveness/readiness/startup), container runtime (containerd no GKE, Docker deprecated em 1.24), kube-proxy (iptables, regras de Service), (8) GKE como abstração — comparativo self-managed vs GKE em 6 componentes, quando Standard ainda faz sentido (DaemonSet privilegiado, GPU/TPU, hostPath/hostNetwork), (9) 13 pegadinhas consolidadas, (10) mnemônicos ASEC-C / KRP / "Scheduler nomeia, Kubelet executa" / "Tudo pelo APIserver".
- [lab-cloud-build-quickstart.md](lab-cloud-build-quickstart.md) — **Lab Module 2 do curso GKE** (2026-05-18) — Cloud Build + Artifact Registry. 4 tasks documentadas: (1) habilitar APIs Cloud Build + Artifact Registry; (2) build "rápido" via `gcloud builds submit --tag` (Dockerfile alpine + quickstart.sh) com criação de repo Docker (`gcloud artifacts repositories create --repository-format=docker --location=REGION`); (3) build via `cloudbuild.yaml` com 1 step explícito (`gcr.io/cloud-builders/docker`, args + images); (4) build de 2 steps onde **a image recém-construída vira o executor do próximo step** com `args: ['fail']` simulando teste que falha, demonstrando fail-fast e propagação de exit code (`echo $?`). Conceitos: Artifact Registry (`pkg.dev`) substituindo Container Registry deprecado, `$PROJECT_ID` (substitution do Cloud Build) × `${DEVSHELL_PROJECT_ID}` (env var do Cloud Shell) — armadilha real, substitution variables built-in (`$BUILD_ID`, `$COMMIT_SHA`), Alpine + musl libc, campo `images:` obrigatório pra pushar. Tabela de 8 pegadinhas de prova. Cheatsheet completo de comandos + diagrama de fluxo Cloud Shell → Cloud Build → Artifact Registry.
- [lab-getting-started-gke-intro.md](lab-getting-started-gke-intro.md) — **Lab Module 1 do curso GKE** (2026-05-17). Walkthrough completo: Cloud Shell Editor (Theia/VS Code) editando `cleanup.sh` e criando `index.html`; `git clone` do repo `orchestrate-with-kubernetes`; `gcloud compute scp` do Cloud Shell pra VM `first-vm` (us-east1-c) com geração de chave SSH na 1ª execução; instalação de nginx via apt em VM Debian; cópia do `index.html` pro document root `/var/www/html`. **Troubleshooting documentado:** (a) `git clone` falhou em `/home` (não tem write access — fix: `cd ~`); (b) `gcloud compute scp` rodado por engano **dentro da VM** retornou `insufficient authentication scopes` (fix: rodar no Cloud Shell, não na VM); (c) browser mostrou ícone quebrado porque o HTML servia a URL **placeholder** do enunciado em vez da Public URL real do bucket Qwiklabs.

---

### 3.3 — Deploying and implementing Cloud Run and Cloud Run functions resources
**Workbook ACE — Diagnostic Questions:** 04, 05

**Tópicos principais:**
- **Cloud Run vs App Engine vs Cloud Run functions:**
  - Cloud Run — containers, pay-per-request, scale to zero
  - App Engine standard — sem container, runtimes fixos
  - App Engine flexible — containers em VMs, sem scale to zero
  - Cloud Run functions — código curto, event-driven
- **Triggers de Cloud Run functions com Cloud Storage:**
  - `google.storage.object.finalize` (objeto criado/finalizado)
  - outros eventos: delete, archive, metadataUpdate

**Arquivos documentados:**
- [3.3-cloud-run.md](3.3-cloud-run.md) — Cloud Run vs App Engine (Standard/Flexible) vs Cloud Run Functions, trigger `google.storage.object.finalize`, comparação de concorrência. DQs 04 e 05 resolvidas.

---

### 3.4 — Deploying and implementing data solutions
**Workbook ACE — Diagnostic Questions:** 06, 07, 08

**Tópicos principais:**
- **Buckets:** comandos `gcloud storage buckets create`, `gsutil mb`, escolha de location (regional / dual-region / multi-region)
- **Cloud SQL HA:** `gcloud sql instances create --availability-type=REGIONAL` para failover automático
- **Carregar dados no BigQuery:** Data Transfer Service vs `bq load` script vs streaming API vs Dataflow — escolher pelo custo/simplicidade

**Arquivos documentados:**
- [3.4-data-solutions.md](3.4-data-solutions.md) — Storage classes, location types, Cloud SQL HA (`--availability-type`), BigQuery ingestão (Data Transfer Service vs Streaming vs Dataflow). DQs 06, 07 e 08 resolvidas.

---

### 3.5 — Deploying and implementing networking resources
**Workbook ACE — Diagnostic Questions:** 09

**Tópicos principais:**
- **Modos de VPC:**
  - **Default** — criada com o projeto, subnets em todas as regiões
  - **Auto mode** — subnets criadas automaticamente em cada região com ranges fixos
  - **Custom mode** — você define tudo (IP ranges, subnets, regiões) → **controle total**
- Subnets regionais, IP ranges
- Firewall rules por tag/SA vs por IP
- IPs de VM (interno obrigatório, externo opcional, alias IPs)

**Arquivos documentados:**
- [3.5-networking-resources.md](3.5-networking-resources.md) — modos de VPC, firewall por tag, IPs de VM. Quiz "Virtual Networks" (2026-05-12) resolvido (Q1 ✅, Q2 ❌, Q3 ✅). **Cloud VPN:** Classic VPN (99.9% SLA, MTU 1460, site-to-site, 2 túneis, IKEv1/v2) vs HA VPN (99.99% SLA, 2 interfaces/IPs fixos, somente BGP, 2 ou 4 túneis, topologias: 2 peers / AWS / HA↔HA). Cloud Router e BGP (link-local 169.254.0.0/16, propagação automática de rotas). **Lab "Configuring Google Cloud HA VPN":** setup completo end-to-end (2 VPCs, 2 HA VPN gateways, 2 Cloud Routers com ASN, 4 túneis IKEv2, BGP peering por túnel com IPs 169.254.x.x/30, firewall cross-VPC, global routing mode, teste de failover com tunnel0 derrubado).
- [3.5-load-balancers.md](3.5-load-balancers.md) — **Curso "Elastic Google Cloud Infrastructure: Scaling and Automation"** (LB + autoscaling de MIGs). Blocos documentados: **(1) Introdução** — Cloud Load Balancing como serviço fully distributed/software-defined/managed (não é instance-based, single anycast IP global, 1M+ QPS); Application LB Layer 7 (HTTP(S) headers, cookies, URL paths, SSL/TLS termination, session affinity, content-based routing) vs Network LB Layer 4 (IP/porta, TCP/UDP, baixa latência, alta vazão, health checks). **(2) Managed Instance Groups** — definição (VMs idênticas via template), 4 superpoderes (rolling updates, autoscaling, LB integration, auto-healing), regional vs zonal (regional é o default recomendado para HA), fluxo de criação (template → MIG), 7 decisões na criação (stateless/stateful, nome, zona, port name mapping, template, autoscaling, health check), armadilhas (template imutável, auto-healing seletivo, health check duplo healing+LB). **(3) Autoscaling, Health Checks e Stateful IPs** — 4 categorias de policy (CPU / LB capacity / Monitoring metric / queue+schedule), modelo de média do grupo vs target, exemplo do slide 92.5% → +1 VM; 4 parâmetros do health check (check interval, timeout, healthy/unhealthy threshold) com cálculo do exemplo "2 falhas em 15s"; stateful IPs (IPv4 interno e externo, promoção ephemeral → static, diferença vs stateful MIG, cenários: licença por IP, lift-and-shift, peers hardcoded). **(4) Application LB — arquitetura detalhada** — Layer 7, backends suportados (CE, GKE, Cloud Storage, Cloud Run, external via internet/híbrido), External vs Internal; implementação GFE (Global/Classic, multi-region no Premium tier) vs Envoy (Global/Regional, advanced traffic management) e os 3 modos (Global/Classic/Regional); stack de 5 componentes (External Forwarding Rule → Target HTTP(S) Proxy + SSL certs → URL Map → Backend Service [health check, session affinity, timeout 30s fixed, lista de backends] → Backend [instance group + balancing mode + capacity scaler]); balancing mode (CPU vs RPS) × capacity scaler (multiplicador 0.0=drain, 1.0=cheio, 0.5=metade) com os 2 exemplos do vídeo esmiuçados; round-robin default vs session affinity; cross-region failover automático no Global; propagação de minutos. **(5) Decision tree — escolhendo o LB (fechamento da trilha)** — fluxo de 3 perguntas (tráfego → external/internal → global/regional) e tabela 3×4 com as ~6 variantes principais (Application/Proxy/Passthrough × External global/External regional/Internal regional/Cross-region); regra de uma frase por LB (Application = HTTP(S) flexível, Proxy NLB = TLS offload/TCP proxy/external multi-region, Passthrough NLB = preservar source IP, evitar overhead, UDP/ESP/ICMP, expor IP do cliente); load-balancing scheme como atributo da forwarding rule + backend service (`EXTERNAL_MANAGED`, `INTERNAL_MANAGED`, `EXTERNAL`, `INTERNAL`) e o que MANAGED significa (GFE ou Envoy interceptam; não-MANAGED = passthrough direto via Andromeda); Network Service Tiers (Premium obrigatório para LBs globais; Standard só regional); compilado final de armadilhas reunindo decision tree + MIG + autoscaling + Application LB stack. Armadilhas-chave: passthrough sempre regional, backend bucket só no Global external Application LB, preservar IP só com Passthrough, TLS offload em TCP não-HTTP só com Proxy NLB, `EXTERNAL_MANAGED` ≠ `EXTERNAL` (legacy).

---

### 3.6 — Implementing resources through infrastructure as code
**Workbook ACE — Diagnostic Questions:** 10

**Tópicos principais:**
- **Terraform — fluxo básico:**
  - `terraform init` — baixa providers
  - `terraform plan` — preview do que vai ser criado/mudado
  - `terraform apply` — efetivamente cria/atualiza recursos
  - `terraform destroy` — remove recursos
- Conceitos: providers, state, modules
- **Não está no exam guide oficial — opcional**

**Arquivos documentados:**
- [3.6-terraform.md](3.6-terraform.md) — Por que IaC, ferramentas (Terraform, Config Connector, Helm, Cloud Foundation Toolkit), lifecycle init→plan→apply→destroy. DQ 10 resolvida. **Curso "Elastic Google Cloud Infrastructure: Scaling and Automation":** bloco "Introdução — Automating the Deployment of GCP Infrastructure" (imperativo via Cloud API vs declarativo via Terraform, problema das dezenas de calls espalhadas, analogia post-its vs planta arquitetônica) + bloco "Terraform: a ferramenta, HCL e o fluxo init→plan→apply" (escada Console→CloudShell→Terraform, IaC benefícios, 5 ferramentas oficiais GCP — Terraform/Chef/Puppet/Ansible/Packer, paralelismo do Terraform vs sequencial do gcloud, sintaxe HCL com Blocks/Arguments/Expressions, anatomia do main.tf com provider/resource/output, exemplo guiado VPC auto-mode + firewall HTTP 80/8080, detalhes do init/plan/apply — plan faz refresh antes do diff).

---

## Como vamos documentar

Cada arquivo: conceito → por que → quando usar → armadilha de prova → exemplo `gcloud`/`kubectl`.
Erros de DQ → [../../flashcards.md](../../flashcards.md).
