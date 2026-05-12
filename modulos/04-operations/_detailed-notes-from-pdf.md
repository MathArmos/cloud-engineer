---
name: ACE Section 4 Detailed Study Notes
description: Notas detalhadas da Seção 4 (Ensuring Successful Operation) do PDF oficial — 6 objetivos completos com comandos, defaults, pegadinhas e detalhes técnicos
type: reference
originSessionId: 89cf7b4e-5dd9-4df8-b755-099227159369
---
# Seção 4 — Ensuring Successful Operation of a Cloud Solution

Notas extraídas do PDF oficial **T-GCPACE-m4-l7-en**. Cobre 6 objetivos (4.1 a 4.6).

## 4.1 — Managing Compute Engine Resources

### Escopo
- Remotely connecting to instance
- Viewing running VM inventory (IDs, details)
- **Snapshots**: create, view, delete, schedule
- **Images**: create from VM/snapshot, view, delete

### Snapshots — características essenciais

| Característica | Detalhe |
|----------------|---------|
| **Natureza** | Incremental (mais barato que imagens) |
| **Aplica-se a** | Apenas **persistent disks** |
| **Armazenamento** | Cloud Storage bucket gerenciado pelo snapshot service |
| **Compressão** | Automática |
| **Localização** | Regional ou multi-regional (afeta custo) |
| **Integridade** | Stored across multiple locations com checksums automáticos |
| **Schedule** | Configurável via `gcloud` + cron, console ou CLI |
| **Restrição de região** | Snapshot schedule e PD-fonte **devem estar na mesma região** |
| **Restore** | Pode ir para PD em **zona/região diferente** → usado para mover VMs |
| **Live snapshots** | Pode criar com a VM rodando |
| **Frequência mínima** | Snapshots do mesmo disco devem ter **≥ 10 minutos de intervalo** |

### Como snapshots incrementais funcionam
- **Snapshot 1** = full backup (contém tudo)
- **Snapshot 2** = só blocos modificados desde Snapshot 1
- **Snapshot 3** = só blocos modificados desde Snapshot 2
- Às vezes (otimização) um novo snapshot vira full para limpar dependências

### Deletando snapshots — pegadinhas
- Ao deletar um snapshot, seus dados são **copiados para snapshots downstream** dependentes → o downstream **aumenta de tamanho**
- **Não dá pra deletar snapshot schedule** que ainda está attached a um PD → faça detach primeiro

### Comandos essenciais

```bash
# Listar
gcloud compute snapshots list --project PROJECT_ID
# Flags: --limit, --regexp, --sort-by, --uri

# Descrever (retorna creation time, size, source disk)
gcloud compute snapshots describe SNAPSHOT_NAME
# Flags: --format, --project, --quiet
```

### Padrão de comandos `gcloud`
> `gcloud <grupo> <subgrupo> <verbo>`
> Verbo (list, describe, delete) **sempre no final**. Verbos disponíveis para snapshots: **list, describe, delete** (não `get`).

---

## 4.2 — Managing Google Kubernetes Engine Resources

### Escopo
- Visualizar inventário do cluster (nodes, pods, services)
- Configurar GKE para acessar Artifact Registry
- Node pools (add, edit, remove)
- Recursos K8s (Pods, Services, StatefulSets)
- Autoscaling horizontal e vertical

### Objetos Kubernetes — relação

```
Deployments
   ↓ manages and monitors
  Pods (replicas)
   ↑ exposes via
Services
```

| Objeto | Função | Detalhes |
|--------|--------|----------|
| **Pod** | Menor unidade implantável | 1+ containers, networking + storage compartilhados, IP interno **mutável** |
| **Deployment** | Gerencia set de pods idênticos | Usa **ReplicaSet** para nº de pods, troca instâncias unhealthy, **rolling upgrade** ao atualizar template |
| **Pod template** | Spec **dentro** de um Deployment | Define labels, container name, image — não é objeto independente |
| **Service** | Grupo de endpoints de pods | **IP estável**, seleciona pods por labels, pode implementar load balancing |

### Load Balancers no GKE — visão completa

| Quero criar | Setting | Camada | Escopo |
|-------------|---------|--------|--------|
| Network LB (externo) | `Service: type: LoadBalancer` | **L4** | Regional |
| External HTTP(S) LB | `Ingress` + `ingress.class: "gce"` | **L7** | Global |
| Internal HTTP(S) LB | `Ingress` + `ingress.class: "gce-internal"` + **NEG annotation** | **L7** | Regional |

### External HTTP(S) LB — detalhes

- Roteamento depende de: **URL path, session affinity, balancing mode dos NEGs**
- Object `kind: Ingress`
- Annotation: `kubernetes.io/ingress.class: "gce"`
- Deployado em **Google Points of Presence (PoPs)**
- Static IP do Ingress **persiste enquanto o objeto existir**

### Internal HTTP(S) LB — detalhes (cai muito!)

- Object `kind: Ingress`
- Metadata: `kubernetes.io/ingress.class: "gce-internal"`
- Proxies em **proxy-only subnet** na região do VPC
- **Apenas NEGs suportados** → annotation obrigatória no Service:
  ```yaml
  cloud.google.com/neg: '{"ingress": true}'
  ```
- Forwarding rule vem do **range do GKE node**
- Cluster precisa ser **VPC-native** (alias IPs nos pods) — não funciona em routes-based cluster

### Service `type: LoadBalancer` — External Traffic Policy

| Policy | Comportamento |
|--------|---------------|
| **Cluster** | Tráfego LB para **qualquer nó saudável**; kube-proxy roteia internamente pro nó com o pod |
| **Local** | Nodes sem o pod são marcados unhealthy; tráfego só vai pra nodes com o pod; **preserva source IP** |

### `kubectl` — imperativo vs declarativo

| Estilo | Comandos | Características |
|--------|----------|-----------------|
| **Imperativo** | `run`, `create`, `replace`, `delete` | Sobrescreve estado, opera em objeto único, especifica **como** |
| **Declarativo** | `apply` | Trabalha em **diretório** de config files, especifica **o que** (estado desejado) |

### Diferenças finas entre comandos imperativos

| Comando | O que faz |
|---------|-----------|
| `kubectl run` | Cria objeto via **argumentos CLI** (default: deployment) |
| `kubectl create` | Cria objeto via **manifest** — falha se já existe, não permite update |
| `kubectl replace` | Baixa spec atual, permite edição, **substitui** objeto completo |
| `kubectl apply` | Cria **OU** atualiza objeto declarativamente — idempotente |
| `kubectl get` | Lista recursos |
| `kubectl expose` | Cria Service que distribui tráfego para pods com labels |

### Specs importantes em manifests

**Deployment:**
- `kind: Deployment`
- `replicas: N`
- `spec.template` → labels, container name, image

**Service:**
- `kind: Service`
- `spec.selector` → labels dos pods que serão incluídos (precisa casar **todos**)
- `port` → porta de entrada (do service)
- `targetPort` → porta que o pod está escutando

---

## 4.3 — Managing Cloud Run Resources

### Escopo
- Deploy de novas versões
- **Traffic splitting** (testes / rollback)
- Setting de **autoscaling**

### Cloud Run vs Cloud Run Functions

| | Cloud Run | Cloud Run Functions |
|--|-----------|---------------------|
| **Conexões por instância** | **Múltiplas concorrentes** | **Uma por instância** |
| **Tipo de carga** | Containers completos | Código funcional |
| **Cobrança** | Por requests/recursos | Por requests |

### Autoscaling — settings e defaults

| Setting | Default | Função |
|---------|---------|--------|
| **CPU utilization** | **60%** | Threshold para autoscaling (não limita conexões) |
| **Concurrency** | **80** (max 1000) | Quantas requests simultâneas por instância |
| **Max instances** | **1000** | Teto de instâncias — limita custo e conexões a backends. Quota increase necessário pra subir |
| **Min instances** | 0 | Piso — mantém instâncias quentes (evita cold start). Cobra mesmo idle |

### Comportamento de idle
- Instâncias podem ficar **idle até 15 minutos** após startup para reduzir cold-start latency
- **Não cobra** durante idle
- Por default, escala para **zero** quando sem tráfego

### Como limitar conexões a um banco backend?
**Resposta correta: Set Max Instances**

| Setting | Por que não? |
|---------|--------------|
| Min instances | É o piso, não o teto |
| CPU utilization | Não afeta conexões a backing services |
| Concurrency | Limita requests/instância, não total de conexões DB |
| **Max instances** ✅ | Limita total de instâncias → limita pool agregado de conexões |

---

## 4.4 — Managing Storage and Database Solutions

### Escopo
- Securing objetos em Cloud Storage buckets
- **Object lifecycle management policies**
- Queries em Cloud SQL, BigQuery, Spanner, Firestore, AlloyDB
- Estimar custo de storage
- Backup/restore (Cloud SQL, Firestore)
- Status de jobs (Dataflow, BigQuery)

### Cloud Storage — Lifecycle Management

**Fluxo:**
```
Conditions → apply to → Objects → when met → Actions
```

**Regras:**
- Aplica-se a **objetos atuais E futuros** no bucket
- Object metadata deve casar **TODAS** as condições da regra (AND)
- Se objeto casa **múltiplas regras**, precedência:
  1. **Delete** vence sempre
  2. Depois, **SetStorageClass** com **menor preço** vence

### Lista completa de Conditions

| Condition | Tipo de critério |
|-----------|------------------|
| `age` | Idade em **dias** |
| `createdBefore` | **Data específica** |
| `customTimeBefore` | Antes do custom time do objeto |
| `daysSinceCustomTime` | Dias desde custom time |
| `daysSinceNoncurrent` | Dias desde virou versão antiga (noncurrent) |
| `isLive` | Versão viva ou não (versionamento) |
| `matchesStorageClass` | Filtra pela class atual |
| `noncurrentTimeBefore` | Antes do timestamp de virar noncurrent |
| `numberOfNewerVersions` | Quantidade de versões mais novas |

### Lista de Actions

| Action | O que faz |
|--------|-----------|
| `Delete` | Apaga o objeto |
| `SetStorageClass` | Move para Nearline / Coldline / Archive |
| `AbortIncompleteMultipartUpload` | Cancela uploads pendentes |

### Exemplos de uso

| Cenário | Solução |
|---------|---------|
| "Downgrade para Coldline após 365 dias" | `age: 365` → `SetStorageClass: COLDLINE` |
| "Delete objetos criados antes de uma data" | `createdBefore: <data>` → `Delete` |
| "Manter 3 versões mais recentes" | `numberOfNewerVersions: 3` → `Delete` |
| **"Standard → Nearline após data X"** | `createdBefore: X` + **`matchesStorageClass: STANDARD`** → `SetStorageClass: NEARLINE` |

⚠️ **Por que precisa de `matchesStorageClass: STANDARD`?** Para garantir que a regra só converta objetos que ainda são Standard, ignorando os já movidos.

---

## 4.5 — Managing Networking Resources

### Escopo
- Adicionar subnet a VPC existente
- **Expandir subnet** (mais IPs)
- Reservar IPs estáticos (externos/internos)
- Cloud DNS e Cloud NAT

### Expandir subnet — regras

| Característica | Detalhe |
|----------------|---------|
| **Direção** | Só **expande** (prefix menor → mais IPs). Não pode encolher. |
| **Downtime** | Sem downtime |
| **Sintaxe** | Reduz o prefix-length (ex: /24 → /20) |

### Comando

```bash
gcloud compute networks subnets expand-ip-range SUBNET \
   --region=REGION \
   --prefix-length=PREFIX_LENGTH
```

### Cálculo de IPs

> **IPs usáveis = 2^(32 − prefix) − 4**

GCP reserva **4 IPs** em qualquer subnet:
- Network address (primeiro)
- Default gateway (segundo)
- Penúltimo (reservado pelo Google)
- Broadcast (último)

| Prefix | Total | Usáveis |
|--------|-------|---------|
| /24 | 256 | 252 |
| /23 | 512 | 508 |
| /22 | 1024 | 1020 |
| /21 | 2048 | **2044** ← cabe 2000 |
| /20 | 4096 | 4092 |

### IPs ephemeral vs static
- IPs default (internos e externos) são **ephemeral** → mudam quando recurso é recriado
- **Static IPs** persistem entre recursos individuais → reserve quando precisar de IP fixo

### Conectividade — exemplos do Cymbal
- **Ecommerce app**: global external (GKE Ingress)
- **Ecommerce middleware**: private regional internal (Spanner)
- **Supply chain app**: external regional + internal regional (Cloud Load Balancers, não Ingress)

---

## 4.6 — Monitoring and Logging

### Escopo (mais amplo da seção)
- Alerts em Cloud Monitoring
- **Custom metrics** (de apps ou logs)
- Exportar logs (on-prem, BigQuery)
- Log buckets, log analytics, log routers
- Filtrar logs no Cloud Logging
- Cloud diagnostics (Error Reporting, Trace, Profiler)
- **Google Cloud status**
- **Ops Agent**
- **Managed Service for Prometheus**
- **Audit logs**

### Ferramentas do Google Cloud Observability

| Ferramenta | Para que serve |
|-----------|----------------|
| **Cloud Monitoring** | Charts, métricas, alertas |
| **Cloud Logging** | Logs timestampados, queries, roteamento |
| **Error Reporting** | Agregação de exceções |
| **Cloud Trace** | Latência distribuída |
| **Cloud Profiler** | Profiling de performance da app |
| **Ops Agent** | Coleta métricas/logs de VMs |
| **Managed Service for Prometheus** | Prometheus gerenciado |

### Alert Policies — estrutura

```
Alert Policy
├── WHAT: Conditions (até 6 por policy)
│   ├── Resource (ex: VM instance)
│   ├── Metric (ex: CPU utilization)
│   └── Threshold (ex: > 0.60 por 5 min)
└── HOW: Notification Channels
    ├── Who: email, slack, pub/sub
    ├── How: tipo de canal
    └── Documentation: anexo opcional
```

### Configuração da Condition

| Campo | Opções |
|-------|--------|
| **Resource type** | VM instance, GKE pod, Cloud Run, etc |
| **Metric** | CPU **utilization** (não load!), memory, request count, etc |
| **Trigger** | "if **any** time series violates" / "if **all** time series violate" |
| **Condition** | **above** / below / equal threshold |
| **Threshold** | valor (em decimal para %: 60% = 0.60) |
| **Duration** | janela de violação (ex: 5 min) |

### Pegadinhas das alertas

| Pegadinha | Resposta certa |
|-----------|----------------|
| CPU usage > 60% por 5 min | **utilization** (não load), **above** (não below), **any** (cada instância, não all) |
| "Each of your instances" | trigger = **any** time series |
| "All instances at once" | trigger = **all** time series |
| Percentual | sempre em decimal (60% → 0.60) |

### Cloud Logging — destinos comuns

| Destino | Quando usar |
|---------|-------------|
| Log buckets (default) | Reter no GCP |
| BigQuery | Análise SQL, retenção longa |
| Cloud Storage | Arquivamento barato |
| Pub/Sub | Streaming para sistemas externos |
| On-premises (via Pub/Sub) | Compliance/SIEM externo |

---

## Resumo de comandos críticos da Seção 4

```bash
# 4.1 Snapshots
gcloud compute snapshots list --project PROJECT_ID
gcloud compute snapshots describe SNAPSHOT_NAME

# 4.2 GKE
kubectl apply -f manifest.yaml         # declarativo (preferido)
kubectl get pods/services/deployments  # listar
kubectl expose ...                     # criar service

# 4.3 Cloud Run (via gcloud)
gcloud run deploy SERVICE --image=IMAGE --max-instances=N

# 4.4 Cloud Storage Lifecycle (via JSON config)
gsutil lifecycle set CONFIG.json gs://BUCKET
# ou via gcloud storage (preferido)

# 4.5 Subnet expansion
gcloud compute networks subnets expand-ip-range SUBNET \
   --region=REGION --prefix-length=N

# 4.6 Alertas (geralmente via console, ou):
gcloud alpha monitoring policies create --policy-from-file=POLICY.yaml
```

---

## Links de documentação oficial relevantes

- Snapshots: `cloud.google.com/compute/docs/disks/create-snapshots`
- Ingress Internal: `cloud.google.com/kubernetes-engine/docs/concepts/ingress-ilb`
- Ingress External: `cloud.google.com/kubernetes-engine/docs/concepts/ingress-xlb`
- K8s Object Management: `kubernetes.io/docs/concepts/overview/working-with-objects/object-management/`
- Cloud Run autoscaling: `cloud.google.com/run/docs/about-instance-autoscaling`
- GCS Lifecycle: `cloud.google.com/storage/docs/lifecycle`
- Subnet expand: `cloud.google.com/sdk/gcloud/reference/compute/networks/subnets/expand-ip-range`
- Alerting: `cloud.google.com/monitoring/alerts`

## Skill Badges relevantes (Section 4)

- **Develop your Google Cloud Network** — `skills.google/course_templates/625`
- **Set Up an App Dev Environment on Google Cloud** — `skills.google/course_templates/637`

## Cursos oficiais Google Cloud (módulos relevantes)

| Curso | Módulos para Seção 4 |
|-------|----------------------|
| Google Cloud Fundamentals: Core Infrastructure | M3 VMs, M4 Storage, M5 Containers, M6 Applications |
| Architecting with Google Compute Engine (ILT) | M2 Virtual Networks, M3 VMs, M5 Storage, M7 Resource Monitoring |
| Essential Google Cloud Infrastructure: Foundation | M2 Virtual Networks, M3 VMs |
| Essential Google Cloud Infrastructure: Core Services | M2 Storage, M4 Resource Monitoring |
| Getting Started with Google Kubernetes Engine | M3 K8s Architecture, M4 K8s Operations |
