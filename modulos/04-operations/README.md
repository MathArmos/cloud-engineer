# Seção 4 — Ensuring successful operation of a cloud solution

> **Domínio 4 da prova ACE.** Gerenciar recursos no dia a dia: snapshots, scaling, lifecycle, monitoring, logging.

---

## Subseções

### 4.1 — Managing Compute Engine resources
**Workbook ACE — Diagnostic Questions:** 01, 02

**Tópicos principais:**
- **Snapshots de persistent disks:**
  - Listar: `gcloud compute snapshots list`
  - Criar: `gcloud compute disks snapshot`
  - Deletar snapshot com schedule: precisa **detach do schedule antes**
- Políticas de snapshot agendado

**Arquivos documentados:**
- [4.1-compute-engine.md](4.1-compute-engine.md) — Snapshots (incremental, só PDs, live, frequência mínima 10min), deletar schedule (detach primeiro), comandos list/describe/delete, imagens vs snapshots.

---

### 4.2 — Managing Google Kubernetes Engine resources
**Workbook ACE — Diagnostic Questions:** 03, 04, 05

**Tópicos principais:**
- **Objetos Kubernetes:**
  - **Pod** — menor unidade, 1+ containers
  - **Service** — endpoint estável para um conjunto de pods (ClusterIP / NodePort / LoadBalancer)
  - **Deployment** — gerencia ReplicaSets, define estado desejado
  - **Pod template** — usado dentro de Deployments
- **Ingress:** rotear HTTP(S) externo para Services
  - **Internal Application LB:** `kubernetes.io/ingress.class: "gce-internal"` ou annotation no Service com NEG
  - **External Application LB:** `kubernetes.io/ingress.class: "gce"`
- **kubectl declarativo vs imperativo:**
  - `kubectl apply -f file.yaml` → **declarativo** (idempotente, recomendado)
  - `kubectl create`, `kubectl run`, `kubectl replace` → imperativos

**Arquivos documentados:**
- [4.2-gke.md](4.2-gke.md) — Objetos K8s (Pod/Deployment/Service), LBs no GKE (L4 Service, L7 Ingress gce/gce-internal + NEG), kubectl imperativo vs declarativo, manifests.

---

### 4.3 — Managing Cloud Run resources
**Workbook ACE — Diagnostic Questions:** 06

**Tópicos principais:**
- **Autoscaling do Cloud Run:**
  - **Min instances** — instâncias sempre quentes (custa, mas evita cold start)
  - **Max instances** — limite superior (controla custo + protege backend)
  - **Concurrency** — requests por instância (limita conexões a DB!)
  - **CPU utilization** — gatilho de scale up
- Caso típico de prova: "limitar conexões ao DB" → ajustar **Max instances**

**Arquivos documentados:**
- [4.3-cloud-run.md](4.3-cloud-run.md) — Autoscaling settings (CPU 60%, concurrency 80, max/min instances), como limitar conexões ao DB (max instances), traffic splitting.

---

### 4.4 — Managing storage and database solutions
**Workbook ACE — Diagnostic Questions:** 07

**Tópicos principais:**
- **Lifecycle rules do Cloud Storage:**
  - Condições: `Age`, `CreatedBefore`, `MatchesStorageClass`, `IsLive`, `NumberOfNewerVersions`, `DaysSinceCustomTime`
  - Ações: `SetStorageClass`, `Delete`
  - Mover de Standard → Nearline depois de **data específica** = usar `CreatedBefore` + `MatchesStorageClass`

**Arquivos documentados:**
- [4.4-storage.md](4.4-storage.md) — Lifecycle rules (conditions: age/createdBefore/matchesStorageClass, actions: SetStorageClass/Delete), precedência de regras, exemplos práticos.

---

### 4.5 — Managing networking resources
**Workbook ACE — Diagnostic Questions:** 08

**Tópicos principais:**
- **Expansão de subnet:** `gcloud compute networks subnets expand-ip-range NAME --region X --prefix-length N`
- **Cálculo de CIDR:** /24 = 256 IPs, /23 = 512, /22 = 1024, /21 = 2048, /20 = 4096
- Regra: prefix MENOR = MAIS IPs (você só pode **expandir**, não reduzir)

**Arquivos documentados:**
- [4.5-networking.md](4.5-networking.md) — Expandir subnet (só expande, zero downtime, prefix menor = mais IPs), cálculo CIDR (2^(32-prefix) − 4), IPs ephemeral vs static, Cloud NAT.

---

### 4.6 — Monitoring and logging
**Workbook ACE — Diagnostic Questions:** 09

**Tópicos principais:**
- **Cloud Monitoring — alerting policies:**
  - Resource type + metric + condition (above/below) + threshold + duration
  - Trigger: **if any time series violates** (mais comum) vs **all time series**
  - Métrica certa: **CPU utilization** (não "CPU load")
- **Cloud Logging:** sinks, retention, exclusion filters

**Arquivos documentados:**
- [4.6-monitoring.md](4.6-monitoring.md) — **SLI/SLO/SLA (2026-05-21, curso Logging and Monitoring in Google Cloud)** — fundação do alerting antes de configurar policy: SLI (indicador, razão good/total — relação linear com experiência), SLO (SLI + alvo + janela; critério S.M.A.R.T.; tabela de noves com downtime/ano-mês-semana; fixed × rolling window), SLA (contrato externo com compensação; anatomia em 5 partes; exemplo billing 0,3% erro), pirâmide SLA→SLO→SLI, por que SLO > SLA (threshold de alerta fica em SLO pra ter respiro), consequências de violar SLO (freeze release + realocar engenharia + suporte executivo), implementação no GCP (Cloud Monitoring → Services → SLOs, comando `gcloud monitoring slos create`), 9 armadilhas de prova (SLI≠alvo, SLO≠SLA, alerta em SLO não SLA, 100% viola Achievable, janela é obrigatória pra Time-bound, etc.). Visão geral do Cloud Observability, Metrics scope (1-375 projetos, acesso unificado, scopes separados para isolamento), Cloud Monitoring (dashboards, alertas, uptime checks, Ops Agent, métricas customizadas), Cloud Logging (sinks: Storage/BigQuery/Pub/Sub, retenção 30 dias, fluxo Splunk via Pub/Sub+Dataflow), Error Reporting (agregação, plataformas incl. EC2, linguagens), Cloud Trace (distributed tracing, latência entre serviços), Cloud Profiler (CPU/mem em produção, baixo overhead). **Recap do módulo Monitoring (2026-05-21)** com 4 takeaways em PT-BR: (1) Scoping project como single-pane-of-glass — diagrama hub-and-spoke, até 375 monitored projects, métricas não migram (só agregação visual), comando `gcloud monitoring metrics-scopes projects add`, armadilhas (logs usam mecanismo separado); (2) Dashboards pre-built × custom com 5 princípios de design; (3) Uptime checks como first line of defense — sondagem black-box de múltiplas regiões, valida DNS+TCP+TLS+200+latência+content, frequência mínima 1min, combinação canônica com alert policy; (4) **PromQL** (conceito novo, recomendação Google atual) — Prometheus Query Language via Managed Service for Prometheus, comparativo MQL × PromQL, sintaxe (`rate()`/`sum by`/`histogram_quantile`), instant vs range vector, exemplo p99 latency, integração com Metrics Explorer/Alert Policies/Dashboards. 13 pegadinhas novas (scoping não move métricas, PromQL não precisa Prometheus self-hosted, `rate()` exige `[5m]`, uptime check mínimo 1min, etc.).
- [curso-core-services.md § Módulo 4](curso-core-services.md#módulo-4--resource-monitoring) — Registro completo do módulo Resource Monitoring: 4.1 visão geral Observability, 4.2 Cloud Monitoring (metrics scope, alertas SRE, uptime checks, Ops Agent, custom metrics + autoscaling), 4.3 Cloud Logging, 4.4 Error Reporting, 4.5 Cloud Trace, 4.6 Cloud Profiler, 4.7 BindPlane + integração Splunk.

---

## Como vamos documentar

Cada arquivo: conceito → por que → quando usar → armadilha de prova → exemplo prático.
Erros de DQ → [../../flashcards.md](../../flashcards.md).
