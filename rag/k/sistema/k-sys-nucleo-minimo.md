# k-sys-nucleo-minimo
versao: 1.0

## OBJETIVO

Definir o conjunto mínimo de artefatos necessários para que o C.A.O.S
funcione como infraestrutura cognitiva operacional.

Responde à pergunta:
"qual é o menor C.A.O.S que ainda é um C.A.O.S?"

---

## PRINCÍPIO

Complexidade opcional sobre núcleo simples.

O núcleo deve ser pequeno o suficiente para ser adotado em uma tarde
e poderoso o suficiente para resolver a maioria dos problemas de continuidade.

---

## NÚCLEO OBRIGATÓRIO — 7 ARTEFATOS

Estes 7 artefatos são o mínimo para preservar continuidade cognitiva:

### 1. AGENTS.md (autoridade máxima)

O que define: identidade do sistema, fluxo operacional, governança.
Sem ele: nenhum agente sabe como operar.

**Mínimo viável:** nome, stack, fase atual, fluxo de 4 etapas, prioridade de autoridade.

---

### 2. rag/index.md (mapa de módulos)

O que define: quais módulos existem e quando usar cada um.
Sem ele: agentes carregam módulos aleatoriamente ou ignoram o RAG.

**Mínimo viável:** TAREFA → MÓDULOS com 5-10 linhas, MÓDULOS EXISTENTES listados.

---

### 3. rag/k/projeto/k-proj-identidade.md (identidade do produto)

O que define: o que o projeto é, qual é a fase atual, qual é o stack.
Sem ele: agente novo não consegue entender o domínio sem ler código.

**Mínimo viável:** nome, tipo, stack, fase atual, próximas entregas.

---

### 4. rag/r/r-rls-padrao.md (segurança de dados)

O que define: regras de isolamento multi-tenant via RLS.
Sem ele: risco de políticas inseguras em projetos com dados de múltiplos usuários.

**Incluso no núcleo porque:** o custo de um erro de RLS é muito alto para omitir.

---

### 5. rag/r/r-sql-idiomatico.md (qualidade de migrations)

O que define: regras para scripts SQL idempotentes e seguros.
Sem ele: migrations quebram em re-execução ou causam downtime.

**Incluso no núcleo porque:** toda operação de banco passa por aqui.

---

### 6. rag/k/sistema/k-sys-registry-dominios.md (mapa de domínios)

O que define: quais domínios operacionais existem, seus aliases e módulos padrão.
Sem ele: matching de domínio é puramente heurístico, sem estrutura.

**Mínimo viável:** 3-5 domínios críticos com aliases básicos.

---

### 7. rag/r/r-recuperacao-contextual.md (memória institucional)

O que define: como snapshots são criados, estruturados e recuperados.
Sem ele: ciclos não produzem memória — o sistema não aprende com a história.

**Mínimo viável:** estrutura de snapshot + 1 exemplo real (mesmo que simples).

---

## RESUMO DO NÚCLEO

```
rag/
├── index.md                          ← mapa de módulos
├── r/
│   ├── r-rls-padrao.md               ← segurança
│   ├── r-sql-idiomatico.md           ← qualidade SQL
│   └── r-recuperacao-contextual.md   ← memória
└── k/
    ├── projeto/
    │   └── k-proj-identidade.md      ← identidade do produto
    └── sistema/
        └── k-sys-registry-dominios.md ← mapa de domínios

AGENTS.md                              ← autoridade máxima (raiz)
```

**7 arquivos. Funcional. Completo para a maioria dos projetos simples.**

---

## CAPACIDADES DO NÚCLEO

Com apenas o núcleo, o C.A.O.S pode:

- Reconstruir o contexto do projeto para um novo agente
- Executar ciclos operacionais de SQL e RLS com segurança
- Criar snapshots de decisões e recuperá-los em sessões futuras
- Identificar domínios conhecidos por aliases
- Manter trilha de decisões arquiteturais

---

## CAPACIDADES QUE REQUEREM MÓDULOS ADICIONAIS

| Capacidade | Módulo necessário |
|---|---|
| Matching estruturado de domínios | r-matching-conceito |
| Handoff formal Claude→Codex | r-handoff-codex + k-sys-handoff-format |
| Estados formais do ciclo | r-estados-ciclo |
| Snapshots incrementais | r-snapshots-incrementais |
| Persistência verificável (Git) | r-git-operacional + r-commit-governance |
| Rollback estruturado | r-rollback-contextual |
| Replay de ciclos | r-replay-operacional |
| Memória semântica | SBERT + r-semantic-retrieval (v4.0) |
| Detecção de staleness | r-staleness-detection |
| Concorrência de ciclos | r-concurrency-guard |
| Telemetria de sessão | r-telemetria-cognitiva |
| Hotfixes de frontend | r-hotfix-padrao |
| Bootstrap de novos projetos | k-bootstrap-caos |

---

## SEQUÊNCIA DE ADOÇÃO RECOMENDADA

```
Nível 0 — Núcleo (7 arquivos)
  → continuidade básica + segurança

Nível 1 — Continuidade estruturada (adicionar em projetos ativos)
  → r-matching-conceito
  → r-snapshots-incrementais
  → r-estados-ciclo
  → r-hotfix-padrao (se tiver frontend)

Nível 2 — Governança operacional (adicionar quando necessário)
  → r-handoff-codex + k-sys-handoff-format
  → r-staleness-detection
  → r-concurrency-guard
  → r-telemetria-cognitiva

Nível 3 — Persistência verificável (adicionar com Python runner)
  → r-git-operacional
  → r-commit-governance
  → r-rollback-contextual
  → r-replay-operacional

Nível 4 — Memória semântica (adicionar com corpus suficiente)
  → SBERT + módulos v4.0
```

---

## O QUE NÃO É NÚCLEO

Módulos que NÃO devem ser incluídos no núcleo de projetos novos:
- r-git-operacional (requer configuração de Git commit governance)
- r-estados-ciclo (adiciona overhead antes de ser necessário)
- r-handoff-codex (requer executor separado real)
- Qualquer módulo de telemetria (antes de ter ciclos reais para rastrear)

Estes módulos têm valor real — mas adicionados prematuramente criam sobrecarga
sem benefício proporcional.
