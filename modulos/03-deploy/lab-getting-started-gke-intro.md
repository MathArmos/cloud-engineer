# Lab — Console, Cloud Shell Editor e nginx em Compute Engine

**Curso Skills Boost:** Getting Started with Google Kubernetes Engine (Module 1 — Introduction to Google Cloud)
**Domínio relacionado:** transversal (toca 03-deploy 3.1 Compute Engine + 3.4 Cloud Storage + setup-environment Cloud Shell)
**Status:** ✅ concluído em 2026-05-17

---

## Objetivo do lab

Fechar o ciclo prático "**criar VM → publicar conteúdo estático**" usando três interfaces diferentes do GCP de forma encadeada:

1. **Cloud Console (GUI)** — criar a VM, abrir SSH, navegar entre serviços.
2. **Cloud Shell Editor (IDE no browser)** — editar arquivos (`cleanup.sh`, `index.html`).
3. **Cloud Shell CLI** — clonar repo, transferir arquivo pra VM via `gcloud compute scp`.
4. **SSH na VM** — instalar nginx, copiar arquivo pro document root.

O Google quer que o aluno saia entendendo que **as três interfaces se complementam**: a GUI descobre, o Editor edita, a CLI transfere, o SSH executa. Cada uma é forte numa parte do fluxo.

---

## Conceitos novos / reforçados

### 1. Cloud Shell Editor (Theia/VS Code-based)

**O quê.** Editor de código completo embutido no Cloud Shell, acessível via botão **"Open Editor"** (ícone de lápis). Roda Theia (fork open-source do VS Code) e abre na mesma sessão do Cloud Shell.

**Recursos relevantes:**
- File tree à esquerda — lista arquivos/pastas de `$HOME`.
- Terminal integrado — alterna entre código e shell sem trocar de aba.
- Pode abrir qualquer pasta dentro de `$HOME` via `File > Open Folder`.
- Edições são salvas **automaticamente** (sem `Ctrl+S` obrigatório).

**Quando usar.**
- Editar scripts, manifests YAML, Dockerfiles, HTML sem precisar de `vim`/`nano`.
- Inspecionar repos clonados visualmente antes de rodar.
- Demonstrar para alguém ao vivo (mais visual que terminal puro).

**Armadilha.** O Editor sempre opera dentro de `$HOME` da VM do Cloud Shell. Se a VM for reciclada, qualquer coisa fora de `$HOME` some — mesma regra da CLI.

### 2. `gcloud compute scp` — onde precisa rodar

**O quê.** Comando do gcloud que faz **secure copy** (SCP) entre seu host local (Cloud Shell) e uma VM do Compute Engine, abstraindo a configuração de chave SSH.

**Forma:**
```bash
gcloud compute scp ARQUIVO_LOCAL NOME_VM:CAMINHO_REMOTO --zone=ZONA
```

**Detalhe crítico (caiu no lab):** este comando precisa rodar **no Cloud Shell** (ou na máquina local com SDK instalado), **NÃO dentro da VM via SSH**. A VM tem scopes limitados por padrão e responderá `Request had insufficient authentication scopes`.

**Quando precisa rodar dentro da VM, use `scp` nativo** (não `gcloud compute scp`).

### 3. Geração de chave SSH na primeira execução

Na **primeira vez** que você roda `gcloud compute ssh` ou `gcloud compute scp` numa nova sessão do Cloud Shell, o gcloud:

1. Detecta que não existe par de chaves em `~/.ssh/google_compute_engine`.
2. Pergunta se pode criar `~/.ssh/` e gerar a chave.
3. Roda `ssh-keygen` (pode pedir passphrase — pressione **Enter** duas vezes para empty).
4. Publica a chave pública no metadata do projeto (`oslogin` ou metadata SSH keys).
5. Conecta usando essa chave.

A partir daí, toda chamada usa a mesma chave — não pergunta de novo.

### 4. Document root do nginx (Debian/Ubuntu)

**Pasta padrão:** `/var/www/html`.

Ao instalar nginx via `apt-get install nginx`, o pacote já configura:
- Serviço systemd `nginx.service` habilitado e rodando.
- `/etc/nginx/sites-available/default` apontando `root /var/www/html;`.
- Página `index.html` de boas-vindas pré-instalada lá.

Para servir conteúdo customizado, basta **copiar o `index.html` por cima** (precisa `sudo` porque a pasta pertence a `root`).

### 5. Quotas no Skills Boost / Qwiklabs

O lab usa um **bucket pré-provisionado** pelo Qwiklabs com nome do tipo `qwiklabs-gcp-PROJECT_ID-bucketN` contendo o `cat.jpg`. A URL pública segue o padrão:

```
https://storage.googleapis.com/BUCKET_NAME/cat.jpg
```

Esse URL precisa estar **explícito no HTML** — o texto `REPLACE_WITH_CAT_URL` que vem no template é **placeholder**, não a URL real.

---

## Comandos-chave executados

### No Cloud Shell

```bash
# 1. Garantir que está no $HOME (não em /home, que pertence ao sistema)
cd ~

# 2. Clonar o repo do codelab
git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git

# 3. Criar diretório de teste
mkdir test

# 4. Inspecionar o cleanup.sh e validar a edição feita no Editor
cd orchestrate-with-kubernetes
cat cleanup.sh

# 5. Transferir o index.html pra VM
gcloud compute scp index.html first-vm:index.html --zone=us-east1-c
```

### Edição via Cloud Shell Editor

- Adicionada linha `echo Finished cleanup!` no final de `cleanup.sh` (save automático).
- Criado `index.html` em `~/orchestrate-with-kubernetes/` com o HTML do gato e a URL real do bucket.

### Na SSH da VM (`first-vm`)

```bash
# Limpar man-db (workaround pra apt-get update mais rápido em VMs Debian)
sudo apt-get remove -y --purge man-db
sudo touch /var/lib/man-db/auto-update

# Atualizar índice de pacotes e instalar nginx
sudo apt-get update
sudo apt-get install nginx

# Servir o index.html customizado
sudo cp index.html /var/www/html/

# Verificar
cat /var/www/html/index.html
```

---

## Troubleshooting — erros que rolaram no lab

| Sintoma | Causa raiz | Fix |
|---|---|---|
| `git clone` retornou `Permission denied` em `/home` | Sessão estava em `/home` (parent dir, só root escreve) em vez de `$HOME` | `cd ~` antes do clone |
| `gcloud compute scp` falhou com `insufficient authentication scopes` | Comando foi rodado **dentro da VM via SSH** (cuja service account não tem `compute` scope) | Rodar o `scp` no **Cloud Shell**, não dentro da VM |
| `gcloud compute scp` pediu pra gerar chave SSH | Primeira execução numa sessão nova do Cloud Shell | Aceitar (`y`) e dar Enter duas vezes em passphrase |
| Browser mostrou ícone quebrado em vez do gato | HTML servia URL placeholder `qwiklabs-Google Cloud-1aeffbc5d0acb416` em vez da URL real do bucket Qwiklabs | Substituir `REPLACE_WITH_CAT_URL` pela Public URL real do `cat.jpg` no bucket do lab (ex: `https://storage.googleapis.com/qwiklabs-gcp-00-XXXX-bucket2/cat.jpg`) |

---

## Cenário real

**Onde isso aparece no trabalho:**

- **Bootstrap rápido de web app estática numa VM** — útil quando o time precisa de uma página interna (status page, demo) sem provisionar Cloud Run ou LB.
- **Pipeline de deploy artesanal** — Editor → SCP → SSH é o "deploy de emergência" antes de configurar CI/CD. Comum em POCs.
- **Debugging de VMs em prod** — entender que `scp` precisa scopes corretos evita 30 minutos perdidos copiando arquivo do lugar errado.

---

## Armadilhas de prova / pegadinhas

1. **`gcloud compute scp` só funciona de fora da VM por padrão.** Service account default da VM tem scope reduzido. Para rodar de dentro da VM, ou (a) recriar a VM com `--scopes=cloud-platform`, ou (b) usar `scp` nativo do OpenSSH.

2. **`/home` ≠ `$HOME` no Cloud Shell.** `/home` é o pai de todos os homes — você não tem write access lá. Sempre `cd ~` ou use caminho absoluto `/home/USERNAME/...`.

3. **Cloud Shell Editor salva automaticamente.** Nada de `Ctrl+S` obrigatório, mas confirme com `cat ARQUIVO` no terminal antes de SCPar.

4. **Document root do nginx é `/var/www/html` no Debian/Ubuntu.** No CentOS/RHEL é `/usr/share/nginx/html`. Cai em pergunta de "onde fica a página default".

5. **Placeholder de URL é literal.** Se o exercício diz "substitua `REPLACE_WITH_CAT_URL` por uma URL como `https://storage.googleapis.com/qwiklabs-Google Cloud-XXX/cat.jpg`", esse último link é **exemplo de formato**, não URL pra colar. Vá no bucket real e copie a Public URL de lá.

6. **Primeira execução de SSH/SCP via gcloud gera chave.** Em VMs novas do Qwiklabs ou Cloud Shell, espere essa interação extra na primeira chamada — não é erro.

---

## Resumo de uma linha

> **Console** descobre, **Editor** edita, **CLI do Cloud Shell** transfere, **SSH na VM** executa. `gcloud compute scp` mora no Cloud Shell, nginx mora em `/var/www/html`, e placeholder de URL é literal — substitua pela real.
