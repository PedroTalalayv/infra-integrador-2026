# Views públicas — financeiro

*Views* somente-leitura que o Financeiro publica para relatórios e agregações (Contrato §9.8). Tela
operacional nunca lê *view*: usa a API. Relatórios que cruzam módulos são do motor do Grupo 3 (RN-59).

## `financeiro.vw_pub_cobrancas`

Contas a receber, sem as canceladas (excluídas logicamente). Não traz dado cadastral da empresa: só
o UUID do CRM (RN-03).

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | `uuid` | Identificador da cobrança |
| `tenant_id` | `uuid` | Tenant dono do registro — **quem lê filtra por ele**, com o valor do token do usuário |
| `empresa_id` | `uuid` | Empresa no CRM (o `clienteId` da API) |
| `contrato_id` | `uuid` | Contrato no módulo Contratos, quando houver |
| `competencia` | `char(7)` | Mês de competência, `AAAA-MM` |
| `data_vencimento` | `date` | Vencimento |
| `valor` | `numeric(15,2)` | Valor original |
| `desconto` | `numeric(15,2)` | Desconto |
| `juros` | `numeric(15,2)` | Juros |
| `multa` | `numeric(15,2)` | Multa |
| `valor_pago` | `numeric(15,2)` | Total já recebido (saldo = valor − desconto + juros + multa − valor_pago) |
| `forma_pagamento` | `varchar(15)` | `PIX`, `BOLETO`, `CARTAO` ou `TRANSFERENCIA` |
| `status` | `varchar(12)` | `ABERTO`, `AGENDADO`, `PAGO`, `ATRASADO`, `ESTORNADO` ou `PARCIAL` |
| `created_at` | `timestamptz` | Quando foi lançada, em UTC |

**Quem pode ler:** `usr_chamados` (motor de relatórios).

**Concedido na migration** `V1__schema_financeiro.sql`, rodando como `own_financeiro`:

```sql
DO $$
BEGIN
  IF EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'usr_chamados') THEN
    GRANT USAGE  ON SCHEMA financeiro          TO usr_chamados;
    GRANT SELECT ON financeiro.vw_pub_cobrancas TO usr_chamados;
  END IF;
END $$;
```
