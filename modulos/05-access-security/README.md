# Seção 5 — Configuring access and security

> **Domínio 5 da prova ACE.** Quem pode fazer o quê em qual recurso: IAM aprofundado + service accounts.

---

## Subseções

### 5.1 — Managing Identity and Access Management (IAM)
**Workbook ACE — Diagnostic Questions:** 01, 02, 03

**Tópicos principais:**
- **Tipos de identidade:**
  - **Google Account** — humano com email Google
  - **Google Workspace account** — humano em domínio gerenciado por Workspace
  - **Cloud Identity** — Workspace sem o Workspace (só identidade)
  - **Service Account** — identidade para aplicações/recursos (NÃO humano)
- **Hierarquia de IAM bindings:** Org → Folder → Project → Resource (herança descendente)
- **Roles necessárias para gerenciar IAM em diferentes níveis:**
  - Editar IAM de um projeto: `resourcemanager.projects.setIamPolicy`
  - Editar IAM de uma pasta: `roles/resourcemanager.folderIamAdmin`
  - Editar IAM da organização: `roles/resourcemanager.organizationAdmin`
- **Custom roles:**
  - Update local (definição YAML/JSON) + comando de update = forma recomendada
  - NÃO deletar e recriar (perde IAM bindings)

**Arquivos documentados:** _(nenhum ainda)_

---

### 5.2 — Managing service accounts
**Workbook ACE — Diagnostic Questions:** 04, 05

**Tópicos principais:**
- **Quando usar Service Account:** para **aplicações** acessarem recursos (não para usuários humanos, não para análise interativa, não para devs)
- **Autenticação client-side (apps mobile/browser):**
  - **OAuth 2.0 user credentials** = padrão recomendado para apps que acessam dados privados em nome do usuário
  - **Service account keys** = perigoso em cliente (chave vaza)
  - **API keys** = só para APIs públicas/anônimas (Maps, etc.), NÃO para dados privados
- **Workload Identity (GKE):** vincula Kubernetes Service Account a Google Service Account — forma segura, sem chaves expostas

**Arquivos documentados:** _(nenhum ainda)_

---

## Como vamos documentar

Cada arquivo: conceito → por que → quando usar → armadilha de prova → exemplo prático.
Erros de DQ → [../../flashcards.md](../../flashcards.md).
