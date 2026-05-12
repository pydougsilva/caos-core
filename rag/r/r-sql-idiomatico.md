# r-sql-idiomatico
versao: 1.0

## OBJETIVO

Garantir geração segura, compatível e idempotente de scripts SQL para PostgreSQL 17.6.

---

## PRINCÍPIOS

Todo script deve:
- ser idempotente
- poder ser reexecutado
- evitar downtime
- preservar integridade multi-tenant

---

## REGRAS OBRIGATÓRIAS

### CRIAÇÃO

Usar:
- CREATE TABLE IF NOT EXISTS
- CREATE INDEX IF NOT EXISTS

---

### ALTERAÇÕES

Antes de:
- ADD CONSTRAINT

Executar:
- DROP CONSTRAINT IF EXISTS

Motivo:
PostgreSQL 17.6 não suporta:
ADD CONSTRAINT IF NOT EXISTS

---

### INSERTS

Usar:
ON CONFLICT DO NOTHING

Quando apropriado:
UPSERT controlado

---

### MIGRAÇÕES

Fluxo obrigatório:

1. ADD COLUMN nullable
2. backfill
3. validar dados
4. SET NOT NULL

Nunca:
- criar NOT NULL diretamente em produção

---

### DO $$

Usar para:
- verificações condicionais
- lógica procedural
- proteção contra duplicação

---

## MULTI-TENANT

Toda tabela operacional deve considerar:
- tenant_id
- RLS
- índices compostos quando necessário

---

## SEGURANÇA

Nunca:
- usar anon para admin
- ignorar RLS
- confiar apenas no frontend

---

## CABEÇALHO OBRIGATÓRIO

Todo script deve conter:

- projeto
- fase
- data
- objetivo
- pré-requisitos

---

## SEÇÃO FINAL

Todo script deve incluir:

- verificações SQL comentadas
- próximos passos
- riscos conhecidos

---

## PROIBIÇÕES

- DROP sem IF EXISTS
- SQL não idempotente
- ALTER destrutivo silencioso
- hardcode inseguro
- migrations gigantes sem validação

---

## RESULTADO ESPERADO

Scripts:
- previsíveis
- auditáveis
- seguros
- compatíveis com Supabase
- reutilizáveis