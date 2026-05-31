# r-hotfix-padrao
versao: 1.0

## OBJETIVO

Gerar patches cirúrgicos em arquivos de aplicação sem gerar o arquivo completo.

---

## REGRAS

- alterar apenas o necessário
- manter compatibilidade com schema atual
- evitar refatorações grandes
- preservar comportamento existente

---

## FORMATO OBRIGATÓRIO

1. CAUSA-RAIZ
2. IMPACTO
3. CORREÇÃO

Depois:

- linha aproximada
- trecho ANTES
- trecho DEPOIS

---

## LIMITES

- preferir patches menores que 50 linhas
- se ultrapassar 100 linhas:
  solicitar validação humana

---

## PROIBIÇÕES

- nunca gerar o arquivo completo sem solicitação explícita
- nunca alterar múltiplos fluxos em um único patch
- nunca alterar CSS sem solicitação
- nunca inventar trecho original

---

## COMPATIBILIDADE

Em projetos multi-tenant com Supabase, toda query deve respeitar:
- tenant_id
- RLS
- joins corretos conforme schema do projeto