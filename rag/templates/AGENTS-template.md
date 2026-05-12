# [NOME DO PROJETO] — Infraestrutura Cognitiva Local
versao: 1.0

## IDENTIDADE

O [NOME DO PROJETO] utiliza o C.A.O.S como infraestrutura cognitiva operacional.

Arquitetura operacional:

- Claude → Orquestrador estratégico e raciocínio
- RAG (/rag/k e /rag/r) → memória modular estruturada
- Snapshots → memória institucional operacional persistente
- Codex → executor técnico controlado
- VSCode → ambiente operacional
- Usuário → validação e governança

---

## VISÃO DO PROJETO

[Descrever em 2-3 linhas o que o projeto é e qual problema resolve.]

---

## STACK

- Frontend: [ex: React 18 + Vite]
- Backend: [ex: Supabase + PostgreSQL 17.6]
- Deploy: [ex: Vercel]

---

## FASE ATUAL

[ex: Fase 2 — Implementação de autenticação]

---

## GOVERNANÇA

Nenhuma alteração estrutural deve ser executada sem validação humana.

Toda mudança deve:
1. ser proposta
2. explicada
3. validada
4. executada

---

## PADRÕES OBRIGATÓRIOS

### SQL
- scripts idempotentes
- [versão do PostgreSQL]
- tenant_id obrigatório (se multi-tenant)

### [Adicionar padrões específicos do projeto]

---

## FLUXO OPERACIONAL

0. Detectar domínio (r-auto-recuperacao-contextual)
0a. Matching por conceito (r-matching-conceito)
0b. Recuperar snapshot (r-recuperacao-contextual)
1. Classificar tarefa
2. Consultar rag/index.md
3. Carregar /r antes de /k (máximo 3 módulos)
4. Gerar instrução estruturada
5. Validar com usuário [GATE]
6. Executar

---

## RESTRIÇÕES

Nunca:
- improvisar arquitetura
- alterar múltiplos domínios sem aprovação
- executar sem instrução validada

---

## PRIORIDADE DE AUTORIDADE

1. AGENTS.md
2. Prompt de Sessão
3. rag/index.md
4. Inferência própria
