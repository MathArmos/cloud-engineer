# Lab — Building Containers with Cloud Build (Dockerfile, cloudbuild.yaml, testes)

**Curso Skills Boost:** Getting Started with Google Kubernetes Engine (Module 2 — Introduction to Containers and Kubernetes)
**Domínio relacionado:** 03-deploy 3.2 GKE (containers + Artifact Registry + Cloud Build — preparação direta para deploy em K8s)
**Status:** ✅ concluído em 2026-05-18

---

## Objetivo do lab

Praticar **3 níveis crescentes de sofisticação** ao construir container images no GCP, usando os 2 serviços canônicos:

| Nível | Build via | Build config | O que demonstra |
|-------|-----------|--------------|-----------------|
| **1** | `gcloud builds submit --tag` | Só Dockerfile | "Build rápido": uma linha, sem YAML |
| **2** | `gcloud builds submit --config cloudbuild.yaml` | YAML com 1 step | Build configurável (steps explícitos, controle fino) |
| **3** | `gcloud builds submit --config cloudbuild2.yaml` | YAML com 2 steps | Build + **teste** do container; exit code propaga pro shell |

O Google quer que o aluno saia entendendo que **Cloud Build é mais que `docker build` gerenciado** — é um **pipeline** onde cada step é um container, podendo encadear build → test → push → deploy. Esse é exatamente o padrão que o curso usará depois para fazer CD para GKE.

**Por que isso é importante.** Em qualquer pipeline real (Cloud Build, GitHub Actions, GitLab CI) você precisa entender:
1. Como **encadear etapas** que dependem da anterior.
2. Como o **exit code não-zero** de qualquer etapa **falha o pipeline inteiro** (princípio "fail fast").
3. Como **a image recém-construída pode ser usada como container do próximo step** (truque elegante do Cloud Build).

---

## Conceitos novos / reforçados

### 1. Artifact Registry — Docker repository

**O quê.** Repositório que armazena container images (e outros formatos: Maven, npm, Python, etc.). **Substituto do antigo Container Registry** — quando a prova falar em "register where your Docker images live in GCP", a resposta canônica hoje é **Artifact Registry**.

**Comando de criação:**
```bash
gcloud artifacts repositories create quickstart-docker-repo \
    --repository-format=docker \
    --location=REGION \
    --description="Docker repository"
```

| Flag | Significado |
|------|-------------|
| `quickstart-docker-repo` | Nome do repositório (não da image — uma repo guarda N images) |
| `--repository-format=docker` | Define que vai guardar Docker-formatted images (outras opções: maven, npm, python, apt, yum, kubeflow) |
| `--location=REGION` | **Regional** (ex.: `us-east1`). Também aceita multi-region (`us`, `europe`) — escolha pela proximidade dos consumidores (Cloud Build, GKE, Cloud Run) |
| `--description` | Texto livre — boa prática para diferenciar repos |

**Formato do path da image dentro do Artifact Registry:**
```
LOCATION-docker.pkg.dev/PROJECT_ID/REPO_NAME/IMAGE_NAME:TAG
```
Exemplo real do lab:
```
us-east1-docker.pkg.dev/qwiklabs-gcp-02-1c7ba5c697a0/quickstart-docker-repo/quickstart-image:tag1
```

**Por quê esse domínio (`pkg.dev`):** Artifact Registry usa `pkg.dev` como sufixo (multi-format). O antigo Container Registry usava `gcr.io` (só Docker) — você verá `gcr.io` em código legado, mas é deprecado.

### 2. `gcloud builds submit --tag` — modo "simples"

**O quê.** Forma mais enxuta de mandar um build pro Cloud Build: nenhum YAML, só **um Dockerfile no diretório atual + uma tag**.

```bash
gcloud builds submit --tag REGION-docker.pkg.dev/${DEVSHELL_PROJECT_ID}/quickstart-docker-repo/quickstart-image:tag1
```

| Componente | O que faz |
|------------|-----------|
| `--tag PATH:TAG` | Cloud Build entende: "use o `Dockerfile` do diretório atual, builda, tagueia e pusha pra essa URL" |
| `${DEVSHELL_PROJECT_ID}` | Variável de ambiente **pré-populada pelo Cloud Shell** com o ID do projeto ativo. Útil pra evitar hardcoding |

**Por baixo dos panos** (importante saber): `--tag` é açúcar sintático. O Cloud Build gera **internamente** um build config com 1 step rodando `gcr.io/cloud-builders/docker build`. Você não vê o YAML, mas ele existe.

**Quando usar `--tag` × `--config`:** `--tag` para builds triviais (um Dockerfile, um destino). `--config` quando precisa de múltiplas etapas, testes, ou substituições customizadas.

### 3. `cloudbuild.yaml` — anatomia

A versão simples (Task 3 do lab) com **1 step**:

```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'us-east1-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1', '.' ]
images:
- 'us-east1-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1'
```

| Campo | Função |
|-------|--------|
| `steps` | Lista ordenada de etapas. Cada step **roda em um container Docker** |
| `name` | **Image** usada como executor do step (aqui: `gcr.io/cloud-builders/docker` — image oficial do Google com docker CLI dentro) |
| `args` | Argumentos passados pra o `ENTRYPOINT` do container do step. Aqui: `docker build -t TAG .` (o ponto = build context = diretório atual do source enviado) |
| `images` | Lista de images a serem **pushadas para o registry após o build**. Sem isso, a image fica só no worker e some |
| `$PROJECT_ID` | **Substitution variable built-in** do Cloud Build. Outras úteis: `$BUILD_ID`, `$COMMIT_SHA`, `$BRANCH_NAME`, `$REPO_NAME` |

**Insight crucial:** o **executor** do step é uma image (`name`). Se você quer rodar `mvn`, usa `gcr.io/cloud-builders/mvn`. Se quer rodar `kubectl`, usa `gcr.io/cloud-builders/kubectl`. Se quer rodar **a sua própria image que acabou de buildar** (truque da Task 4), usa o path do Artifact Registry da image.

### 4. `cloudbuild2.yaml` — pipeline com teste

A versão de 2 steps (Task 4 do lab):

```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'us-east1-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1', '.' ]
- name: 'us-east1-docker.pkg.dev/$PROJECT_ID/quickstart-image:tag1'
  args: ['fail']
images:
- 'us-east1-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1'
```

**O truque:** o `name` do **segundo step é a image que o primeiro step acabou de buildar**. Cloud Build executa essa image, passando `args: ['fail']` (o script `quickstart.sh` foi modificado pra retornar `exit 1` se receber qualquer argumento — simulando um teste que falha).

**O que isso ensina:**

| Capacidade | Demonstração no lab |
|------------|---------------------|
| Steps encadeados | Step 2 só roda depois do Step 1 terminar |
| Build artifact reuso | Image construída no Step 1 é o executor do Step 2 |
| Fail fast | Step 2 falha (exit 1) → build inteiro falha → image **não é pushada** |
| Exit code propagado | `echo $?` no Cloud Shell retorna não-zero |

**Por que isso importa em CI/CD real:** se você não rodar testes no pipeline, código quebrado vai pra prod. Esse padrão "build → testa → push só se passar" é o **mínimo viável** de qualquer CI sério.

### 5. Substitution variables — `$PROJECT_ID` × `${DEVSHELL_PROJECT_ID}`

São **coisas diferentes** apesar de aparentar o mesmo:

| Variável | Onde é interpretada | Quando usar |
|----------|---------------------|-------------|
| `${DEVSHELL_PROJECT_ID}` | **Pelo shell** (Cloud Shell) antes de chamar o `gcloud` | Em comandos de Cloud Shell (Task 2 do lab) |
| `$PROJECT_ID` | **Pelo Cloud Build** durante a execução do build | Dentro do `cloudbuild.yaml` (Tasks 3 e 4) |

**Armadilha:** se você usar `${DEVSHELL_PROJECT_ID}` dentro de um YAML, ele **não existe** no ambiente do worker do Cloud Build → vai virar string vazia → build quebra. Use `$PROJECT_ID` no YAML.

### 6. Alpine como base image

Por que o lab escolhe `FROM alpine`?

- Alpine Linux é uma **distro minúscula** (~5 MB) baseada em musl libc e BusyBox.
- Ideal para containers que só precisam de shell + binários pequenos.
- **Armadilha real (não cobrada no quiz mas vale saber):** Alpine usa **musl** em vez de glibc. Binários compilados pra glibc (Node, alguns Python wheels) podem não rodar direto. Solução: usar `alpine:X.Y` com runtime ajustado, ou trocar pra `debian-slim`/`distroless`.

### 7. Exit code propagation

```bash
gcloud builds submit --config cloudbuild2.yaml   # falha
echo $?                                          # retorna não-zero
```

**Por que isso importa.** Se você embutir esse comando em um script shell, conseguirá detectar o erro:
```bash
gcloud builds submit --config cloudbuild2.yaml || { echo "Build falhou"; exit 1; }
```

Em pipelines reais (GitHub Actions, GitLab CI), isso é o que **automaticamente reprova o PR** ou **bloqueia o deploy** quando o teste falha.

---

## Comandos-chave (cheatsheet do lab)

```bash
# --- Task 1: APIs ---
# (feito pelo console — equivalentes gcloud)
gcloud services enable cloudbuild.googleapis.com
gcloud services enable artifactregistry.googleapis.com

# --- Task 2: Build "rápido" com --tag ---
# 1. Editar quickstart.sh
nano quickstart.sh

# 2. Editar Dockerfile
nano Dockerfile

# 3. Tornar quickstart.sh executável
chmod +x quickstart.sh

# 4. Criar Artifact Registry repo (Docker format, regional)
gcloud artifacts repositories create quickstart-docker-repo \
    --repository-format=docker \
    --location=REGION \
    --description="Docker repository"

# 5. Build + push em uma linha
gcloud builds submit \
    --tag REGION-docker.pkg.dev/${DEVSHELL_PROJECT_ID}/quickstart-docker-repo/quickstart-image:tag1

# --- Task 3: Build com cloudbuild.yaml ---
nano cloudbuild.yaml
export REGION="us-east1"   # ou outra
sed -i "s/YourRegionHere/$REGION/g" cloudbuild.yaml
gcloud builds submit --config cloudbuild.yaml

# --- Task 4: Build + test com cloudbuild2.yaml ---
nano cloudbuild2.yaml
sed -i "s/YourRegionHere/$REGION/g" cloudbuild2.yaml
gcloud builds submit --config cloudbuild2.yaml   # falha intencional
echo $?                                          # confirma exit code != 0
```

### Arquivos finais

**`quickstart.sh` (versão Task 4 — falha se receber argumento):**
```sh
#!/bin/sh
if [ -z "$1" ]
then
    echo "Hello, world! The time is $(date)."
    exit 0
else
    exit 1
fi
```

**`Dockerfile`:**
```dockerfile
FROM alpine
COPY quickstart.sh /
CMD ["/quickstart.sh"]
```

**`cloudbuild.yaml` (Task 3):**
```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'REGION-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1', '.' ]
images:
- 'REGION-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1'
```

**`cloudbuild2.yaml` (Task 4):**
```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'REGION-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1', '.' ]
- name: 'REGION-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1'
  args: ['fail']
images:
- 'REGION-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1'
```

---

## Fluxo completo visualizado

```
┌──────────────────────┐
│ Cloud Shell          │
│                      │
│ - cria quickstart.sh │
│ - cria Dockerfile    │
│ - cria cloudbuild.yml│
└──────────┬───────────┘
           │  gcloud builds submit
           ▼
┌──────────────────────────────────────────────────────┐
│ Cloud Build (worker effêmero)                        │
│                                                      │
│ Step 1: docker build  →  image                       │
│           ↓                                          │
│ Step 2: image (executor)  →  args: ['fail']  →  ✗   │
│           ↓                                          │
│ BUILD FAILURE → image NÃO é pushada                  │
└──────────────────────────────────────────────────────┘
           │  (se passasse)
           ▼
┌──────────────────────┐
│ Artifact Registry    │
│ quickstart-docker-   │
│ repo / quickstart-   │
│ image:tag1           │
└──────────────────────┘
           │
           ▼
   Próximo passo: GKE / Cloud Run / etc
```

---

## Cenário real (para a feira)

> **"Como vocês fazem CI/CD de containers no GCP?"**
>
> Aqui é direto: **Cloud Build** lê um `cloudbuild.yaml` versionado junto com o código. Steps típicos do nosso pipeline: (1) `npm test` ou equivalente, (2) `docker build`, (3) `docker push` para Artifact Registry, (4) `gcloud run deploy` ou `kubectl set image`. Cada step roda num container; se qualquer um falha, o pipeline para e o deploy nunca acontece. Variáveis sensíveis ficam em **Secret Manager** acopladas ao build. Para PR previews, usamos tags por commit SHA (`$COMMIT_SHA`) e prefixamos a image. Para prod, taggeamos com a versão semântica. **Artifact Registry** substituiu o antigo Container Registry — `pkg.dev` em vez de `gcr.io`.

---

## Armadilhas de prova

| Pegadinha | Resposta correta |
|-----------|------------------|
| "Container Registry é onde guardo images no GCP" | ❌ Deprecado. Use **Artifact Registry** (`pkg.dev`) |
| "Cada step do Cloud Build é uma função serverless" | ❌ Cada step **roda em um container Docker** (a image é o campo `name`) |
| "`$PROJECT_ID` funciona no Cloud Shell antes de chamar gcloud" | ❌ Só **dentro do `cloudbuild.yaml`** (substitution do Cloud Build). No shell use `${DEVSHELL_PROJECT_ID}` |
| "Sem o campo `images:` no YAML, a image ainda vai pro registry" | ❌ Sem `images:`, a image é só local ao worker e some quando o build acaba |
| "Step que falha não impede o push da image" | ❌ Build inteiro falha → **nada** é pushado |
| "Pra rodar teste, preciso de um step com framework de teste pré-instalado" | ❌ O próprio container recém-buildado pode ser o executor do step de teste (truque do `name: image-que-acabei-de-criar`) |
| "Alpine é compatível com qualquer binário Linux" | ❌ Alpine usa **musl libc**, não glibc. Binários compilados pra glibc podem quebrar |
| "Build sem `--tag` nem `--config` funciona se houver Dockerfile" | ❌ Precisa de pelo menos um dos dois para Cloud Build saber o destino da image |

---

## Próximos passos (depois do lab)

1. **Module 3 do curso** vai usar essa mesma image em GKE → o ciclo finalmente fecha (Cloud Build → Artifact Registry → GKE pull → Pod rodando).
2. Para CI/CD real: adicionar trigger no Cloud Build conectado ao GitHub para builds automáticos em push/PR.
3. Para builds mais seguros: configurar **Binary Authorization** no GKE (só roda images assinadas por builds aprovados).
