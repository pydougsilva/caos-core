# r-rls-padrao
versao: 1.0

## OBJETIVO

Garantir isolamento seguro entre tenants utilizando Row Level Security (RLS).

---

## PRINCÍPIO CENTRAL

Toda policy multi-tenant deve respeitar:

tenant_id = get_tenant_id()

---

## REGRAS OBRIGATÓRIAS

### SELECT

Toda leitura deve:
- restringir tenant
- evitar exposição cruzada

---

### INSERT

Toda policy INSERT deve usar:
WITH CHECK

Exemplo lógico:
tenant_id = get_tenant_id()

---

### UPDATE

Toda atualização deve:
- validar tenant
- validar permissões

---

### DELETE

Nunca permitir:
DELETE global sem restrição

---

## FUNÇÕES OFICIAIS

Usar:

- get_tenant_id()
- is_tenant_admin()
- is_platform_admin()

---

## FUNÇÃO LEGADA

is_admin()

Status:
LEGADO

Nunca usar:
em contexto multi-tenant novo

---

## ROLE ANON

Role anon:
- apenas leitura pública controlada
- nunca operações administrativas

---

## JWT

tenant_id deve existir:
- no app_metadata
- no token JWT

Mudanças de tenant:
- podem exigir refresh de sessão

---

## POLÍTICAS PÚBLICAS

Policies anon devem:
- ser explícitas
- ter escopo reduzido
- nunca expor dados administrativos

---

## MULTI-TENANT

Toda tabela operacional deve possuir:
- tenant_id
- policy de isolamento
- índice apropriado

---

## AUDITORIA

Antes de aprovar policy:

verificar:
- USING
- WITH CHECK
- role
- tenant_id
- impacto frontend

---

## PROIBIÇÕES

Nunca:
- usar USING (true)
- usar anon para admin
- usar is_admin() legado
- criar policy sem tenant_id
- confiar apenas no frontend

---

## RESULTADO ESPERADO

RLS:
- previsível
- auditável
- segura
- compatível com Supabase
- sustentável para white label