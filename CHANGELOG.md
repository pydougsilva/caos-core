# CHANGELOG — C.A.O.S

---

## v6.0-pre — Sprint Documental: Indexação Relacional Institucional (2026-05-28)

**Marco:** formalização institucional da proposta de indexação relacional — transição de
infraestrutura cognitiva operacional para infraestrutura cognitiva relacional.

**Origem:** auditoria cognitiva estrutural completa (2026-05-28) revelou três limites
estruturais do modelo v5.0: dependências invisíveis, órfãos indetectáveis, lineage
como reconstrução (não como recuperação).

Adicionado em /k/sistema:
- k-sys-lineage-arquitetural (v1.0): cadeia causal completa da evolução v1.0→v6.0
  com problema_motivador, resposta_arquitetural e problema_revelado por versão
- k-sys-proposta-indexacao-relacional (v1.0): proposta arquitetural formal da v6.0
  contendo os 4 planos (estrutural, frontmatter, grafo, SBERT), roadmap v5.x→v7.0,
  riscos com mitigações, critérios de readiness para implementação
- k-sys-principios-fundamentais (v1.0): 15 invariantes arquiteturais imutáveis
  que toda versão futura deve respeitar — base constitucional do sistema

Atualizado:
- rag/index.md v5.2: 3 novos módulos indexados, 3 novas entradas em TAREFA→MÓDULOS,
  linha evolutiva v6.0 declarada

Esta sprint é exclusivamente documental.
Nenhuma estrutura de grafo, banco ou embedding foi criada.
Nenhum módulo /r foi modificado.
A implementação da v6.0 é sprint separada com gate de readiness definido.

Proposta de estrutura futura declarada (não implementada):
- rag/graph/ (a ser criado na sprint de implementação v6.0)
- rag/graph/relations.yaml, lineage.md, drift-map.yaml, orphans.yaml
- rag/graph/embeddings/ (requer SBERT — v5.5)
- Novos módulos v6.0: k-sys-grafo, r-grafo-institucional, k-sys-sbert

---

## v5.0 — Continuidade Cognitiva e Runtime Session-Bound (2026-05-21)

**Marco:** formalização do runtime session-bound + contingência de executor + descontaminação institucional + formato de saída do orquestrador.

Adicionado:
- r-continuidade-cognitiva (4 contratos, níveis, protocolo de promoção de módulo)
- r-executor-contingencia (4 modos, taxonomia 3D, pré-requisitos Nível 0, T-GEM.0)
- r-telemetria-cognitiva v1.2 (métricas de continuidade, RISCOS ARQUITETURAIS, modo_operacao)

Atualizado:
- AGENTS.md v5.0: PRINCÍPIO DO ISOLAMENTO + anti-fine-tuning + CONTINUIDADE MÍNIMA
- rag/index.md v5.0: novos módulos + entradas tarefa→módulo + marco v5.0
- r-orquestracao-caos v1.4: FORMATO DE SAÍDA (5 seções + SEÇÃO 0 + ANEXO A + bloco único)

Descontaminação institucional executada:
- k-sys-governanca-git: exemplos fornadas/pedidos substituídos por placeholders genéricos
- r-matching-conceito: exemplos com domínios afetoeforma substituídos por exemplos neutros
- r-concurrency-guard: domínios específicos substituídos por [dominio_a]/[dominio_b]
- r-hotfix-padrao: App.jsx removido como referência hardcoded
- r-staleness-detection: k-proj-identidade e App.jsx substituídos por referências genéricas
- r-telemetria-cognitiva: exemplo de checkout afetoeforma substituído
- r-auto-recuperacao-contextual: aliases específicos substituídos por placeholders
- k-sys-handoff-institucional: referência a k-proj-identidade generalizada

Axioma formalizado:
  agente sem artefatos = agente genérico
  agente + artefatos C.A.O.S = runtime institucional temporário
  Fundamentação: Lewis et al. (2020)

---

## v4.0 — Replicabilidade Institucional (2026-05-12)

**Marco:** extração do core como infraestrutura reutilizável independente do Afeto em Forma.

Adicionado:
- k-fe-app-estrutura (mapeamento estrutural de frontends monolíticos)
- r-anti-burocracia (limites operacionais contra hipercomplexidade)
- k-bootstrap-caos (guia de adoção para 3 perfis de projeto)
- k-sys-nucleo-minimo (definição dos 7 artefatos essenciais)
- MANUAL-OPERACIONAL (manual para humanos e agentes)
- DISTRIBUICAO-GITHUB (política de distribuição)
- Templates (AGENTS, snapshot, telemetria)
- Extração do core como repositório independente

Separação formalizada:
- caos-core: núcleo universal reutilizável
- afeto-em-forma: primeiro projeto operacional homologado

---

## v3.9 — Hardening Institucional (2026-05-12)

**Marco:** auditoria de continuidade cognitiva por novo agente + melhorias.

Adicionado:
- r-staleness-detection
- r-module-pruning
- r-concurrency-guard
- r-telemetria-cognitiva
- k-sys-handoff-institucional

Corrigido:
- Módulos fantasma em index.md
- Duplicata hotfix-padrao.md arquivada

---

## v3.5 — Persistência Operacional Verificável (2026-05-11)

**Marco:** Git como ledger institucional de execuções.

Adicionado:
- r-git-operacional
- r-commit-governance
- r-rollback-contextual
- r-replay-operacional
- k-sys-persistencia-operacional
- k-sys-governanca-git

Ativado condicionalmente:
- r-estados-ciclo v2.0 (COMMITADO, VERIFICADO, DIVERGENTE)
- r-handoff-codex v2.0 (campos commit_type, branch_sugerido, commit_hash)

---

## v3.0 — Continuidade Operacional entre Agentes (2026-05-11)

**Marco:** handoff formal Claude→Codex + estados + matching estruturado.

Adicionado:
- k-sys-registry-dominios
- r-estados-ciclo
- k-sys-handoff-format
- r-handoff-codex
- r-matching-conceito
- r-snapshots-incrementais

---

## v2.2 — Auto-recuperação Contextual (anterior a 2026-05-11)

**Marco:** detecção automática de domínios e disparo de recovery.

Adicionado:
- r-auto-recuperacao-contextual
- r-recuperacao-contextual

---

## v2.1 — Persistência Operacional (anterior a 2026-05-11)

**Marco:** snapshots como memória institucional.

Adicionado:
- Estrutura de snapshots
- Protocolo de ciclo operacional

---

## v1.0–v2.0 — Framework Operacional Inicial

**Marco:** validação do ciclo no Afeto em Forma + governança via RAG.

- RAG modular (/r e /k)
- Primeiro ciclo operacional real (hotfix RLS audit_logs)
- Descoberta: necessidade de persistência além da sessão
