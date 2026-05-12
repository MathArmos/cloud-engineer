---
name: ACE Section 3 Detailed Study Notes
description: Notas detalhadas da Seção 3 (Deploying and Implementing a Cloud Solution) do PDF oficial — 6 objetivos completos com comandos, defaults, pegadinhas e detalhes técnicos
type: reference
originSessionId: 89cf7b4e-5dd9-4df8-b755-099227159369
---
# Seção 3 — Deploying and Implementing a Cloud Solution

Notas extraídas do PDF oficial **T-GCPACE-m3-l7-en**. Cobre 6 objetivos (3.1 a 3.6).

## 3.1 — Deploying and Implementing Compute Engine Resources

### Escopo
- Launching compute instances (disks, availability policy, SSH keys)
- Criar **autoscaled MIG** usando instance template
- Configurar **OS Login**
- Configurar **VM Manager**

### Machine Types — Categorias

| Categoria | Uso recomendado |
|-----------|-----------------|
| **Standard** | Workloads de uso geral |
| **High Memory** | Workloads com muita RAM |
| **High CPU** | Workloads com muita CPU |
| **Memory Optimized** (M2) | Bancos grandes, in-memory |
| **Compute Optimized** (C2) | Workloads compute-heavy |
| **Shared Core** | Dev/test, baixo custo |
| **Custom** | Quando nenhum predefinido serve |

### Famílias mencionadas no exame

| Família | Característica | Caso típico |
|---------|----------------|-------------|
| **N2** | **Balanced** (CPU + RAM equilibrados) | **DBs médios-grandes** ✅ |
| **E2** | **Cost-optimized** | Workloads gerais com economia |
| **N1** | General purpose com suporte a **GPU** | Workloads com GPU genérico |
| **A2** | Accelerator-optimized | GPU para ML/training |
| **C2** | Compute-optimized | HPC, gaming, simulações |
| **M2** | Memory-optimized | SAP HANA, in-memory DBs |

### GPUs disponíveis

- **NVIDIA®**: A100, T4, V100, P100, P4, K80
- **Restrição**: GPUs só funcionam em **N1 (general-purpose)** ou **A2 (accelerator-optimized)**
- **Zona-dependente**: nem toda zona tem GPU — verifique antes

### Opções de disco

| Tipo | Redundância | Encryption at rest | Snapshotting | Bootable | Use case |
|------|-------------|---------------------|--------------|----------|----------|
| **Persistent disk HDD** | ✅ | ✅ | ✅ | ✅ | Bulk file, custo baixo |
| **Persistent disk SSD** | ✅ | ✅ | ✅ | ✅ | Random IOPS |
| **Local SSD** | ❌ | ✅ | ❌ | ❌ | **High IOPS + low latency** |
| **RAM disk** | ❌ | N/A | ❌ | ❌ | Low latency, risco de perda |

### Detalhes críticos de disco

- **Persistent disks**: armazenamento de rede (separado do hardware da VM)
- **Regional PD**: réplicas em **2 zonas** → protege contra falha zonal
- **Default ao criar VM**:
  - **Console**: balanced SSD
  - **gcloud**: standard HDD
- **Local SSD**:
  - **Ephemeral** — apaga ao **stop ou terminate**
  - **Sobrevive a reboot**
  - Você é responsável por formatar e striping

### Managed Instance Groups (MIGs) — conceitos

| Pilar | Detalhe |
|-------|---------|
| **Availability** | Recria VMs falhas com base no template; health checks **application-based**; **regional MIG** distribui em múltiplas zonas; LB distribui tráfego |
| **Scalability** | Autoscaling policies aumentam/diminuem instâncias com demanda |
| **Automated updates** | Rolling updates (todas eventualmente) ou **canary** (subset para teste) |

### MIG — tipos de update

| Tipo | Comportamento | Quando cai no quiz |
|------|---------------|---------------------|
| **PROACTIVE** ✅ | **Rolling update**, surge=1 automático, **recursos mínimos** | "Atualizar SO de forma automatizada com recursos mínimos" |
| **Opportunistic** | Não é interativo, só atualiza em ações manuais | Quando NÃO quer automação |

### Pegadinhas MIG no quiz

| Pegadinha | Resposta |
|-----------|----------|
| "Update automatizado + recursos mínimos" | PROACTIVE com surge=1 default |
| "Max surge 5" | ❌ NÃO usa recursos mínimos (cria 5 VMs novas de uma vez) |
| "Abandon instances" | ❌ Não é automatizado, instâncias órfãs não são deletadas |
| "Opportunistic" | ❌ Não é interativo (não automatiza) |

### Q1 (oficial) — Migração MySQL com UDFs (rápido + econômico)

**Resposta: C — Compute Engine VM com N2 + instalar MySQL + restore**

Por quê?
- **Cloud SQL NÃO suporta UDFs** ← elimina B
- **N2 = balanced** (recomendado para DB médio-grande)
- **E2 = cost-optimized** (não recomendado para DB médio) ← elimina D
- **Marketplace** = requer config manual extra (não é o mais rápido) ← elimina A

---

## 3.2 — Deploying and Implementing GKE Resources

### Escopo
- Instalar e configurar **kubectl**
- Deploy GKE com configs variadas: **Autopilot, regional, private, GKE Enterprise**
- Deploy de aplicação containerizada

### Conceitos K8s/GKE — visão geral

- **Nodes**: rodam containers, são **Compute Engine VMs**
- **Cluster**: conjunto de nodes + control plane
- **API**: você descreve apps, K8s faz acontecer

### Modos de Cluster — Autopilot vs Standard

| Característica | **Autopilot** | **Standard** |
|----------------|---------------|--------------|
| Provisionamento | Fully-provisioned & managed | Você define e gerencia |
| Cobrança | Por **recursos dos pods** | Por nodes |
| **Localização** | **Regional sempre** | **Zonal ou regional** |
| Gerenciamento | **Nível de pod** | Nível de cluster |
| Image type | Apenas **container-optimized** | Ubuntu, container-optimized, COS, etc |
| Custom packages | ❌ Não suporta | ✅ Suporta |
| Flexibilidade arquitetura | ❌ Limitada | ✅ Total |

### Availability — Zonal vs Regional

| Tipo | Control Plane | Nodes |
|------|---------------|-------|
| **Zonal** | Único, em uma zona | Pode espalhar em múltiplas zonas |
| **Regional** | Múltiplas réplicas em zonas da região | Replicados em **3 zonas** (mudável) |

### Network Routing — VPC-native vs Routes-based

| Tipo | Como funciona |
|------|---------------|
| **VPC-native** | Routing entre pods usa **alias IPs** ✅ obrigatório para Internal HTTP(S) LB |
| **Routes-based** | Usa Google Cloud Routes |

### Network Isolation

| Tipo | Característica |
|------|----------------|
| **Public GKE** | Setup routing de redes públicas para o cluster |
| **Private GKE** | Pods/nodes usam IPs internos, isolados de redes públicas |

### Versions e Features

- Setup: escolher versão específica **OU** release channel (Rapid, Regular, Stable)
- Sem escolha → versão default atual
- **Best practice**: habilitar **auto-upgrade** dos nodes E do cluster
- Features: **Alpha, Beta, Stable** (status de desenvolvimento)

### Q3 (oficial) — Cluster pequeno, piloto, privado, não-HA, flexível para mudar arquitetura

**Resposta: B — Private standard zonal cluster em us-central1-a + default pool + Ubuntu image**

Por quê?
- **Standard pode ser zonal** ✅ (Autopilot é sempre regional)
- **Autopilot us-central1-a** ❌ — Autopilot é regional, us-central1-a é zona
- **Container-optimized** ❌ — não suporta custom packages
- **Autopilot + Ubuntu** ❌ — Autopilot não suporta Ubuntu

---

## 3.3 — Deploying Cloud Run and Cloud Functions Resources

### Escopo
- Deploy de aplicação
- Deploy para receber **Google Cloud events** (Pub/Sub, Cloud Storage object changes, Eventarc)

### Cloud Run — capabilities

| Capability | Detalhe |
|-----------|---------|
| **Tipo** | Serverless container management |
| **Recurso principal** | **Service** — regional, replicado em múltiplas zonas, exposto como endpoint |
| **Scaling** | Por **incoming requests**, escala para zero |
| **Revisions** | Mudanças criam nova revision → traffic splitting para **canary testing** |
| **Base** | Open source — **Knative** |
| **Billing** | A cada **100ms** de uso |
| **Libraries** | Pode usar system libraries do container |
| **Timeout** | **60 minutos** (requests longos) |
| **Concorrência** | Múltiplas requests **concorrentes** por container instance |

### App Engine — Standard vs Flexible

| Característica | **Standard** | **Flexible** |
|----------------|--------------|--------------|
| Runtime | Sandbox com **linguagem específica** | Docker containers em Compute Engine VMs |
| Linguagens | **Restritas** | Mais opções |
| Scale to zero | ✅ | ❌ |
| Native code | ❌ | ✅ |
| Modificar runtime | ❌ | ✅ |
| Startup | **Segundos** | **Minutos** |
| Acesso ao Compute Engine | ❌ | ✅ |
| Deployment | Rápido | Lento (minutos) |
| Use case | Scaling rápido | Native code / customização |

### Cloud Run Functions — capabilities

| Capability | Detalhe |
|-----------|---------|
| **Tipo** | Serverless function execution |
| **Trigger** | Eventos do Cloud (HTTPS request para endpoint) |
| **Scaling** | Por **número de eventos** |
| **Estado** | **Stateless** — persistir externamente (Datastore, Cloud Storage) |
| **Concorrência** | **1 request por instância** — escala criando novas instâncias |
| **Cold start** | Pode definir min instances |
| **Pricing** | Eventos + compute time + memory + network I/O |
| **Use cases** | IoT processing, lightweight ETL |
| **Base image/runtime** | Provido pelo Cloud Run Functions, auto-patched |

### Q4 (oficial) — Container web app, serverless, pay-per-request, custom packages

**Resposta: C — Cloud Run**

| Opção | Por que não? |
|-------|--------------|
| App Engine flexible | ❌ não escala para zero |
| App Engine standard | ❌ não permite custom packages |
| **Cloud Run** ✅ | Serverless, endpoint, sem infra, custom packages OK |
| Cloud Run functions | ❌ não usa containers para deploy da lógica (snippets) |

### Cloud Storage Triggers — formato correto

```bash
gcloud functions deploy NAME --trigger-event google.storage.object.finalize
```

| Event | Válido? | O que dispara |
|-------|---------|---------------|
| **`google.storage.object.finalize`** ✅ | **Único válido** | Write para Cloud Storage **completo** |
| `google.storage.object.create` | ❌ | Não é evento real |
| `google.storage.object.change` | ❌ | Não é evento real |
| `google.storage.object.add` | ❌ | Não é evento real |

---

## 3.4 — Deploying and Implementing Data Solutions

### Escopo
- Deploy de produtos: Cloud SQL, Firestore, BigQuery, Spanner, Pub/Sub, Dataflow, Cloud Storage, AlloyDB
- Carregar dados (CLI, Cloud Storage, Storage Transfer Service)

### Cloud Storage — fluxograma de storage class

```
Structured data? → Use a structured DB service (Spanner, Cloud SQL, etc.)
Unstructured data?
  ├─ Read < 1/year? → Archive
  ├─ Read < 1/90 days? → Coldline
  ├─ Read < 1/30 days? → Nearline
  └─ Reads frequentes → Standard
```

### Storage Classes — comparação

| Class | Min duration | Retrieval charge | Quando usar |
|-------|--------------|------------------|-------------|
| **Standard** | Nenhum | Não | Acesso imediato, dados ativos |
| **Nearline** | **30 dias** | ✅ Sim | Backup mensal, acesso < 1/mês |
| **Coldline** | **90 dias** | ✅ Sim | Backup trimestral |
| **Archive** | **365 dias** | ✅ Sim | Compliance, arquivamento |

### Location types

| Type | O que é |
|------|---------|
| **Regional** | Uma região específica — minimiza latência local |
| **Dual-region** | Par específico de regiões — geo-redundância. **Mesmo continente apenas** |
| **Multi-region** | Área geográfica ampla (US, EU, ASIA) |

### Passos para criar um bucket

1. **Nome**: globalmente único, **sem info sensível**
2. **Location type + option** (regional/dual/multi)
3. **Default storage class** (Standard/Nearline/Coldline/Archive) — pode ser overridden por objeto
4. **Click Create / submit command**

### Comando CLI

```bash
# Criar bucket (gcloud storage substitui gsutil)
gcloud storage buckets create gs://BUCKET_NAME [--location=LOCATION]

# Default sem --location: US multi-region
```

### Pegadinhas Cloud Storage

| Pegadinha | Detalhe |
|-----------|---------|
| `gsutil` | **Sendo descontinuado** (minimally maintained) — prefira `gcloud storage` |
| Sem `--location` | Cria em **US multi-region** por default |
| `--placement` (dual-region) | **Só aceita regiões do mesmo continente** (us-east1 + europe-west2 ❌) |
| ACL grant remove | `gcloud storage objects --remove-acl-grant` — **NÃO cria bucket** |

### Q6 (oficial) — Bucket para NY + SF, sem ACLs

**Resposta: C — `gcloud storage buckets create` SEM `--location`**
- Default = US multi-region → cobre NY + SF, exclui Londres ✅
- Não usa ACLs por default

### Cloud SQL — características

| Aspecto | Detalhe |
|---------|---------|
| **Tipo** | Managed DB |
| **Suporta** | MySQL, PostgreSQL, SQL Server |
| **Gerencia** | Backups, HA, encryption, updates, logging/monitoring |
| **Storage** | Persistent disks em Compute Engine VMs |
| **Network** | **Static IP** |
| **Limitação crítica** | **NÃO suporta UDFs** (User-Defined Functions) |

### Cloud SQL — passos de setup

1. Create instance
2. Select database type
3. Enter name (público — sem PII)
4. Enter password for root user
5. **Select version** ⚠️ **Não pode editar depois!**
6. **Region/zone** ⚠️ **Não pode modificar!**
7. Primary + secondary zone (secondary ≠ primary)
8. Config: machine type, IP (private/public), storage type/capacity, auto-increase

### Cloud SQL — flags de HA

| Flag | Função |
|------|--------|
| **`--availability-type`** ✅ | **Zonal ou regional** (regional = **failover automático**) |
| `--replica-type` | Tipo de read replica (default `read` ou legacy `failover` deprecated) |
| `--secondary-zone` | Opcional, **só válido com availability=regional** |
| `--master-instance-name` | Cria **read replica** (replica dados, NÃO automatiza failover) |

### Q7 (oficial) — Cloud SQL com failover automático em outage zonal

**Resposta: A — `--availability-type`**
- Apenas `--availability-type=regional` fornece **failover automático** para zona standby

### BigQuery — métodos de ingestão

| Método | Custo | Caso típico |
|--------|-------|-------------|
| **Batch load (load job)** | **Grátis** | CSV, JSON, Avro, Parquet, ORC |
| **BigQuery Data Transfer Service** | **Grátis** | **Slowly changing data, agendado** ✅ |
| **Cloud Composer** (Airflow gerenciado) | Pago | Pipelines complexas |
| **bq load + cron** | Grátis | Manual, mais complexo |
| **Connectors** (Spark, Hadoop) | Varia | Big data integrations |
| **Streaming API** | **Pago por volume** | Real-time, queryable imediato |
| **Dataflow + Apache Beam** | Pago (recursos) | ETL complexo, BQ como source/sink |
| **CTAS (Create Table As Select)** | Custo de query | Salvar resultado de query em tabela |

### Formatos suportados em batch load

- CSV, JSON, Avro, Parquet, ORC

### Q8 (oficial) — Slowly changing data → BigQuery, menor custo, menos passos

**Resposta: D — BigQuery Data Transfer Service**
- **Grátis**
- **Encompassed by one command**
- Mais simples do que `bq load + cron` (que é grátis mas mais complexo)
- Streaming API tem custo
- Dataflow tem custo de recursos

---

## 3.5 — Deploying Networking Resources

### Escopo
- Criar VPC com subnets (custom-mode VPC, **shared VPC**)
- Criar **firewall rules e policies** ingress/egress (IP subnets, network tags, service accounts)
- Peering: **Cloud VPN, VPC Network Peering**

### VPC — visão geral

- Parte da rede **software-defined** do Google
- Fornece: conectividade para Compute Engine, internal TCP/UDP LB, Cloud VPN tunnels, distribuição de tráfego para backends

### Tipos de VPC — Auto mode vs Custom mode

| Característica | **Auto mode** | **Custom mode** |
|----------------|---------------|-----------------|
| Subnets | 1 por região, **automático** | Você cria nas regiões que quiser |
| Novas regiões | Subnets criadas auto | Você cria manualmente |
| IP ranges | **Predeterminados** | **Você controla totalmente** |
| Use case | Easy setup, dev/test | **Produção** ✅ |
| Default VPC | É um auto mode | — |
| Risco | Pode sobrepor com on-prem | Você controla para evitar |

### Conversões

| De → Para | Permitido? | Detalhe |
|-----------|-----------|---------|
| Auto → Custom | ✅ | Mantém IPs atuais, exige passos extras |
| **Custom → Auto** | ❌ | **Caminho unidirecional** |

### Default Project Network

- É um **auto mode network**
- Criado automaticamente em todo project novo

### Q9 (oficial) — VPC com controle total de IPs e subnets regionais

**Resposta: C — Custom mode network**
- Único tipo que dá controle **total** de regions + IP ranges
- Default e Auto mode são predeterminados
- Auto convertido para custom retém IPs antigos (não é "criar com controle total")

---

## 3.6 — Implementing Resources via Infrastructure as Code

### Escopo
- Ferramentas IaC: **Cloud Foundation Toolkit, Config Connector, Terraform, Helm**

### Por que IaC?

- Declarativo: você diz **o quê**, a tool decide o **como**
- Repetível e versionável
- Reset de ambientes dev/test
- Source control (git) sobre infraestrutura
- DevOps best practices

### Terraform — visão geral

- **Open source**
- **Declarativo**
- Config file descreve recursos
- Em GCP: config files vivem em **Cloud Storage bucket com versioning enabled**
- **Cloud Build** submete comandos Terraform via YAML

### Arquivos do projeto Terraform em GCP

| Arquivo | Função |
|---------|--------|
| `cloudbuild.yaml` | Instruções para Cloud Build |
| `backend.tf` | **Remote** Terraform state info |
| `terraform.tfstate` | Local state |
| `main.tf` | Configuração Terraform principal |

### Lifecycle / comandos

```
terraform init  →  terraform plan  →  terraform apply
                                            ↓
                                  terraform destroy (quando descartar)
```

| Comando | O que faz |
|---------|-----------|
| **`terraform init`** | **Baixa última versão do provider** |
| **`terraform plan`** | **Verifica sintaxe + supporting files + preview** dos recursos |
| **`terraform apply`** ✅ | **Cria/configura recursos** declarados no config file |
| **`terraform destroy`** | **Destrói** recursos do config file |

### Q10 (oficial) — O que o `terraform apply` faz?

**Resposta: D — Sets up resources requested in the terraform config file**

Diferenciação clara:
- `init` → baixa provider
- `plan` → preview + sintaxe
- `apply` → **cria recursos**
- `destroy` → destrói recursos

### Outras ferramentas IaC (citadas mas não em quiz)

| Ferramenta | Para que serve |
|-----------|----------------|
| **Cloud Foundation Toolkit** | Templates Google-best-practice para Terraform/Helm |
| **Config Connector** | Gerencia GCP via objetos Kubernetes |
| **Helm** | Package manager para Kubernetes |

---

## Resumo de comandos críticos da Seção 3

```bash
# 3.1 — Compute Engine
gcloud compute instances create NAME --machine-type=n2-standard-4 --zone=ZONE
gcloud compute instance-templates create TEMPLATE --machine-type=...
gcloud compute instance-groups managed create MIG --template=TEMPLATE --size=N
gcloud compute instance-groups managed rolling-action start-update MIG \
   --version=template=NEW_TEMPLATE --type=proactive

# 3.2 — GKE
gcloud container clusters create CLUSTER --zone=us-central1-a --image-type=UBUNTU
gcloud container clusters create-auto CLUSTER --region=us-central1
kubectl apply -f manifest.yaml

# 3.3 — Cloud Run
gcloud run deploy SERVICE --image=IMAGE --region=REGION
gcloud functions deploy NAME --trigger-event google.storage.object.finalize \
   --trigger-resource BUCKET_NAME --runtime python311

# 3.4 — Storage
gcloud storage buckets create gs://BUCKET_NAME    # default: US multi-region
gcloud sql instances create NAME --availability-type=regional --database-version=MYSQL_8_0
bq load DATASET.TABLE FILE schema.json            # ou usar Data Transfer Service

# 3.5 — VPC
gcloud compute networks create NETWORK --subnet-mode=custom
gcloud compute networks subnets create SUBNET --network=NETWORK \
   --region=REGION --range=10.0.0.0/24

# 3.6 — Terraform
terraform init
terraform plan
terraform apply
terraform destroy
```

---

## Top pegadinhas da Seção 3

| Pegadinha | Resposta certa |
|-----------|----------------|
| Migrar MySQL **com UDFs** | Compute Engine + N2 (Cloud SQL **não suporta UDFs**) |
| DB médio-grande, qual machine type? | **N2 (balanced)**, não E2 (cost-optimized) |
| Update SO em MIG, automatizado, mínimos recursos | `--type=proactive` (surge=1 default) |
| Cluster GKE zonal pequeno | **Standard**, não Autopilot (Autopilot é sempre regional) |
| GKE com Ubuntu | **Standard**, não Autopilot |
| Container serverless pay-per-request + custom packages | **Cloud Run** |
| Trigger Cloud Storage para função | `google.storage.object.finalize` (único válido) |
| Bucket que cobre só EUA | Sem `--location` → default US multi-region |
| `gsutil` ou `gcloud storage`? | **`gcloud storage`** (gsutil sendo descontinuado) |
| Failover automático Cloud SQL | `--availability-type=regional` |
| Slowly changing data → BigQuery | **BigQuery Data Transfer Service** (grátis) |
| VPC com controle total de IPs | **Custom mode** |
| Auto VPC → Custom | Permitido (mantém IPs); reverso **não** |
| `terraform apply` faz o quê? | **Cria recursos** (não baixa provider — isso é `init`) |

---

## Cursos oficiais Google Cloud (módulos relevantes — Seção 3)

| Curso | Módulos para Seção 3 |
|-------|----------------------|
| Google Cloud Fundamentals: Core Infrastructure | M3 VMs, M4 Storage, M5 Containers, M6 Applications, M7 Developing/Deploying |
| Architecting with Google Compute Engine (ILT) | M2 Virtual Networks, M3 VMs, M5 Storage, M9 LB+Autoscaling, M10 IaC |
| Essential Google Cloud Infrastructure: Foundation | M2 Virtual Networks, M3 VMs |
| Essential Google Cloud Infrastructure: Core Services | M2 Storage and Database |
| Elastic Google Cloud Infrastructure: Scaling and Automation | M2 LB+Autoscaling, M3 IaC |
| Getting Started with Google Kubernetes Engine | M2 Containers, M3 K8s Architecture |

## Skill Badges relevantes (Section 3)

- **Develop your Google Cloud Network** — `skills.google/course_templates/625`
- **Set Up an App Dev Environment on Google Cloud** — `skills.google/course_templates/637`
- **Build Infrastructure with Terraform on Google Cloud** — `skills.google/course_templates/636`
