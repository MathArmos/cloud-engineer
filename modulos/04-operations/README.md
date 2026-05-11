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

**Arquivos documentados:** _(nenhum ainda)_

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

**Arquivos documentados:** _(nenhum ainda)_

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

**Arquivos documentados:** _(nenhum ainda)_

---

### 4.4 — Managing storage and database solutions
**Workbook ACE — Diagnostic Questions:** 07

**Tópicos principais:**
- **Lifecycle rules do Cloud Storage:**
  - Condições: `Age`, `CreatedBefore`, `MatchesStorageClass`, `IsLive`, `NumberOfNewerVersions`, `DaysSinceCustomTime`
  - Ações: `SetStorageClass`, `Delete`
  - Mover de Standard → Nearline depois de **data específica** = usar `CreatedBefore` + `MatchesStorageClass`

**Arquivos documentados:** _(nenhum ainda)_

---

### 4.5 — Managing networking resources
**Workbook ACE — Diagnostic Questions:** 08

**Tópicos principais:**
- **Expansão de subnet:** `gcloud compute networks subnets expand-ip-range NAME --region X --prefix-length N`
- **Cálculo de CIDR:** /24 = 256 IPs, /23 = 512, /22 = 1024, /21 = 2048, /20 = 4096
- Regra: prefix MENOR = MAIS IPs (você só pode **expandir**, não reduzir)

**Arquivos documentados:** _(nenhum ainda)_

---

### 4.6 — Monitoring and logging
**Workbook ACE — Diagnostic Questions:** 09

**Tópicos principais:**
- **Cloud Monitoring — alerting policies:**
  - Resource type + metric + condition (above/below) + threshold + duration
  - Trigger: **if any time series violates** (mais comum) vs **all time series**
  - Métrica certa: **CPU utilization** (não "CPU load")
- **Cloud Logging:** sinks, retention, exclusion filters

**Arquivos documentados:** _(nenhum ainda)_

---

## Como vamos documentar

Cada arquivo: conceito → por que → quando usar → armadilha de prova → exemplo prático.
Erros de DQ → [../../flashcards.md](../../flashcards.md).
