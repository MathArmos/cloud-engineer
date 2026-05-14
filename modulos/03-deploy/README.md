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
- [3.2-gke.md](3.2-gke.md) — Autopilot vs Standard, Zonal vs Regional, VPC-native, kubectl básico. DQ 03 resolvida.

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
- [3.5-networking-resources.md](3.5-networking-resources.md) — modos de VPC, firewall por tag, IPs de VM. Quiz "Virtual Networks" (2026-05-12) resolvido (Q1 ✅, Q2 ❌, Q3 ✅).

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
- [3.6-terraform.md](3.6-terraform.md) — Por que IaC, ferramentas (Terraform, Config Connector, Helm, Cloud Foundation Toolkit), lifecycle init→plan→apply→destroy. DQ 10 resolvida.

---

## Como vamos documentar

Cada arquivo: conceito → por que → quando usar → armadilha de prova → exemplo `gcloud`/`kubectl`.
Erros de DQ → [../../flashcards.md](../../flashcards.md).
