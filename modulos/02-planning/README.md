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

**Arquivos documentados:**
- [2.1-compute-resources.md](2.1-compute-resources.md) — fundação completa (PDF M2 L7): 2 famílias, 5 serviços, tabela de pistas léxicas, DQs 01-04 resolvidas

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

**Arquivos documentados:**
- [2.2-data-storage.md](2.2-data-storage.md) — fundação completa (PDF M2 L7): 4 famílias (relational/NoSQL/object/warehouse), OLTP vs OLAP, storage classes, DQs 05-07 resolvidas
- [2.2-storage-database-overview.md](2.2-storage-database-overview.md) — overview do curso Essential GCP Infra: Core Services M3: decision tree formal, Filestore, AlloyDB, Memorystore, HTAP
- [2.2-cloud-storage.md](2.2-cloud-storage.md) — deep dive Cloud Storage (Core Services M3): buckets/objetos, 4 storage classes, location types, durability vs availability, IAM/ACL/Signed URLs, encryption (CMEK/CSEK), Versioning vs Soft Delete, Lifecycle, Retention Lock, Autoclass, Pub/Sub notifications, transfer services, strong consistency
- [2.2-filestore.md](2.2-filestore.md) — deep dive Filestore (Core Services M3): NFSv3 gerenciado, comparativo Cloud Storage vs PD vs Filestore, 6 use cases, tiers, mount em GCE e GKE (RWX)
- [2.2-cloud-sql.md](2.2-cloud-sql.md) — deep dive Cloud SQL (Core Services M3): managed vs self-managed, 3 engines (MySQL/Postgres/SQL Server), limites (64TB/60k IOPS/624GB/96 vCPU), HA regional síncrona, backups + PITR, scale up vs read replicas, 4 connection types (Private IP / Auth Proxy / SSL manual / IP autorizado)
- [2.2-spanner.md](2.2-spanner.md) — deep dive Spanner (Core Services M3): relacional + horizontal scale + global strong consistency, TrueTime / atomic clocks, regional vs multi-regional SLA, 5 gatilhos canônicos, comparativo Spanner vs Cloud SQL, pricing por node, dialeto Postgres, quando NÃO usar
- [2.2-alloydb.md](2.2-alloydb.md) — deep dive AlloyDB (Core Services M3): Postgres-compatible com HTAP, storage/compute desacoplados, engine colunar, 4x OLTP / 100x OLAP vs Postgres padrão, 99,99% SLA inclusive of maintenance, Vertex AI inline em SQL, comparativo Cloud SQL vs AlloyDB vs Spanner
- [2.2-firestore.md](2.2-firestore.md) — deep dive Firestore (Core Services M3): NoSQL document, serverless + scale to zero, ACID multi-document, mobile/web SDKs (live sync + offline), Native mode vs Datastore mode (3 limitações antigas removidas), multi-region vs regional, decision tree Firestore vs Bigtable
- [2.2-bigtable.md](2.2-bigtable.md) — deep dive Bigtable (Core Services M3): NoSQL wide-column PB-scale, <10ms, mesmo banco de Search/Maps/Gmail, storage model (row key + column family + sparse + multi-version), processing separado de Colossus, tablets + SSTable, rebalance sem mover dado, throughput linear, 5 gatilhos canônicos, HBase API, 3 nodes = 30k ops/s mínimos (não escala a zero)
- [2.2-memorystore.md](2.2-memorystore.md) — deep dive Memorystore (Core Services M3): cache in-memory gerenciado (Redis/Memcached), sub-ms, HA 99,9% entre 2 zonas, até 300 GB / 12 Gbps, Basic vs Standard tier, lift-and-shift Redis OSS (zero code), Private IP only, 6 padrões de uso (cache aside, session, leaderboard, rate limit, fila, pub/sub leve), NÃO é source of truth

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

**Arquivos documentados:**
- [2.3-network-resources.md](2.3-network-resources.md) — fundação completa (PDF M2 L7): 5 eixos de decisão de LB, catálogo dos 10 tipos, Network Service Tiers, DQs 08-09 resolvidas
- [2.3-vpc-fundamentals.md](2.3-vpc-fundamentals.md) — VPC theory (Essential GCP Infra Foundation M2): networks default/auto/custom, subnets, IPs internal/external, DNS, Cloud DNS, Alias IPs, gotchas

---

## Como vamos documentar

Cada arquivo dedicado vai cobrir: conceito → por que importa → quando usar → armadilha de prova → exemplo prático.
Erros de DQ vão para [../../flashcards.md](../../flashcards.md).
