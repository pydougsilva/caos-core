# RAG Index — C.A.O.S Core
versao: 5.5

---

# REGRA PRINCIPAL

Carregar apenas módulos necessários.
Nunca carregar o RAG inteiro.
Máximo recomendado: 3 módulos por tarefa.

---

# ORDEM DE CARREGAMENTO

```
Etapa 0   → Detectar gatilho (r-auto-recuperacao-contextual)
Etapa 0a  → Matching por conceito (r-matching-conceito + k-sys-registry-dominios)
Etapa 0b  → Recuperar snapshot (r-recuperacao-contextual)
Etapa 1   → /r  (regras — antes de /k)
Etapa 2   → /k  (conhecimento)
```

---

# FALLBACK

| Módulo ausente | Comportamento |
|---|---|
| r-matching-conceito | matching heurístico (r-auto-recuperacao-contextual) |
| r-handoff-codex | instrução textual informal |
| r-handoff-executor | instrução textual informal (complementar a r-handoff-codex) |
| r-estados-ciclo | sem rastreamento formal |
| r-git-operacional | sem verificação de diff |

---

# TAREFA → MÓDULOS

| Tarefa | Arquivo 1 | Arquivo 2 | Arquivo 3 |
|---|---|---|---|
| Script SQL | r/r-sql-idiomatico | k/banco/[tabelas] | r/r-rls-padrao |
| Policy RLS | r/r-rls-padrao | k/banco/[funcoes] | — |
| Hotfix frontend | r/r-hotfix-padrao | k/frontend/[componente] | — |
| Domínio com histórico | r/r-recuperacao-contextual | — | — |
| Handoff estruturado | r/r-handoff-codex | k/sistema/k-sys-handoff-format | r/r-estados-ciclo |
| Handoff estruturado (genérico) | r/r-handoff-executor | k/sistema/k-sys-handoff-format | r/r-estados-ciclo |
| Restauracao de orquestrador | r/r-restauracao-orquestrador | r/r-estados-ciclo | r/r-telemetria-cognitiva |
| Governanca de repositorios | k/sistema/k-sys-governanca-repositorios | — | — |
| Retomada de ciclo | r/r-estados-ciclo | r/r-recuperacao-contextual | — |
| Matching de domínio | r/r-matching-conceito | k/sistema/k-sys-registry-dominios | — |
| Snapshot incremental | r/r-snapshots-incrementais | r/r-recuperacao-contextual | — |
| Commit institucional | r/r-commit-governance | k/sistema/k-sys-governanca-git | r/r-git-operacional |
| Rollback de ciclo | r/r-rollback-contextual | r/r-estados-ciclo | — |
| Replay de ciclo | r/r-replay-operacional | r/r-recuperacao-contextual | — |
| Auditoria Git | r/r-git-operacional | k/sistema/k-sys-persistencia-operacional | — |
| Entrada novo agente | k/sistema/k-sys-handoff-institucional | r/r-staleness-detection | r/r-telemetria-cognitiva |
| Bootstrap projeto | k/projeto/k-bootstrap-caos | k/sistema/k-sys-nucleo-minimo | — |
| Anti-burocracia | r/r-anti-burocracia | r/r-module-pruning | — |
| Continuidade cognitiva | r/r-continuidade-cognitiva | r/r-telemetria-cognitiva | r/r-staleness-detection |
| Contingência de executor | r/r-executor-contingencia | r/r-handoff-executor | r/r-handoff-codex |
| Lineage arquitetural | k/sistema/k-sys-lineage-arquitetural | — | — |
| Proposta arquitetural v6.0 | k/sistema/k-sys-proposta-indexacao-relacional | k/sistema/k-sys-principios-fundamentais | — |
| Princípios fundamentais | k/sistema/k-sys-principios-fundamentais | — | — |
| Auditoria arquitetural | rag/docs/validacoes/A1.0-indiferenca-agente-auditoria | — | — |
| Adapter Layer / Bootstrap Verificável | k/sistema/k-sys-adapter-layer | k/sistema/k-sys-principios-fundamentais | — |

---

# MÓDULOS EXISTENTES

## /r — Regras Operacionais

| Arquivo | Versão | Finalidade |
|---|---|---|
| r-sql-idiomatico | 1.0 | regras para migrations SQL idempotentes |
| r-rls-padrao | 1.0 | isolamento multi-tenant via RLS |
| r-hotfix-padrao | 1.0 | patches cirúrgicos sem reescrever arquivo completo |
| r-atualizacao-rag | v1.1 | quando e como evoluir o RAG |
| r-recuperacao-contextual | 1.1 | snapshots históricos entre sessões |
| r-auto-recuperacao-contextual | 1.0 | detecção automática de domínios |
| r-estados-ciclo | 2.0 | estados formais — COMMITADO/VERIFICADO/DIVERGENTE |
| r-handoff-codex | 2.0 | protocolo handoff Claude→Codex |
| r-matching-conceito | 1.0 | matching por score ponderado de aliases |
| r-snapshots-incrementais | 1.0 | cadeia de deltas sobre snapshot base |
| r-git-operacional | 1.0 | leitura Git + detecção de drift |
| r-commit-governance | 1.0 | commits institucionais |
| r-rollback-contextual | 1.0 | rollback técnico + institucional |
| r-replay-operacional | 1.0 | replay assistido de ciclos |
| r-staleness-detection | 1.0 | detecção de artefatos desatualizados |
| r-module-pruning | 1.0 | arquivamento e simplificação de módulos |
| r-concurrency-guard | 1.0 | prevenção de colisão entre ciclos |
| r-telemetria-cognitiva | 1.2 | registro mínimo de sessão + métricas de continuidade |
| r-anti-burocracia | 1.0 | limites contra hipercomplexidade |
| r-continuidade-cognitiva | 1.0 | continuidade cognitiva entre sessões, agentes e projetos |
| r-executor-contingencia | 1.0 | contingência de executor + taxonomia + modos operacionais |
| r-orquestracao-caos | v1.4 | comportamento do orquestrador Claude + formato de saída |
| r-restauracao-orquestrador | 1.0 | restauração do papel agente_orquestrador sem dependência nominal |
| r-handoff-executor | 2.0 | protocolo handoff com papéis genéricos (complementar a r-handoff-codex) |

---

## /k/sistema — Conhecimento do Sistema

| Arquivo | Versão | Finalidade |
|---|---|---|
| k-sys-registry-dominios | 1.0 | catálogo de domínios (PREENCHER por projeto) |
| k-sys-handoff-format | 1.0 | estrutura dos documentos handoff e retorno |
| k-sys-persistencia-operacional | 1.0 | camada Git no C.A.O.S |
| k-sys-governanca-git | 1.0 | branches, commits e convenções |
| k-sys-handoff-institucional | 1.0 | protocolo de entrada para novo agente |
| k-sys-nucleo-minimo | 1.0 | núcleo mínimo replicável |
| k-sys-governanca-repositorios | 1.0 | separação institucional produto ↔ caos-core |
| k-sys-lineage-arquitetural | 1.0 | cadeia causal completa da evolução do C.A.O.S (arqueologia institucional) |
| k-sys-proposta-indexacao-relacional | 1.0 | proposta arquitetural v6.0 — indexação relacional institucional |
| k-sys-principios-fundamentais | 1.0 | invariantes arquiteturais — restrições imutáveis de toda versão futura |
| k-sys-adapter-layer | 1.0 | Adapter Layer — Bootstrap Institucional Verificável, context_receipt, adapters por ferramenta |

---

## /k/projeto — Metodologia

| Arquivo | Versão | Finalidade |
|---|---|---|
| k-bootstrap-caos | 1.0 | guia de inicialização (3 perfis de projeto) |
| k-proj-caos-metodo | 1.1 | metodologia C.A.O.S — componentes, fluxo, economia cognitiva (papéis nominais) |

---

## /docs — Documentação Institucional

| Arquivo | Versão | Finalidade |
|---|---|---|
| MANUAL-OPERACIONAL | 1.0 | manual para humanos e novos agentes — onboarding, fluxo, troubleshooting |
| DISTRIBUICAO-GITHUB | 1.0 | política de distribuição via Git/GitHub, branches, tags, rollback |
| PREPARACAO-CONTAINERIZACAO | 1.0 | contratos para containerização futura — não implementado |

---

# EVOLUÇÃO ARQUITETURAL

| Versão | Marco |
|---|---|
| v2.x | snapshots + auto-recuperação |
| v3.0 | continuidade entre agentes (registry, handoff, estados) |
| v3.5 | persistência verificável (Git) |
| v3.9 | hardening (staleness, pruning, concurrency, telemetria) |
| v4.0 | replicabilidade institucional (bootstrap, núcleo mínimo, manual) |
| v5.0 | continuidade cognitiva + runtime session-bound + contingência de executor |
| v5.3 | Bootstrap Institucional Verificável — CLAUDE.md, .caos/adapters/, context_receipt |
| v6.0 | indexação relacional institucional + grafo + SBERT offline — proposta formalizada |

---

# PRINCÍPIOS

Cada módulo:
- responde uma única pergunta
- possui escopo isolado
- evita duplicação
- reduz tokens

---

# OBJETIVO FINAL

Preservar continuidade cognitiva entre sessões efêmeras de IA.
Transformar experiência operacional em memória institucional reutilizável.
