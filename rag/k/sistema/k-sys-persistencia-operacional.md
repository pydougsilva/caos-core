# k-sys-persistencia-operacional
versao: 1.0

## OBJETIVO

Definir o papel institucional do Git como camada de persistência
operacional verificável dentro do C.A.O.S.

Responde à pergunta:
"o que é a camada Git e como ela se relaciona com snapshots e RAG?"

---

## A STACK OPERACIONAL COMPLETA

O C.A.O.S opera com três camadas de persistência distintas e complementares:

```
┌─────────────────────────────────────────────────────────────┐
│  Camada 1 — RAG (/r e /k)                                   │
│  Como operar: regras e conhecimento estruturado             │
│  Persiste: diretrizes, padrões, domínios                    │
│  Muda quando: arquitetura evolui                            │
├─────────────────────────────────────────────────────────────┤
│  Camada 2 — Snapshots                                       │
│  O que foi decidido: decisões homologadas                   │
│  Persiste: contexto, riscos, resultado esperado             │
│  Muda quando: novo ciclo operacional ocorre                 │
├─────────────────────────────────────────────────────────────┤
│  Camada 3 — Git (v3.5)                                      │
│  O que foi executado: evidência verificável                 │
│  Persiste: diff real, autoria, timestamp                    │
│  Muda quando: Codex executa instrução validada              │
└─────────────────────────────────────────────────────────────┘
```

Cada camada responde uma pergunta diferente:

| Pergunta | Camada | Componente |
|---|---|---|
| "Como devo proceder?" | RAG /r | regras operacionais |
| "O que existe neste domínio?" | RAG /k | conhecimento estruturado |
| "O que foi decidido antes?" | Snapshot | memória institucional |
| "O que foi realmente executado?" | Git | evidência operacional |

Nenhuma camada substitui as outras.

---

## SEPARAÇÃO FUNDAMENTAL: DECISÃO ↔ EXECUÇÃO

Esta separação é o princípio central da camada Git no C.A.O.S.

### Snapshot — a decisão

Registra o que foi analisado, proposto, validado e decidido.

```yaml
snapshot: audit-logs-001
dominio: public.audit_logs
decisao: |
  Substituir policy baseada em email_admin por get_tenant_id()
  e adicionar cobertura para is_platform_admin()
resultado: sucesso
riscos_vistos: [cross-tenant, ausência platform_admin, performance]
```

O snapshot documenta a intenção institucional homologada.
Um snapshot pode existir sem commit correspondente.
Isso significa: decisão tomada, execução pendente.

### Commit — a execução

Registra o que foi efetivamente alterado, quando, por quem.

```
[rls](public.audit_logs): substituir policy email_admin por get_tenant_id()

snapshot: audit-logs-001
agent-executor: Codex
agent-orchestrator: Claude
risks-addressed: 4
```

O commit documenta a execução real com autoria verificável.
Um commit deve sempre ter snapshot correspondente.
Commit sem `snapshot:` é violação arquitetural — execução sem decisão.

### O que cada um não faz

| Snapshot | Commit |
|---|---|
| Não prova que algo foi executado | Não explica por que foi executado |
| Não contém o diff real | Não contém análise de riscos |
| Pode existir sem commit | Não deve existir sem snapshot |
| Imutável após criação | Imutável após push |

---

## VÍNCULO BIDIRECIONAL SNAPSHOT ↔ COMMIT

A rastreabilidade institucional exige que cada artefato referencie o outro.

### Snapshot referencia commit

Após a execução ser verificada, o snapshot é atualizado com:

```yaml
estado_atual: CONCLUÍDO
commit_hash: abc123f7
branch: ops/audit-logs-20260509
```

### Commit referencia snapshot

O commit message inclui obrigatoriamente:

```
snapshot: audit-logs-001
```

### Rastreabilidade resultante

```
Dado snapshot audit-logs-001:
  → commit_hash: abc123f7
  → git show abc123f7
  → diff exato das alterações realizadas

Dado commit abc123f7:
  → snapshot: audit-logs-001
  → decisao, contexto, riscos, agentes do ciclo
```

A rastreabilidade é completa nos dois sentidos.
Qualquer agente, em qualquer sessão futura, pode reconstruir
o ciclo completo a partir de qualquer um dos dois artefatos.

---

## O QUE É UM COMMIT INSTITUCIONAL

Um commit é institucional quando:

1. Seu prefixo segue o formato `[tipo](domínio):`
2. Contém o campo `snapshot:` com ID válido
3. Declara `agent-executor:` e `agent-orchestrator:`
4. Declara `risks-addressed:`
5. Inclui `Co-Authored-By:` e `Orchestrated-By:`
6. Foi criado em branch `ops/`, não diretamente em `main`
7. Representa exatamente os artefatos listados no handoff correspondente

Commits que não atendem a esses critérios não são reconhecidos como
eventos institucionais do C.A.O.S — são commits técnicos comuns.

---

## FLUXO COMPLETO COM EVIDÊNCIA OPERACIONAL

```
Etapa 5    Claude gera instrução estruturada
           ↓
Etapa 6    Usuário valida [GATE 1]
           ciclo: PROPOSTO → VALIDADO
           ↓
Etapa 7    Claude emite handoff (estado: VALIDADO)
           commit_type e branch_sugerido incluídos
           ciclo: VALIDADO → EXECUTANDO
           ↓
Etapa 7a   Codex cria branch ops/ e executa instrução
           ↓
Etapa 7a   Codex cria commit institucional na branch ops/
           ciclo: EXECUTANDO → COMMITADO
           ↓
Etapa 7b   Claude lê diff do commit
           Valida: diff ↔ instrução autorizada?
           → correspondência: ciclo → VERIFICADO
           → divergência: retorna ao usuário com diff
           ↓
Etapa 8a   Usuário autoriza merge ops/ → main [GATE 2]
           ↓
Etapa 9    Snapshot atualizado com commit_hash e estado: CONCLUÍDO
```

Dois gates humanos com evidência verificável em cada um.

---

## A MUDANÇA DE CATEGORIA: CONFIÁVEL → VERIFICÁVEL

Antes da camada Git (v3.0):
- Os ciclos são confiáveis — baseados em snapshots homologados
- Não há como verificar objetivamente se a execução correspondeu à decisão
- Rollback depende de reconstrução manual

Com a camada Git (v3.5):
- Os ciclos são verificáveis — snapshot + commit = evidência completa
- A correspondência instrução ↔ execução é auditável pelo diff
- Rollback é rastreável e reversível

A evidência operacional verificável é o que transforma
o sistema de confiável em auditável.

---

## COMPATIBILIDADE COM CICLOS V3.0 PRÉ-GIT

Snapshots criados antes de v3.5 não possuem `commit_hash`.

```yaml
# Snapshot v3.0 — sem commit_hash
audit-logs-001:
  commit_hash: null   ← ausente ou null
```

Comportamento do sistema com ciclos pré-Git:

| Operação | Com commit_hash | Sem commit_hash (pré-Git) |
|---|---|---|
| Recuperação contextual | snapshot + evidência Git | snapshot apenas |
| Replay operacional | snapshot + diff do commit | snapshot apenas (sem diff) |
| Rollback | git revert + snapshot | rollback manual + snapshot |
| Detecção de drift | objetiva (diff) | por análise (subjetiva) |

Ciclos pré-Git são válidos. Têm contexto institucional completo.
Não têm evidência verificável — limitação documentada, não erro.

O sistema não força migração retroativa de snapshots v3.0.
A evidência verificável começa a partir do primeiro ciclo v3.5.

---

## LIMITES

- Git não substitui snapshots — são complementares
- Git não substitui governança humana — gates permanecem obrigatórios
- Git não aprende nem decide — persiste evidência, não raciocínio
- Commits não são auto-interpretativos sem o snapshot correspondente
- A verificabilidade começa no baseline de Sprint 0 — não antes
