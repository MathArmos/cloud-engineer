# Estudo — Associate Cloud Engineer Certification

**Curso:** Google Cloud Skills Boost — ACE Learning Path
**Workbook:** Preparing for Your ACE Journey (39 Diagnostic Questions cobrindo os 5 domínios da prova)
**Início:** 2026-05-11
**Deadline:** 2026-05-25 (feira de trabalho)
**Créditos GCP:** ✅ disponíveis · **Skills Boost:** ✅ acesso
**Prova oficial:** não será feita — objetivo é entender de verdade pra conversar como engenheiro de cloud na feira

---

## Como funciona

Documentação orgânica organizada pelos **5 domínios oficiais do exame ACE**:

1. Você cola conteúdo de aula → documento na subseção correta em [modulos/](modulos/)
2. Você cola questão (workbook ou quiz Skills Boost) → te guio antes de revelar resposta; erros vão para [flashcards.md](flashcards.md)
3. Você cola instruções de lab → explico cada passo + porquê, comandos viram parte da doc
4. Para cada conceito: **o quê + por quê + quando usar + armadilha de prova**

Use a skill `/estudar-ace` (ou cole o conteúdo direto) que eu cuido do resto.

---

## Estrutura

```
cloud_engineer/
  README.md                  # este arquivo
  flashcards.md              # erros de DQs para revisão antes da feira
  modulos/
    01-setup-environment/    # IAM, hierarquia, projetos, billing
    02-planning/             # escolha de compute/storage/network
    03-deploy/               # implementar GCE/GKE/Cloud Run/dados/network/IaC
    04-operations/           # gestão dos recursos + monitoring/logging
    05-access-security/      # IAM aprofundado, service accounts
  .claude/skills/estudar-ace/SKILL.md
```

Cada pasta de seção tem um **README** com:
- Lista de subseções (1.1, 1.2, etc.) com seus tópicos
- Quais DQs do workbook caem em cada subseção
- Onde os arquivos detalhados vão sendo criados conforme estudamos

---

## Seções da prova (5 domínios)

| # | Domínio | Subseções | Workbook DQs |
|---|---------|-----------|--------------|
| [1](modulos/01-setup-environment/README.md) | Setting up environment | 1.1 projects/IAM · 1.2 billing | 1-8 |
| [2](modulos/02-planning/README.md) | Planning & configuring | 2.1 compute · 2.2 storage · 2.3 network | 1-9 |
| [3](modulos/03-deploy/README.md) | Deploying & implementing | 3.1 GCE · 3.2 GKE · 3.3 Cloud Run · 3.4 data · 3.5 networking · 3.6 IaC | 1-10 |
| [4](modulos/04-operations/README.md) | Ensuring operation | 4.1 GCE mgmt · 4.2 GKE mgmt · 4.3 CR mgmt · 4.4 storage · 4.5 net mgmt · 4.6 monitoring | 1-9 |
| [5](modulos/05-access-security/README.md) | Access & security | 5.1 IAM mgmt · 5.2 service accounts | 1-5 |

Total: **39 Diagnostic Questions** ao longo dos 5 domínios.

---

## Grade do Skills Boost (cursos em paralelo)

| # | Curso | Duração | Status |
|---|-------|---------|--------|
| 1 | A Tour of Google Cloud Hands-on Labs | 45min | ✅ |
| 2 | Preparing for Your ACE Journey | — | 🟡 9% |
| 3 | Essential GCP Infrastructure: Foundation | 6h45 | ⬜ |
| 4 | Essential GCP Infrastructure: Core Services | 8h15 | ⬜ |
| 5 | Select a GCP Database for Your Applications | 6h | ⬜ |
| 6 | Elastic GCP Infrastructure: Scaling and Automation | 7h | ⬜ |
| 7 | Getting Started with Google Kubernetes Engine | 5h45 | ⬜ |
| 8 | Developing Applications with Cloud Run: Fundamentals | 8h | ⬜ |
| 9 | Logging and Monitoring in Google Cloud | 8h30 | ⬜ |
| 10 | Observability in Google Cloud | 6h30 | ⬜ |
| 11 | Implementing Cloud Load Balancing for Compute Engine | 30min | ✅ |
| 12 | Set Up an App Dev Environment on Google Cloud | 1h15 | ✅ |
| 13 | Develop Your Google Cloud Network | 1h15 | ⬜ |
| 14 | Build Infrastructure with Terraform on Google Cloud | 1h45 | ⬜ (opcional) |

**Legenda:** ⬜ pendente · 🟡 em andamento · ✅ concluído
