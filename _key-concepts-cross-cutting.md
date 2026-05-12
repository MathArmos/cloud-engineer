---
name: ACE Key Concepts and Gotchas
description: Critical concepts, traps and gotchas for the Google Cloud Associate Cloud Engineer exam — distilled from Section 3 diagnostic mistakes
type: reference
originSessionId: 488c0de8-00cc-4df2-8923-ea2633411d9c
---
# Conceitos-Chave e Pegadinhas do ACE

## Compute Engine — Machine Types
| Tipo | Uso recomendado |
|------|-----------------|
| **N2** | Balanced — bancos de dados médios/grandes ✅ |
| **E2** | Cost-optimized (custo baixo) |
| **C2** | Compute-optimized |
| **M2** | Memory-optimized |
| **N1** | General purpose com suporte a GPU |

## Cloud SQL — Limitações Críticas
- **NÃO suporta UDFs (User-Defined Functions)** personalizadas → para MySQL com UDFs, usar Compute Engine
- `--availability-type=REGIONAL` → failover automático
- `--availability-type=ZONAL` → sem failover

## Managed Instance Groups (MIG) — Tipos de Update
| Tipo | Comportamento |
|------|---------------|
| **PROACTIVE** | Atualiza todas as VMs automaticamente (rolling, surge=1) |
| **Opportunistic** | NÃO é automático — só atualiza em ações manuais |
| **Max Surge** | Define quantas VMs novas criar de uma vez |

## GKE — Modos de Cluster
| Característica | Standard | Autopilot |
|----------------|----------|-----------|
| Zonal | ✅ | ❌ (sempre regional) |
| Permite alterar arquitetura | ✅ | ❌ (gerenciado a nível de pod) |
| Suporta Ubuntu | ✅ | ❌ (apenas container-optimized) |
| Pacotes customizados | ✅ | ❌ |

## Cloud Storage — Localizações
- **Multi-region** (US, EU, ASIA) → cobre continente inteiro
- **Dual-region** (`--placement`) → 2 regiões **do mesmo continente**
- **Sem `--location`** → padrão é US multi-region
- ⚠️ `gsutil` está sendo **descontinuado** — preferir `gcloud storage`

## Cloud Storage — Triggers
- ✅ `google.storage.object.finalize` — único evento válido para "objeto criado/escrito"
- ❌ `.create`, `.change`, `.add` — **NÃO existem**

## BigQuery — Métodos de Ingestão
| Método | Custo | Caso de uso |
|--------|-------|-------------|
| **BigQuery Data Transfer Service** | **Grátis** | Slowly changing data, agendado ✅ |
| `bq load` + cron | Grátis | Mais complexo, manual |
| Streaming API | Pago por volume | Real-time |
| Dataflow pipeline | Pago | ETL complexo |

## App Engine — Standard vs Flexible
| Característica | Standard | Flexible |
|----------------|----------|----------|
| Escala para zero | ✅ | ❌ |
| Pacotes customizados | ❌ | ✅ |
| Linguagens | Restritas | Mais opções |
| Modifica runtime | ❌ | ✅ |
| Startup | Segundos | Minutos |

## Cloud Run — Características
- Serverless container management
- **Scale to zero** ✅
- Suporta pacotes/dependências customizadas ✅
- Pay-per-request (billing 100ms)
- Timeout 60min
- Baseado em Knative

## VPC — Tipos de Rede
| Tipo | Controle |
|------|----------|
| **Default** | Auto mode pré-criado |
| **Auto mode** | 1 subnet/região automática, IPs predeterminados |
| **Custom mode** | Controle total sobre IPs e sub-redes ✅ |
| **Auto → Custom** | Conversão mantém IPs originais (não pode reverter) |

## Terraform — Lifecycle
1. `terraform init` → baixa providers
2. `terraform plan` → preview + verifica sintaxe
3. `terraform apply` → cria recursos
4. `terraform destroy` → destrói recursos

## Pegadinhas Comuns no Exame
1. **Cloud SQL ≠ MySQL completo** (sem UDFs)
2. **`gsutil` está saindo** → preferir `gcloud storage`
3. **PROACTIVE vs Opportunistic** → só PROACTIVE é automático
4. **Autopilot ≠ Standard GKE** → Autopilot é regional e restrito
5. **`finalize`** é o trigger correto do Cloud Storage (não `create`)
6. **BigQuery Data Transfer Service é GRÁTIS** para Cloud Storage → BigQuery
7. **Bucket sem `--location`** → padrão US multi-region

## GKE — Load Balancers (Seção 4)
| Quero criar | Setting K8s |
|-------------|-------------|
| **External Network LB (L4)** | `Service: type: LoadBalancer` |
| **Internal Network LB (L4)** | `Service: LoadBalancer` + annotation `cloud.google.com/load-balancer-type: "Internal"` |
| **External HTTP(S) LB (L7)** | `Ingress` + `kubernetes.io/ingress.class: "gce"` |
| **Internal HTTP(S) LB (L7)** | `Ingress` + `kubernetes.io/ingress.class: "gce-internal"` + **Service com NEG** `cloud.google.com/neg: '{"ingress": true}'` |

⚠️ **Internal HTTP(S) LB EXIGE NEGs** (container-native load balancing) — sem isso não funciona.

## VPC — Cálculo CIDR para Subnets
- **Fórmula**: IPs usáveis = 2^(32 − prefix) − 4
- GCP reserva **4 IPs por subnet**: network address, gateway, penúltimo (Google), broadcast
- Tabela rápida:
  - /24 → 252 usáveis
  - /23 → 508
  - /22 → 1020
  - /21 → 2044 (atende 2000)
  - /20 → 4092
- ✅ Subnet pode ser **expandida** (prefix menor), **não pode encolher**
- Comando: `gcloud compute networks subnets expand-ip-range <subnet> --region=<r> --prefix-length=<n>`

## Cloud Run — Controles de Escala
| Setting | Função |
|---------|--------|
| **Max instances** | Teto — controla custos E limita conexões a backends (DB) |
| **Min instances** | Piso — reduz cold starts mas custa idle |
| **Concurrency** | Requests simultâneas POR instância (não limita total) |
| **CPU Utilization** | Threshold para autoscaling, não limita conexões |

Para **limitar conexões ao DB**: usar **Max instances** (cada instância tem pool próprio).

## Snapshots — Pegadinhas
- Snapshots são **incrementais e gerenciados pelo Google** — você pode deletar qualquer um sem se preocupar com a cadeia
- **Snapshot schedule attached a PD não pode ser deletada** → fazer `gcloud compute disks remove-resource-policies` primeiro
- Listar snapshots: `gcloud compute snapshots list`

## Cloud Storage — Lifecycle Rules
**Conditions disponíveis**:
- `age` (dias desde criação)
- `createdBefore` (data específica) ✅
- `customTimeBefore` / `daysSinceCustomTime`
- `daysSinceNoncurrentTime`
- `isLive` (versionamento)
- `matchesStorageClass` (filtra pela class atual) ✅
- `numNewerVersions`
- `matchesPrefix` / `matchesSuffix`

**Actions**:
- `Delete`
- `SetStorageClass` (Nearline/Coldline/Archive)
- `AbortIncompleteMultipartUpload`

⚠️ Para **converter por data** (Standard → Nearline): usar **`createdBefore` + `matchesStorageClass`** juntos (matchesStorageClass evita reconverter objetos que já saíram de Standard).

## Cloud Monitoring — Alert Policies
- **Trigger "any time series violates"** = dispara se **qualquer** VM passar do limite (mais sensível, comum)
- **Trigger "all time series violates"** = só se TODAS violarem (raro)
- **CPU utilization** ≠ **CPU load** (utilization = %, load = média de processos)
- Threshold + Duration → ambos necessários (evita falsos positivos por picos)
- Valores percentuais em decimal: 60% = **0.60**

## BigQuery External Tables (Bigtable) — Seção 4
Sequência para query SQL de dados do Bigtable:
1. Criar **table definition file (JSON)** com URI, column families, columns
2. `bq mk --external_table_definition=<file> <dataset>.<table>` → cria **permanent external table**
3. Query SQL com `FROM <dataset>.<table>` no BigQuery

**Permanent** (este caso) vs **Temporary**:
- Permanent → query regular, fica salva, tem IAM próprio (`bq mk`)
- Temporary → ad-hoc, some após sessão (`bq query` com flags inline)
