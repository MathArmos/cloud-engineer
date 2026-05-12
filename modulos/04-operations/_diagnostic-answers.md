---
name: ACE Section 4 Official Answers
description: Gabarito oficial das 9 questões diagnósticas da Seção 4 (Ensuring Successful Operations) do ACE — usuário tirou 100%
type: reference
originSessionId: 89cf7b4e-5dd9-4df8-b755-099227159369
---
# Gabarito Oficial — Seção 4 (Ensuring Successful Operations)

Usuário tirou **9/9 (100%)** nesta seção em 2026-05-11.

## Questões e respostas oficiais

| Q | Tema | Resposta correta |
|---|------|-----------------|
| 1 | Sintaxe gcloud — listar snapshots | `gcloud compute snapshots list` |
| 2 | Alert policy CPU > 60% por 5 min | VM instance + CPU **utilization** + trigger if **any** time series violates + condition **above** + threshold 0.60 / 5min |
| 3 | Objeto K8s que expõe lógica via endpoints | **Services** |
| 4 | Forma declarativa de inicializar/atualizar K8s | `kubectl apply` |
| 5 | Internal Application LB no GKE | **Annotate service object with "neg" reference** (NEGs obrigatórios para Internal HTTP(S) LB) |
| 6 | Lifecycle rule Standard → Nearline em data específica | **CreatedBefore** + **MatchesStorageClass** |
| 7 | Erro ao deletar snapshot agendado | **Detach the snapshot schedule** (não pode deletar schedule attached a PD) |
| 8 | Limitar conexões DB de Cloud Run | **Set Max instances** |
| 9 | Expandir subnet /24 para suportar 2000 IPs | `gcloud compute networks subnets expand-ip-range mysubnet --region us-central1 --prefix-length 21` (2046 IPs disponíveis) |

## Justificativas oficiais (do feedback do quiz)

- **Q1**: gcloud segue padrão `<grupo> <subgrupo> <verbo>` — verbo no final
- **Q3**: Service endpoints definidos por pods com labels que casam com a configuração do service
- **Q4**: `kubectl apply` cria E atualiza objetos K8s de forma declarativa via manifest files
- **Q5**: Internal Application LB **só pode usar NEGs** (Network Endpoint Groups)
- **Q6**: `CreatedBefore` permite especificar data; `MatchesStorageClass` filtra objetos Standard
- **Q7**: Não dá pra deletar uma snapshot schedule que ainda está attached a um PD
- **Q8**: Max instances controla custos e limita conexões com serviços de backend
- **Q9**: /21 = 2046 endereços disponíveis (atende 2000 usuários)
