---
name: ACE Section 3 Diagnostic Answers (Official)
description: Official answers and rationale for Section 3 diagnostic questions (Deploying and Implementing a Cloud Solution) from Google's ACE prep guide
type: reference
originSessionId: 488c0de8-00cc-4df2-8923-ea2633411d9c
---
# Seção 3 — Respostas Oficiais (Google Cloud ACE Prep Guide)

Respostas verificadas no PDF oficial "T-GCPACE-m3-l7-en-file-21". Use esta referência ao revisar com o usuário.

## 3.1 Compute Engine

### Q1: Migração MySQL com UDFs (rápido + econômico)
**Resposta correta: C — Compute Engine VM com N2, instalar MySQL e restaurar**
- N2 = "balanced machine type" → recomendado para BDs médios/grandes
- E2 = cost-optimized (não recomendado para BD médio)
- Cloud SQL → **NÃO suporta UDFs** (eliminado)
- Marketplace → requer config manual adicional (não é o mais rápido)

### Q2: Atualizar OS em MIG (automatizado, recursos mínimos)
**Resposta correta: B — Novo template + Update VMs + tipo PROACTIVE**
- PROACTIVE = rolling update, surge=1 automaticamente (mínimo de recursos)
- Opportunistic = não é automatizado
- Max surge 5 = NÃO usa recursos mínimos
- Abandonar instâncias = não é automático

## 3.2 GKE

### Q3: Cluster Kubernetes piloto (privado, não-HA, arquitetura flexível)
**Resposta correta: B — Cluster zonal standard privado em us-central1-a, default pool, Ubuntu**
- Standard clusters podem ser zonais (Autopilot é sempre regional)
- Autopilot é gerenciado a nível de pod (não permite alterar arquitetura)
- Container-optimized image NÃO suporta pacotes customizados
- Autopilot NÃO suporta Ubuntu

## 3.3 Cloud Run / Cloud Functions

### Q4: App containerizado, serverless, pay-per-request, pacotes custom
**Resposta correta: C — Cloud Run**
- App Engine flexible → NÃO escala para zero
- App Engine standard → NÃO permite pacotes customizados
- Cloud Run Functions → não usa containers para deploy da lógica

### Q5: Trigger Cloud Storage para Cloud Run Function (arquivos adicionados)
**Resposta correta: A — `--trigger-event google.storage.object.finalize`**
- `finalize` = evento disparado quando write no Cloud Storage está completo
- `create`, `change`, `add` → NÃO são eventos válidos do Cloud Storage

## 3.4 Data Solutions

### Q6: Bucket Cloud Storage para NY e SF, sem ACLs
**Resposta correta: C — `gcloud storage buckets create` SEM `--location`**
- Sem `--location` → cria por padrão em **US multi-region** (cobre NY e SF, exclui Londres)
- `gsutil mb` → CLI sendo descontinuado (minimamente mantido)
- `--placement` → só aceita regiões do mesmo continente (us-east1 + europe-west2 inválido)
- `gcloud storage objects --remove-acl-grant` → remove ACL de usuário, não cria bucket

### Q7: Cloud SQL failover automático em caso de falha zonal
**Resposta correta: A — `--availability-type`**
- Define availability zonal vs regional (regional = failover automático)
- `--replica-type` → define tipo de réplica (read/failover legacy)
- `--secondary-zone` → só válido com availability-type regional
- `--master-instance-name` → cria read replica, NÃO automatiza failover

### Q8: Carregar dados slowly changing no BigQuery (custo mínimo, menos etapas)
**Resposta correta: D — BigQuery Data Transfer Service**
- É o processo mais simples (1 comando) e **gratuito**
- `bq load` + cron → funciona mas é mais complexo
- Streaming API → tem custos baseados em volume
- Dataflow pipeline → incorre em custos de recursos

## 3.5 Networking

### Q9: VPC com controle total sobre IP ranges e sub-redes
**Resposta correta: C — Custom mode network**
- Custom mode = controle total sobre regiões e IP ranges
- Auto mode / Default = sub-redes criadas automaticamente com IPs predeterminados
- Auto convertido em custom = mantém IPs já atribuídos, requer passos extras

## 3.6 Infrastructure as Code

### Q10: Comando `terraform apply`
**Resposta correta: D — Configura os recursos solicitados no arquivo de config**
- `terraform init` → baixa última versão do provider
- `terraform plan` → verifica sintaxe e mostra preview
- `terraform apply` → cria/configura os recursos
- `terraform destroy` → destrói recursos
