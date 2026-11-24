# Views públicas — Marketing

*View* somente-leitura que o Marketing publica para relatórios e agregações (Contrato §9.8).
Tela operacional nunca lê *view*: usa a API. Relatórios que cruzam módulos são do motor do Grupo 3.

## `marketing.vw_pub_leads`

Uma linha por lead, sem os excluídos. Serve para **contar e agrupar** leads, não para identificá-los:
não publica nome, e-mail, telefone, WhatsApp, respostas de formulário, IP nem token de descadastro
(LGPD). Leads anonimizados continuam na *view*, com `anonimizado = true`.

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | `uuid` | Identificador do lead |
| `tenant_id` | `uuid` | Tenant dono do registro — **quem lê filtra por ele**, com o valor do token do usuário |
| `etapa` | `smallint` | 0 a 3 |
| `temperatura` | `varchar(10)` | `frio`, `medio` ou `quente` (espelho da etapa) |
| `origem` | `varchar(20)` | `meta_ads`, `landing_page` ou `manual` |
| `canal` | `varchar(20)` | `formulario`, `landing_page`, `embed` ou `chatbot`; nulo em lead do Meta |
| `utm_source` | `varchar(255)` | UTM de origem |
| `utm_medium` | `varchar(255)` | UTM de mídia |
| `utm_campaign` | `varchar(255)` | UTM de campanha |
| `formulario_id` | `uuid` | Formulário do Landing; pode ser nulo |
| `pontuacao` | `int` | Pontuação de qualificação |
| `faixa_qualificacao` | `varchar(20)` | `mql_quente`, `mql_morno` ou `nutricao` |
| `qualificado_em` | `timestamptz` | Quando foi qualificado, em UTC |
| `vendedor_id` | `uuid` | Vendedor atribuído |
| `vendedor_atribuido_em` | `timestamptz` | Quando o vendedor foi atribuído, em UTC |
| `crm_empresa_id` | `uuid` | Empresa no CRM, depois do handoff |
| `crm_oportunidade_id` | `uuid` | Oportunidade no CRM, depois do handoff |
| `consentimento_marketing` | `boolean` | Aceitou comunicação de marketing |
| `status_aquecimento` | `varchar(20)` | `ativo`, `esgotado` ou `opt_out` |
| `anonimizado` | `boolean` | Dados pessoais já apagados |
| `created_at` | `timestamptz` | Quando o lead entrou, em UTC |

**Quem pode ler:** `usr_chamados` (motor de relatórios).

**Concedido na migration** `V14__view_relatorios.sql`, rodando como `own_marketing`:

```sql
DO $$
BEGIN
  IF EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'usr_chamados') THEN
    GRANT USAGE  ON SCHEMA marketing       TO usr_chamados;
    GRANT SELECT ON marketing.vw_pub_leads TO usr_chamados;
  END IF;
END $$;
```
