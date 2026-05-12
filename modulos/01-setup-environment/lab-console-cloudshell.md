# Lab — Working with the Google Cloud Console and Cloud Shell

**Seção 1 — Setting up a cloud solution environment** (suporte transversal a todas as seções)
**Curso Skills Boost:** A Tour of Google Cloud Hands-on Labs (1º lab do learning path ACE)
**Duração:** ~20 min · **Status:** ✅ concluído em 2026-05-12

---

## Objetivo do lab

Familiarizar o aluno com as **duas interfaces principais do GCP**:

1. **Cloud Console (GUI):** painel web em `console.cloud.google.com` — para ações pontuais, descoberta de serviços e validação visual.
2. **Cloud Shell (CLI):** terminal Linux gratuito acoplado ao Console — para automação, scripts e operações em escala.

O Google quer que você saia desse lab sabendo que **as duas interfaces são equivalentes** (qualquer coisa feita em uma pode ser feita na outra), mas com **trade-offs distintos**.

---

## Conceitos

### 1. Cloud Console (interface web)

**O quê.** GUI em `console.cloud.google.com`. Centraliza todos os serviços GCP em menus categorizados (Compute, Storage, Network, IAM, Billing, etc.).

**Por que importa.**
- **Velocidade para ações pontuais:** criar 1 bucket, ativar 1 API, conceder 1 role — clicks bem mais rápidos que digitar comandos longos.
- **Reduz erros:** só oferece opções válidas nos menus (você não consegue digitar uma região inexistente, por exemplo).
- **Validação por trás dos panos:** antes de submeter, o Console chama o SDK para validar a configuração — coisa que a CLI crua não faz.
- **Notifications:** o sininho no topo mostra histórico de operações assíncronas (boas para debug quando algo "não acontece").

**Limitações importantes.**
- Nem todo recurso/flag está exposto na GUI. Tecnologias muito novas ou opções avançadas de API muitas vezes **só existem na CLI ou via API REST**.
- Layout muda com frequência (Google itera o produto) — não decore caminhos de menu, decore os **nomes dos serviços**.

### 2. Cloud Shell (terminal acoplado ao Console)

**O quê.** Máquina virtual Linux temporária (Debian) que abre direto no navegador, dentro do Console. Vem com tudo pré-instalado.

**Recursos pré-instalados:**
- `gcloud` — CLI principal do GCP (Compute, IAM, projetos, etc.)
- `gcloud storage` (substituiu o antigo `gsutil`) — para Cloud Storage
- `kubectl` — para Kubernetes/GKE
- `bq` — para BigQuery
- Linguagens: Java, Go, Python, Node.js, PHP, Ruby
- Editor de código embutido (Cloud Shell Editor, baseado em Theia/VS Code)
- Web preview (porta 8080+ para testar apps localmente no shell)

**Modelo de persistência (ARMADILHA importante):**
- `$HOME` (`/home/<seu_user>`) → **5 GB persistentes**, sobrevive a reciclagem.
- Tudo fora de `$HOME` (incluindo pacotes instalados via `apt`, variáveis de ambiente, `/tmp`, etc.) → **perdido após 1h de inatividade** ou ao fechar a sessão. A VM é reciclada.

**Por que importa.**
- **Sem instalar SDK no PC:** acesso instantâneo a partir de qualquer máquina com navegador. Útil em demos, troubleshooting de emergência, ou em hosts onde você não pode instalar nada.
- **Autorização built-in:** o Cloud Shell já vem autenticado com a conta que abriu o Console. Sem `gcloud auth login`.
- **Custo zero:** não consome créditos do projeto.

### 3. Os 4 botões da barra do Cloud Shell

| Botão | Função |
|---|---|
| **Minimize** | Recolhe o terminal, mantém a sessão viva. |
| **Restore/Maximize** | Expande Cloud Shell ocupando a tela toda (mas mantém Console acessível). |
| **Open in a new window** | Pop-out: terminal em janela própria. Útil quando você vai editar arquivos longos ou ver outputs extensos. |
| **Close terminal** | **Encerra a sessão e reseta a VM.** Tudo fora de `$HOME` é perdido. |

⚠️ **Cuidado:** fechar o Cloud Shell ≠ minimizar. Fechar reseta a VM.

### 4. Persistência de variáveis de ambiente (`.profile`)

**Problema.** Você define `export REGION=us-central1` no Cloud Shell. Fecha a sessão. Volta no dia seguinte → variável sumiu. A VM foi reciclada.

**Solução em 3 camadas:**

1. **Guarde os valores em um arquivo dentro de `$HOME`** (ex: `~/infraclass/config`):
   ```bash
   mkdir -p ~/infraclass
   echo "INFRACLASS_REGION=us-central1" >> ~/infraclass/config
   echo "INFRACLASS_PROJECT_ID=meu-projeto-123" >> ~/infraclass/config
   ```
   Esse arquivo sobrevive porque está em `$HOME`.

2. **Carregue manualmente com `source`** sempre que abrir o shell:
   ```bash
   source ~/infraclass/config
   echo $INFRACLASS_REGION   # us-central1
   ```

3. **Automatize via `.profile`** para não precisar lembrar:
   ```bash
   nano ~/.profile
   # adicione no final:
   source ~/infraclass/config
   ```
   Salva (`Ctrl+O`, `Enter`) e sai (`Ctrl+X`). A partir daí, **toda sessão nova do Cloud Shell já abre com as variáveis carregadas**.

**Por que `.profile` e não `.bashrc`?**
O Cloud Shell abre como **shell de login**, e nesse modo o bash executa `.profile` (ou `.bash_profile`/`.profile`, nessa ordem). O `.bashrc` é lido por shells **interativos não-login** — não é o caso aqui. Pra Cloud Shell, `.profile` é o caminho canônico.

---

## Comandos-chave do lab

```bash
# Listar regiões disponíveis
gcloud compute regions list

# Criar um bucket pelo CLI (nome precisa ser globalmente único no GCP inteiro)
gcloud storage buckets create gs://meu-bucket-unico-123

# Copiar arquivo local pro bucket
gcloud storage cp arquivo.txt gs://meu-bucket-unico-123

# Se o nome tiver espaço, use aspas simples
gcloud storage cp 'meu arquivo.txt' gs://meu-bucket-unico-123

# Verificar variável de ambiente
echo $INFRACLASS_REGION

# Carregar variáveis de um arquivo
source ~/infraclass/config
```

📌 **Substituição do `gsutil`:** o comando antigo era `gsutil cp ... gs://...`. A Google migrou tudo para `gcloud storage` (mais rápido, integrado ao gcloud). Em prova/material antigo, você ainda verá `gsutil` — saiba que **são equivalentes funcionais**.

---

## Questionários

### Q1 — Cloud Shell oferece quais recursos? (multi-select)
✅ **5 GB de armazenamento persistente (`/home`)** — correto, esse é o limite oficial.
❌ Uma CLI que requer instalar o Cloud SDK — errado, o SDK já vem instalado.
✅ **Built-in authorization para acesso a recursos e instâncias** — correto, autenticação automática.
✅ **Command-line access a uma VM temporária gratuita do Compute Engine** — correto.

### Q2 — Para criar estado persistente no Cloud Shell, qual arquivo configurar?
✅ **`.profile`**

Lógica: é o arquivo executado automaticamente em shells de login (caso do Cloud Shell). Colocando `source ~/infraclass/config` lá, toda nova sessão recarrega as variáveis automaticamente. `.bashrc` não é executado em shell de login; `.my_variables` e `.config` não são arquivos padrão do bash.

---

## Cenário real

**Quando usar Console:**
- Demonstrar para colega/cliente (visual ajuda).
- Explorar um serviço novo do GCP (descobrir features via menus).
- Conceder uma role IAM ad-hoc para alguém.
- Diagnosticar visualmente: ver logs no Cloud Logging, métricas no Monitoring.

**Quando usar Cloud Shell:**
- Provisionar **N recursos parecidos** (loop bash com `gcloud`).
- Reproduzir setup do zero: roda um script e tudo está montado.
- Trabalhar de qualquer PC sem instalar nada.
- Pipeline de CI/CD em fase de prototipagem (depois migra pra Cloud Build).

**Quando usar SDK instalado localmente:**
- Volume alto de operações (Cloud Shell tem cota de uso/horas).
- Integração com ferramentas locais (IDE, Terraform, scripts versionados).
- Trabalho offline com cache local.

---

## Armadilhas de prova / pegadinhas

1. **Bucket name é globalmente único.** Não é único por projeto nem por região — é único **em todo o GCP, no mundo todo**. Se alguém em outra empresa já criou `gs://logs`, você nunca conseguirá esse nome.

2. **Cloud Shell ≠ sua máquina pessoal.** A VM é descartável. **Não armazene segredos, chaves SSH privadas ou certificados** lá sem entender o ciclo de vida.

3. **Apenas `/home` persiste.** Pacotes instalados via `sudo apt install` somem na reciclagem. Para persistir uma ferramenta, ou (a) coloque o binário em `~/bin` e adicione ao `PATH` no `.profile`, ou (b) use um container customizado do Cloud Shell.

4. **`.profile` vs `.bashrc`.** Em prova do mundo Cloud Shell, sempre `.profile`. `.bashrc` é distração.

5. **Console pode estar atrasado em relação à CLI.** Se uma feature foi lançada hoje, frequentemente ela só está em `gcloud` (às vezes só em `gcloud beta` ou `gcloud alpha`).

6. **"Open in a new window" não desconecta a sessão.** Só muda o container visual. "Close terminal" sim — esse encerra e reseta.

---

## Resumo de uma linha

> **Console** = veloz, seguro, descoberta. **Cloud Shell** = controle total, scriptável, automatizável. **`$HOME` persiste**, o resto não. **`.profile`** é onde se monta o estado inicial.
