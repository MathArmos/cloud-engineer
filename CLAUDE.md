# cloud_engineer — workspace de estudo ACE

Este repositório é um **caderno de estudo** da certificação **Google Cloud Associate Cloud Engineer**. Não é código — é documentação organizada por domínio de exame, alimentada conforme o usuário avança no curso oficial do **Google Cloud Skills Boost** e no **ACE Workbook**.

**Deadline:** 2026-05-25 (feira de trabalho). O objetivo do usuário é **entender em profundidade para conversar como engenheiro de cloud**, não tirar nota máxima na prova oficial.

---

## Estrutura

```
cloud_engineer/
├── CLAUDE.md                    ← este arquivo (norte do projeto)
├── README.md                    ← visão geral do plano de estudo
├── flashcards.md                ← DQs erradas + armadilhas (revisão pré-feira)
├── .claude/
│   └── skills/estudar-ace/      ← skill com fluxo detalhado
└── modulos/
    ├── 01-setup-environment/    ← Domínio 1 — IAM, projetos, billing
    ├── 02-planning/             ← Domínio 2 — escolha de compute/storage/network
    ├── 03-deploy/               ← Domínio 3 — implementar recursos
    ├── 04-operations/           ← Domínio 4 — gestão + monitoring
    └── 05-access-security/      ← Domínio 5 — IAM aprofundado + SAs
```

Cada módulo tem um `README.md` que lista as **subseções** do domínio (ex.: 3.1, 3.2, ...) com o que cobre e quais DQs do workbook se aplicam. Os arquivos de tópico são criados **sob demanda** dentro de cada módulo (ex.: `03-deploy/3.1-compute-engine.md`).

---

## Formato dos arquivos de tópico

Todo arquivo `X.Y-nome.md` segue esta estrutura:

```markdown
# X.Y — Nome do tópico
> Fonte, DQs cobertas, considerações da prova

## Conceitos
   (notas explicativas em profundidade)

## Questionários
   (DQs do workbook e quizzes do Skills Boost, resolvidos)

## Labs
   (labs feitos, comandos-chave, observações)

## Cenário real
   (onde isso aparece no trabalho de verdade)

## Armadilhas de prova
   (pegadinhas e detalhes que confundem)
```

Quando um arquivo de tópico cobre vários blocos do mesmo módulo (vídeos, sub-vídeos), use seções `## Bloco "Nome do bloco"` dentro de `## Conceitos` pra manter a origem visível.

---

## Estilo de documentação

**Idioma:** sempre **português brasileiro**.

**Profundidade > concisão.** O usuário quer entender, não decorar. Para cada conceito documentado, cubra:
- **O quê** (definição precisa)
- **Por quê** (qual problema resolve)
- **Quando usar** (cenário típico)
- **Armadilha de prova** (se houver)
- **Exemplo prático** — comando `gcloud` ou cenário concreto

**Use analogias** quando o conceito for abstrato (já temos várias: "Local SSD é como RAM", "burst capability é como bônus de dados pré-pago", "suspend é hibernar laptop").

**Sempre inclua o equivalente `gcloud`** quando o passo foi feito via Console. A prova ACE favorece quem conhece os flags.

**Tabelas e diagramas ASCII** são bem-vindos pra visualizar ciclos de vida, hierarquias, comparativos.

---

## Fluxos de trabalho

A skill **`estudar-ace`** (em `.claude/skills/`) tem o fluxo detalhado dos 3 caminhos principais:

1. **Usuário cola conteúdo de aula** (texto/transcrição/screenshot) → identificar subseção → documentar na seção `## Conceitos` do arquivo do tópico.
2. **Usuário cola DQ ou quiz** → **NÃO revelar resposta antes** que ele tente → após resposta, explicar por que cada alternativa está certa/errada → se errou, adicionar a `flashcards.md`.
3. **Usuário cola instruções de lab** → explicar objetivo do Google → para cada passo, dizer o que faz + por que essa flag → documentar comandos-chave em `## Labs`.

A skill dispara em frases como "documenta isso", "ajuda com esse quiz", "ajuda com esse lab", "explica isso". Quando ela não disparar, **aplique as mesmas convenções manualmente**.

---

## Regras importantes

1. **Nunca revele a resposta de uma DQ antes do usuário tentar.** Pergunte qual ele escolheu e por quê.
2. **Sempre identifique a subseção** ao documentar: "isso é da 3.1 (Compute Engine)" — facilita encontrar depois.
3. **Atualize o README do módulo** quando criar um arquivo novo ou expandir um existente significativamente (lista "Arquivos documentados").
4. **DQs erradas (e armadilhas acertadas por sorte) viram entrada em `flashcards.md`** no formato definido lá.
5. **Datas relativas viram absolutas** ao documentar ("amanhã" → "2026-05-15").
6. **Cortes de emergência** se atrasar: primeiro 3.6 (Terraform — não está no exam guide oficial), depois 4.6 (Observability parte 2).

---

## Pré-feira (a partir de 2026-05-24)

Quando se aproximar do deadline, lembre o usuário de:
- Revisar **todo o `flashcards.md`**
- Reler **os READMEs dos 5 módulos** (visão panorâmica)
- Compilar **1 página de cheatsheet** (hierarquia IAM, árvore de bancos, tipos de LB, `gcloud` críticos)
- Praticar **explicar 3 cenários reais em voz alta** (como faria na feira)

---

## O que NÃO fazer

- ❌ Não usar emojis exceto quando o usuário pedir explicitamente (✅/❌ em tabelas comparativas são exceção aceita).
- ❌ Não criar arquivos fora da estrutura `modulos/0N-.../` sem motivo claro.
- ❌ Não responder em inglês, mesmo que o conteúdo colado esteja em inglês.
- ❌ Não pular o "porquê" ao documentar — explicação rasa não ajuda na feira.
- ❌ Não inflar feedback em DQs — se errou bonito, explique direto onde foi a confusão.
