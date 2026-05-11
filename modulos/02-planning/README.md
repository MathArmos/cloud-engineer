# Seção 2 — Planning and configuring a cloud solution

> **Domínio 2 da prova ACE.** Escolha das tecnologias certas para o problema: compute, storage, network.

---

## Subseções

### 2.1 — Planning and configuring compute resources
**Workbook ACE — Diagnostic Questions:** 01, 02, 03, 04

**Tópicos principais:**
- **Árvore de decisão de compute:**
  - **Compute Engine (VMs):** OS customizado, controle total, lift-and-shift
  - **GKE:** microsserviços, controle de container, sem gerenciar control plane
  - **Cloud Run:** containers portáveis, scale to zero, foco em código
  - **App Engine:** apps web, sem container
  - **Cloud Run functions:** event-driven, código curto
- Quando cada um faz sentido (caso de uso → escolha)

**Arquivos documentados:** _(nenhum ainda)_

---

### 2.2 — Planning and configuring data storage options
**Workbook ACE — Diagnostic Questions:** 05, 06, 07

**Tópicos principais:**
- **Storage classes (Cloud Storage):** Standard / Nearline / Coldline / Archive — critério é frequência de acesso
- **Bancos de dados:**
  - **BigQuery** — data warehouse analítico, SQL, OLAP
  - **Cloud SQL** — relacional gerenciado (MySQL/Postgres/SQL Server), OLTP
  - **Spanner** — relacional global, escala horizontal massiva
  - **Firestore** — NoSQL documento, transacional
  - **Bigtable** — NoSQL wide-column, séries temporais e analítico de alto throughput
- Árvore de decisão "qual banco usar?"

**Arquivos documentados:** _(nenhum ainda)_

---

### 2.3 — Planning and configuring network resources
**Workbook ACE — Diagnostic Questions:** 08, 09

**Tópicos principais:**
- **Cloud Load Balancing — combinações:**
  - **Application** (camada 7, HTTP/HTTPS) vs **Network** (camada 4, TCP/UDP)
  - **Global** vs **Regional**
  - **External** (internet) vs **Internal** (VPC)
  - **Proxy** vs **Passthrough**
  - **Premium tier** (rede Google global) vs **Standard tier** (rede regional, mais barato)
- Camadas TCP (L4 vs L7)

**Arquivos documentados:** _(nenhum ainda)_

---

## Como vamos documentar

Cada arquivo dedicado vai cobrir: conceito → por que importa → quando usar → armadilha de prova → exemplo prático.
Erros de DQ vão para [../../flashcards.md](../../flashcards.md).
