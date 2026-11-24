# Views públicas — CRM

*Views* somente-leitura que o CRM publica para relatórios e agregações (Contrato §9.8). Tela
operacional nunca lê *view*: usa a API. Relatórios que cruzam módulos são do motor do Grupo 3.
Nenhuma das duas traz dado de contato (nome, e-mail, telefone).

## `crm.vw_pub_empresas`

Empresas ativas, sem as excluídas. As seis primeiras colunas são da V1; as demais entraram na V5,
acrescentadas ao fim para não quebrar quem já lia a *view*.

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | `uuid` | Identificador da empresa (o `empresaId` dos outros módulos) |
| `tenant_id` | `uuid` | Tenant dono do registro — **quem lê filtra por ele**, com o valor do token do usuário |
| `razao_social` | `varchar(255)` | Razão social |
| `cnpj` | `varchar(14)` | Só dígitos; pode ser nulo |
| `cidade` | `varchar(100)` | Cidade |
| `created_at` | `timestamptz` | Quando foi cadastrada, em UTC |
| `segmento` | `varchar(100)` | Segmento de mercado |
| `porte` | `varchar(50)` | Porte |
| `estado` | `varchar(2)` | UF |
| `status_comercial` | `varchar(20)` | `lead`, `prospect`, `cliente_ativo` ou `inativo` |
| `vendedor_responsavel` | `uuid` | Usuário responsável; pode ser nulo |
| `origem` | `varchar(30)` | Módulo ou canal que originou o cadastro |

## `crm.vw_pub_oportunidades`

Uma linha por oportunidade, sem as excluídas, já com o funil e a etapa (nome e ordem), para
o relatório de funil não precisar de outra *view*.

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | `uuid` | Identificador da oportunidade |
| `tenant_id` | `uuid` | Tenant dono do registro — **quem lê filtra por ele** |
| `empresa_id` | `uuid` | Empresa (`crm.vw_pub_empresas.id`) |
| `funil_id` | `uuid` | Funil |
| `funil_nome` | `varchar(100)` | Nome do funil |
| `etapa_id` | `uuid` | Etapa atual |
| `etapa_nome` | `varchar(100)` | Nome da etapa atual |
| `etapa_ordem` | `integer` | Posição da etapa no funil, a partir de 1 |
| `tipo` | `varchar(20)` | `nova`, `upsell` ou `cross_sell` |
| `produto_id` | `uuid` | Item do catálogo de Produtos; pode ser nulo |
| `valor_implantacao` | `numeric(15,2)` | Valor de implantação |
| `valor_mrr` | `numeric(15,2)` | Valor recorrente mensal |
| `probabilidade` | `integer` | 0 a 100 |
| `data_prevista_fechamento` | `date` | Previsão de fechamento |
| `responsavel_id` | `uuid` | Vendedor responsável |
| `equipe_id` | `uuid` | Equipe |
| `status` | `varchar(20)` | `aberta`, `ganha` ou `perdida` |
| `motivo_perda` | `varchar(255)` | Preenchido quando `perdida` |
| `fechada_em` | `timestamptz` | Quando foi ganha ou perdida, em UTC |
| `origem` | `varchar(30)` | Módulo ou canal de origem |
| `created_at` | `timestamptz` | Quando foi criada, em UTC |

**Quem pode ler:** `usr_chamados` (motor de relatórios). `vw_pub_empresas` também `usr_financeiro`.

**Concedido nas migrations** `V1__crm.sql` e `V5__views_relatorios.sql`, rodando como `own_crm`:

```sql
DO $$
BEGIN
  IF EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'usr_chamados') THEN
    GRANT USAGE  ON SCHEMA crm               TO usr_chamados;
    GRANT SELECT ON crm.vw_pub_oportunidades TO usr_chamados;
  END IF;
END $$;
```
