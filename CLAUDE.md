# CLAUDE.md — cloud-engineer

Repositório de estudo para a certificação **Google Cloud Associate Cloud Engineer (ACE)**.

**Objetivo:** entender GCP de verdade para conversar como engenheiro de cloud em uma feira de trabalho.  
**Deadline:** 2026-05-25  
**Prova oficial:** não está planejada — foco é competência técnica real.

---

## Estrutura do repositório

```
cloud-engineer/
  README.md                        # visão geral, grade de cursos, progresso
  CLAUDE.md                        # este arquivo
  flashcards.md                    # erros de DQs para revisão pré-feira
  _key-concepts-cross-cutting.md   # conceitos críticos transversais a todos os domínios

  modulos/
    01-setup-environment/          # IAM, hierarquia, projetos, billing
    02-planning/                   # escolha de compute / storage / network
    03-deploy/                     # implementar GCE, GKE, Cloud Run, dados, network, IaC
    04-operations/                 # gerenciar recursos + monitoring/logging
    05-access-security/            # IAM aprofundado, service accounts

  .claude/skills/estudar-ace/SKILL.md   # skill de estudo
```

### Dentro de cada módulo

```
modulos/XX-nome/
  README.md                      # índice de subseções, DQs, arquivos criados
  X.Y-topico.md                  # arquivo por subseção da prova
  _detailed-notes-from-pdf.md    # notas brutas do workbook PDF (fonte temporária)
  _diagnostic-answers.md         # gabarito oficial das DQs
  lab-nome-do-lab.md             # labs práticos documentados
  curso-nome-do-curso.md         # notas de curso Skills Boost (novo padrão)
```

---

## Convenções de documentação

### Arquivos de subseção (`X.Y-topico.md`)
Formato padrão para cada arquivo:

```
# X.Y — Nome da Subseção
> DQs: XX, YY · Curso: Nome do Curso

## Conceito principal (tabela comparativa)
## Quando usar cada opção
## Comandos essenciais (bash blocks)
## DQ XX — enunciado resumido
Resposta: X — justificativa
tabela com por que cada opção errada está errada

## Pegadinhas de prova (tabela)
```

### Arquivos de curso (`curso-nome.md`)
Para cursos do Skills Boost estudados por completo — novo padrão a partir de "Core Services":

```
# Curso: Nome do Curso
> Duração · Seções da prova cobertas

## Módulo N — Título
conceitos + tabelas + comandos

## DQs cobertas por este curso
## Labs completados
## Conexões com arquivos de subseção
```

### Flashcards (`flashcards.md`)
Só para **erros reais** em DQs. Formato:

```
## [Seção X.Y — Tópico] DQ NN
**Pergunta:** ...
**Meu erro:** ...
**Correto:** ...
**Por que as outras estão erradas:** ...
**Mnemônico:** ...
```

---

## Status dos módulos

| Módulo | Subseções | Arquivos detalhados | DQs |
|--------|-----------|---------------------|-----|
| **01 — Setup/IAM** | 1.1, 1.2 | ✅ Completo | 1–8 |
| **02 — Planning** | 2.1, 2.2, 2.3 | ✅ Completo | 1–9 |
| **03 — Deploy** | 3.1–3.6 | ✅ Completo | 1–10 |
| **04 — Operations** | 4.1–4.6 | ✅ Completo | 1–9 |
| **05 — Access & Security** | 5.1, 5.2 | ✅ Completo | 1–5 |

---

## Grade de cursos Skills Boost

| # | Curso | Status |
|---|-------|--------|
| 1 | A Tour of Google Cloud Hands-on Labs | ✅ |
| 2 | Preparing for Your ACE Journey | ✅ |
| 3 | Essential GCP Infrastructure: Foundation | ✅ |
| 4 | Essential GCP Infrastructure: Core Services | 🟡 em andamento |
| 5 | Select a GCP Database for Your Applications | ⬜ |
| 6 | Elastic GCP Infrastructure: Scaling and Automation | ⬜ |
| 7 | Getting Started with Google Kubernetes Engine | ⬜ |
| 8 | Developing Applications with Cloud Run: Fundamentals | ⬜ |
| 9 | Logging and Monitoring in Google Cloud | ⬜ |
| 10 | Observability in Google Cloud | ⬜ |
| 11 | Implementing Cloud Load Balancing for Compute Engine | ✅ |
| 12 | Set Up an App Dev Environment on Google Cloud | ✅ |
| 13 | Develop Your Google Cloud Network | ⬜ |
| 14 | Build Infrastructure with Terraform on Google Cloud | ⬜ (opcional) |

---

## Seções da prova ACE

| # | Domínio | DQs |
|---|---------|-----|
| 1 | Setting up a cloud solution environment | 1–8 |
| 2 | Planning and configuring a cloud solution | 1–9 |
| 3 | Deploying and implementing a cloud solution | 1–10 |
| 4 | Ensuring successful operation of a cloud solution | 1–9 |
| 5 | Configuring access and security | 1–5 |

Total: **39 Diagnostic Questions**

---

## Como estudar com este repositório

### Fluxo padrão
1. Cole conteúdo de aula → use a skill `/estudar-ace` ou cole direto
2. Cole questão do workbook → Claude guia antes de revelar resposta; erros → `flashcards.md`
3. Cole instruções de lab → explica cada passo + porquê; comandos viram parte da doc
4. Cada conceito: **o quê + por quê + quando usar + armadilha de prova**

### Para cursos Skills Boost
- Criar `curso-nome-do-curso.md` na pasta do módulo mais relevante
- Ao finalizar o curso, extrair conceitos novos para os arquivos de subseção correspondentes
- Marcar o curso como ✅ no `README.md` e neste `CLAUDE.md`

### Para Diagnostic Questions
- DQs do workbook estão mapeadas por seção nos READMEs de cada módulo
- Gabaritos oficiais em `_diagnostic-answers.md` de cada módulo
- Erros vão para `flashcards.md` na raiz

---

## Arquivos mais importantes para revisão rápida

| Arquivo | Para que serve |
|---------|----------------|
| [`flashcards.md`](flashcards.md) | Revisão de todos os erros de DQ |
| [`_key-concepts-cross-cutting.md`](_key-concepts-cross-cutting.md) | Conceitos críticos transversais |
| [`modulos/03-deploy/3.1-compute-engine.md`](modulos/03-deploy/3.1-compute-engine.md) | Compute Engine completo (1.700 linhas) |
| [`modulos/05-access-security/5.2-service-accounts.md`](modulos/05-access-security/5.2-service-accounts.md) | Service Accounts + OAuth + Workload Identity |

---

## Linguagem e estilo

- Documentação em **PT-BR**
- Termos técnicos GCP permanecem em inglês (Cloud Run, Persistent Disk, etc.)
- Tabelas comparativas preferidas a texto corrido
- Comandos `gcloud`/`kubectl` sempre em blocos de código com flags explícitas
- Sem comentários óbvios no código — só quando o "por quê" não é evidente
