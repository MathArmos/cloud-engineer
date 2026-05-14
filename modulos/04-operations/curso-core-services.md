# Curso: Essential Google Cloud Infrastructure: Core Services

> **Duração:** 8h 15min · **Status:** 🟡 em andamento  
> **Seções da prova cobertas:** 1.1 (IAM), 3.4 (Storage/DB), 4.1 (GCE mgmt), 4.4 (Storage mgmt), 4.6 (Monitoring)

---

## Módulo 1 — Cloud IAM

### 1.1 — Introdução ao IAM

**IAM (Identity and Access Management)** é o sistema do Google Cloud para controlar *quem* pode fazer *o quê* em *quais recursos*.

**Componentes do IAM:**

| Componente | Descrição |
|---|---|
| **Organizations** | Nó raiz da hierarquia de recursos |
| **Roles** | Coleção de permissões agrupadas por "tipo de trabalho" |
| **Members** | Identidades que recebem roles (usuários, grupos, service accounts) |
| **Service Accounts** | Identidades para aplicações/VMs, não para humanos |

**Destaques do IAM no GCP:**
- Identidades usam formato de **endereço de e-mail**
- Roles são baseadas em **tipo de função** (job-type roles), não permissões avulsas
- Permissões são **granulares** — maior controle com menor superfície de ataque
- Inclui o recurso **Organization Restrictions** para limitar escopo de acesso

> **Tópicos do módulo:** Organizations → Roles → Members → Service Accounts → Organization Restrictions → Best Practices → Lab

---

### 1.2 — O que é IAM? + Resource Hierarchy

**Definição central:**

> **Quem** pode fazer **o quê** em **qual recurso**

| Elemento | Descrição | Exemplo |
|---|---|---|
| **Quem** (Who) | Pessoa, grupo ou aplicação | `user@example.com` |
| **O quê** (What) | Privilégios ou ações específicas | Role: `Compute Viewer` |
| **Qual recurso** (Which) | Qualquer serviço do GCP | Compute Engine, Cloud Storage... |

**Exemplo prático:** Role `Compute Viewer` → acesso read-only para get/list de recursos Compute Engine, **sem** acesso aos dados armazenados neles.

---

#### Hierarquia de Recursos do IAM

```
Organization  (example.com)          ← nó raiz
    └── Folders  (Dept X, Dept Y)     ← representa departamentos
            └── Projects              ← trust boundary
                    └── Resources     ← instâncias, buckets, topics...
```

**Regras críticas:**

| Regra | Detalhe |
|---|---|
| Cada recurso tem **exatamente 1 pai** | Sem herança múltipla |
| Policies são **herdadas para baixo** | Role no org → vale para tudo abaixo |
| **Organization** = sua empresa | Roles aqui valem para todos os recursos |
| **Folder** = departamento | Roles aqui valem para tudo dentro da pasta |
| **Project** = trust boundary | Serviços no mesmo projeto têm o mesmo nível de confiança padrão |

> **Pegadinha de prova:** herança é sempre **top-down** — um role concedido num nível superior não pode ser negado num nível inferior.

---

### 1.3 — Organization Node e Folders

#### Organization Node

O nó raiz da hierarquia. É provisionado automaticamente quando um usuário com conta **Google Workspace** ou **Cloud Identity** cria um projeto GCP.

**Roles principais na Organization:**

| Role | Quem recebe | Responsabilidades |
|---|---|---|
| **Org Admin** | Bob (admin de TI) | Define IAM policies, estrutura da hierarquia, delega billing/networking via roles |
| **Project Creator** | Alice | Cria projetos dentro da org (pode ser aplicado no nível da org e herdado) |
| **Org Viewer** | Auditores | View-only em todos os recursos da org |

**Dois perfis críticos na criação da org:**

| Perfil | Quem é | Responsabilidades |
|---|---|---|
| **Workspace/Cloud Identity Super Admin** | Admin do domínio | Atribui o Org Admin role; ponto de contato para recovery; controla lifecycle da conta |
| **GCP Organization Admin** | Admin do GCP | Define IAM policies; determina estrutura da hierarquia; delega roles críticos |

> **Importante:** Os dois roles são geralmente atribuídos a **pessoas diferentes**. O Org Admin, por princípio de least privilege, **não inclui** permissão para criar folders — precisa atribuir roles adicionais à própria conta.

---

#### Folders

Folders são **sub-organizações** dentro da org — mecanismo de agrupamento e isolamento entre projetos.

**Usos típicos:**

| Nível de Folder | Representa |
|---|---|
| 1º nível | Departamentos (Dept X, Dept Y, Shared Infrastructure) |
| 2º nível | Times dentro do departamento (Team A, Team B) |
| 3º nível | Produtos/aplicações (Product 1, Product 2) |

**Roles de Folder:**

| Role | O que faz |
|---|---|
| **Folder Admin** | Controle total sobre folders |
| **Folder Creator** | Navega na hierarquia e cria folders |
| **Folder Viewer** | Visualiza folders e projetos abaixo do recurso |

**Roles de Project (nível resource manager):**

| Role | O que faz |
|---|---|
| **Project Creator** | Cria projetos → torna-se automaticamente **Owner** do projeto |
| **Project Deleter** | Permissão para deletar projetos |

> **Delegação via Folders:** o chefe de um departamento pode receber full ownership de todos os recursos dentro da sua folder. Usuários de um departamento só acessam recursos dentro da sua folder — isolamento por design.

---

## Módulo 2 — Storage and Database Services

<!-- colar conteúdo aqui -->

---

## Módulo 3 — Resource Management

<!-- colar conteúdo aqui -->

---

## Módulo 4 — Resource Monitoring

<!-- colar conteúdo aqui -->

---

## Labs completados

<!-- documentar labs aqui -->

---

## DQs cobertas por este curso

<!-- listar quais DQs este curso esclarece -->

---

## Conexões com arquivos de subseção

<!-- ao terminar o curso, listar quais arquivos foram atualizados/criados com base neste conteúdo -->
