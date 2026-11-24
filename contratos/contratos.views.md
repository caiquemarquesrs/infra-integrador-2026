# Views públicas — Contratos

*Views* somente-leitura que o módulo de Contratos publica para relatórios e agregações (Contrato §9.8).
Tela operacional nunca lê *view*: usa a API. Relatórios que cruzam módulos são do motor do Grupo 3.

## `contratos.vw_pub_contratos`

Uma linha por contrato, sem os excluídos: status, valores e vigência, para MRR, contratos a vencer
e renovações. Não traz o nome da empresa (cópia local): o nome vem de `crm.vw_pub_empresas`.

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | `uuid` | Identificador do contrato |
| `tenant_id` | `uuid` | Tenant dono do registro — **quem lê filtra por ele**, com o valor do token do usuário |
| `numero` | `varchar(20)` | `CT-AAAA-NNNNNN` |
| `empresa_id` | `uuid` | Empresa no CRM |
| `status` | `varchar(30)` | Um dos nove estados do RF-05 (`RASCUNHO` … `FINALIZADO`) |
| `valor_mensal` | `numeric(15,2)` | Valor mensal |
| `mrr` | `numeric(15,2)` | Receita recorrente mensal |
| `data_inicio` | `date` | Início da vigência |
| `vigencia_meses` | `integer` | Duração; nulo = indeterminado |
| `vigencia_fim` | `date` | Fim previsto (início + vigência); nulo = indeterminado |
| `renovacao` | `varchar(20)` | `automatica`, `manual` ou `sem_renovacao` |
| `indice_reajuste` | `varchar(10)` | `IPCA`, `IGPM`, `INPC` ou `nenhum` |
| `aviso_previo_dias` | `integer` | Dias de aviso prévio |
| `responsavel_id` | `uuid` | Usuário responsável |
| `assinado_em` | `timestamptz` | Quando foi assinado, em UTC; nulo antes da assinatura |
| `created_at` | `timestamptz` | Quando foi criado, em UTC |

## `contratos.vw_pub_servicos_contratados`

Uma linha por serviço de um contrato não excluído, para cruzar com `produtos.vw_pub_itens`
(vendas por item do catálogo).

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | `uuid` | Identificador da linha |
| `tenant_id` | `uuid` | Tenant dono do registro — **quem lê filtra por ele** |
| `contrato_id` | `uuid` | Contrato (`vw_pub_contratos.id`) |
| `item_catalogo_id` | `uuid` | Item em Produtos; pode ser nulo |
| `descricao` | `varchar(255)` | Descrição do serviço |
| `quantidade` | `numeric(15,4)` | Quantidade |
| `valor_unitario` | `numeric(15,2)` | Valor unitário |
| `valor_total` | `numeric(15,2)` | Valor total da linha |
| `created_at` | `timestamptz` | Quando foi incluído, em UTC |

**Quem pode ler:** `usr_chamados` (motor de relatórios).

**Concedido na migration** `V21__views_relatorios.sql`, rodando como `own_contratos`:

```sql
DO $$
BEGIN
  IF EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'usr_chamados') THEN
    GRANT USAGE  ON SCHEMA contratos                      TO usr_chamados;
    GRANT SELECT ON contratos.vw_pub_contratos            TO usr_chamados;
    GRANT SELECT ON contratos.vw_pub_servicos_contratados TO usr_chamados;
  END IF;
END $$;
```
