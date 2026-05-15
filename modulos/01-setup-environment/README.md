# Seção 1 — Setting up a cloud solution environment

> **Domínio 1 da prova ACE.** Configuração inicial do ambiente Google Cloud: IAM, hierarquia de recursos, projetos, billing.

---

## Subseções

### 1.1 — Setting up cloud projects and accounts
**Workbook ACE — Diagnostic Questions:** 01, 02, 03, 04, 05, 06

**Tópicos principais:**
- Hierarquia de recursos: **Organization → Folder → Project → Resource**
- IAM roles: **basic (primitivas)** vs **predefinidas** vs **custom**
- Atributos de projeto: Project **ID** (imutável), **Name** (mutável), **Number** (imutável)
- Princípio do **least privilege**
- Uso de **Google Groups** para escalar permissões

**Arquivos documentados:**
- [1.1-resource-hierarchy-iam.md](1.1-resource-hierarchy-iam.md) — criado em 2026-05-11, expandido em 2026-05-14 (cobre hierarquia, projetos, IAM, APIs, Cloud Identity, project scoping para monitoring, **allow vs deny policy**, **billing bottom-up**, **classificação global/regional/zonal**, **quotas**)

---

### 1.2 — Managing billing configuration
**Workbook ACE — Diagnostic Questions:** 07, 08

**Tópicos principais:**
- Billing accounts e relação com projetos (1 billing account → N projects)
- Um projeto só pode estar vinculado a **uma** billing account por vez
- **Budget alerts:** destinatários default vs custom email recipients
- Monitorar gastos sem expor billing info sensível (Pub/Sub topic, Cloud Monitoring)

**Arquivos documentados:**
- [1.2-billing.md](1.2-billing.md) — criado em 2026-05-11, expandido em 2026-05-14 (cobre billing accounts, vincular projeto, budgets/alerts, custom email recipients, billing export, **labels para chargeback intra-projeto**, **labels vs network tags**)

---

### Labs práticos (suporte à seção)

- [lab-console-cloudshell.md](lab-console-cloudshell.md) — concluído em 2026-05-12 · "Working with the Google Cloud Console and Cloud Shell" (1º lab do learning path). Cobre interfaces GCP, persistência via `.profile`, `gcloud storage`, comandos-chave de Cloud Shell.
- **Lab "Examine Billing Data with BigQuery"** — concluído em 2026-05-15 · documentado em [1.2-billing.md `## Labs`](1.2-billing.md). Cobre criar dataset, importar Avro do GCS, schema do billing export (RECORD/REPEATED), 7 queries SQL (filtros, ORDER BY, GROUP BY, SUM agregado), volume ≠ custo.

---

## Como vamos documentar

Para cada subseção, quando começarmos:
1. Crio um arquivo dedicado (ex.: `1.1-iam-e-hierarquia.md`)
2. Vamos pelas DQs (Diagnostic Questions) uma a uma — você tenta antes de eu revelar
3. Cada conceito ganha entrada no arquivo: **o quê + por quê + quando usar + armadilha de prova**
4. Erros viram entradas em [../../flashcards.md](../../flashcards.md)
