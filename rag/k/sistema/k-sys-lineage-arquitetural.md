# k-sys-lineage-arquitetural
versao: 1.0
data: 2026-05-28
tipo: conhecimento_sistêmico
status: canônico

## OBJETIVO

Registrar a cadeia causal completa da evolução do C.A.O.S:
não apenas o que foi construído em cada versão,
mas por que foi construído e qual problema cada versão revelou.

Este documento é memória arqueológica institucional.

Não é changelog operacional — é lineage cognitivo.

O CHANGELOG.md registra o que mudou.
Este documento registra por que mudou e o que cada mudança expôs.

---

## PRINCÍPIO DE LEITURA

Cada versão do C.A.O.S resolve um problema e revela o próximo.

A coluna "problema_revelado" é tão importante quanto "problema_motivador":
ela é o germe da versão seguinte.

Ler em ordem cronológica revela a lógica interna da evolução.
Ler em sentido inverso revela o problema que ainda não foi resolvido.

---

## LINEAGE

---

### v1.0 — Framework Operacional Inicial
**Período:** anterior a 2026-05-11 (primeiro ciclo real: 2026-05-09)

**Problema motivador:**
LLMs falham não por falta de inteligência, mas por falta de contexto organizado.
Cada sessão começa do zero. Regras não persistem. Decisões se perdem entre sessões.
O modelo é capaz — mas sem estrutura de memória, recomeça sempre.

**Resposta arquitetural:**
RAG modular dividido em /r (regras) e /k (conhecimento).
Ciclo operacional estruturado: proposta → validação → execução → registro.
AGENTS.md como autoridade de identidade do sistema.

**Primitivo central:** separação regras/conhecimento como infraestrutura de memória modular.

**Evidência operacional:** primeiro ciclo real — hotfix de RLS em public.audit_logs (2026-05-09).

**Insight central:**
"Documentação não é arquivo morto. É memória operacional ativa."

**Problema revelado:**
Decisões de uma sessão não sobrevivem para a próxima.
O ciclo operacional funciona, mas o que aconteceu não persiste de forma recuperável.
O contexto é reconstruído do zero a cada sessão — custo O(n), deveria ser O(1).

---

### v2.1 — Persistência Operacional (Snapshots)
**Período:** anterior a 2026-05-11

**Problema motivador:**
Cada sessão repetia análise de estado que a sessão anterior já havia executado.
Decisões arquiteturais eram relembradas ao invés de recuperadas.
O custo cognitivo por sessão era proporcional ao tamanho da história — insustentável.

**Resposta arquitetural:**
Snapshots como memória institucional persistente.
Protocolo de ciclo com snapshot obrigatório ao final: proposta → validação → execução → snapshot.

**Primitivo central:** snapshot como exteriorização de estado decisório.

**Insight central:**
"Memória institucional não é o que foi discutido.
É o que foi decidido, executado e homologado."

**Problema revelado:**
Snapshots existem, mas precisam ser encontrados.
Qual snapshot é relevante para a sessão atual?
Matching por nome de arquivo é inadequado: a sessão não sabe o nome do arquivo correto.
O sistema acumula memória mas não sabe acessá-la autonomamente.

---

### v2.2 — Auto-recuperação Contextual
**Período:** anterior a 2026-05-11

**Problema motivador:**
Novos agentes não sabiam qual snapshot carregar.
O matching era manual: dependia de o usuário lembrar os nomes de domínio exatos.
A memória existia — mas era inacessível sem intervenção humana.

**Resposta arquitetural:**
r-auto-recuperacao-contextual: detecção automática de domínio por gatilhos semânticos.
r-recuperacao-contextual: protocolo de recuperação de snapshot relevante.
Hierarquia de recuperação: último sucesso > riscos ativos > última falha > base.

**Primitivo central:** auto-detecção de domínio como pré-condição de recuperação.

**Insight central:**
"O sistema deve recuperar seu próprio contexto.
Não deve depender de o usuário lembrar onde as coisas estão."

**Problema revelado:**
Matching por substring é frágil: o mesmo domínio tem múltiplos nomes possíveis.
Dois agentes executando no mesmo domínio simultaneamente podem colidir sem saber.
O handoff entre agentes (Claude → Codex) é instrução textual informal:
sem campos obrigatórios, sem verificação de completude, sem rastreamento de estado.

---

### v3.0 — Continuidade Operacional entre Agentes
**Período:** 2026-05-11

**Problema motivador:**
Claude orquestra. Codex executa.
Mas o protocolo de transferência entre eles era instrução textual informal —
sem campos obrigatórios, sem verificação, sem estado rastreável.
O ciclo podia falhar silenciosamente entre a orquestração e a execução.

**Resposta arquitetural:**
Cinco primitivos implementados simultaneamente:
1. k-sys-registry-dominios — catálogo canônico de domínios com aliases e pesos
2. r-estados-ciclo — estados formais: PENDENTE, EM_EXECUÇÃO, COMMITADO, VERIFICADO, DIVERGENTE
3. k-sys-handoff-format + r-handoff-codex — protocolo estruturado de transferência
4. r-matching-conceito — score ponderado por aliases (substitui substring matching)
5. r-snapshots-incrementais — cadeia de deltas sobre snapshot base (máx 5 por base)

**Primitivo central:** protocolo formal de handoff com estado rastreável.

**Insight central:**
"Um agente que não sabe em que estado o ciclo está
não pode decidir se deve continuar, parar ou escalar."

**Problema revelado:**
O handoff é estruturado, mas a execução não deixa rastro verificável externo.
Não há evidência de que o que foi executado corresponde ao que foi autorizado.
Git existe como infraestrutura, mas não é usado como ledger institucional.
"Foi executado" depende de confiar no retorno do executor — sem verificação independente.

---

### v3.5 — Persistência Operacional Verificável
**Período:** 2026-05-11

**Problema motivador:**
Ciclos eram concluídos sem evidência verificável externa.
O diff do que realmente mudou não era capturado como parte do protocolo.
A distância entre "o que foi autorizado" e "o que foi executado" era invisível.

**Resposta arquitetural:**
Git como ledger institucional de execuções.
Seis módulos: r-git-operacional, r-commit-governance, r-rollback-contextual,
r-replay-operacional, k-sys-persistencia-operacional, k-sys-governanca-git.
Fluxo estendido: branch ops/ → commit institucional → verificação de diff → merge com gate humano.
Estados v2.0 ativados condicionalmente com r-git-operacional: COMMITADO, VERIFICADO, DIVERGENTE.

**Primitivo central:** commit institucional como evidência verificável de execução autorizada.

**Insight central:**
"Execução sem evidência não é execução institucional. É operação em modo fé."

**Problema revelado:**
O sistema cresceu para 20+ módulos.
Módulos tornaram-se desatualizados sem que ninguém detectasse.
Ciclos em execução simultânea podem colidir sem guard formal.
Telemetria de sessão não existe: continuidade entre agentes diferentes é frágil.
Um novo agente chegando não tem protocolo de entrada — começa sem contexto estruturado.
O sistema não audita sua própria saúde.

---

### v3.9 — Hardening Institucional
**Período:** 2026-05-12

**Problema motivador:**
Auditoria conduzida por novo agente revelou múltiplas lacunas simultâneas:
módulos desatualizados sem detecção, ausência de telemetria de sessão,
sem protocolo formal de entrada para novo executor,
sem proteção contra colisão entre ciclos concorrentes,
duplicata de módulo ativa na pasta principal.

**Resposta arquitetural:**
r-staleness-detection (detecção de artefatos desatualizados),
r-module-pruning (critérios de arquivamento e simplificação),
r-concurrency-guard (prevenção de colisão entre ciclos),
r-telemetria-cognitiva (registro mínimo de sessão),
k-sys-handoff-institucional (protocolo de entrada para novo agente executor).
Duplicata hotfix-padrao.md arquivada em rag/arquivo/r/.

**Primitivo central:** auditabilidade do próprio sistema como capacidade nativa.

**Insight central:**
"Um sistema que não audita seus próprios módulos acumula entropia silenciosa.
A complexidade invisível é mais perigosa que a complexidade visível."

**Problema revelado:**
O C.A.O.S está acoplado ao Afeto em Forma como projeto de origem.
Módulos genéricos contêm exemplos e aliases específicos do produto.
O sistema não pode ser adotado por outros projetos sem carregar história de produto.
A infraestrutura cognitiva precisa ser separável do projeto que a criou.

---

### v4.0 — Replicabilidade Institucional
**Período:** 2026-05-12

**Problema motivador:**
O C.A.O.S só existia dentro do Afeto em Forma.
Módulos genéricos referenciavam entidades do produto (fornadas, pedidos, App.jsx).
Adoção por outros projetos carregava bagagem histórica de produto.
A infraestrutura e o produto eram inseparáveis.

**Resposta arquitetural:**
Extração do core como repositório independente (caos-core).
k-bootstrap-caos (3 perfis de adoção: greenfield, legado com Git, legado sem governança).
k-sys-nucleo-minimo (definição formal dos 7 artefatos essenciais do núcleo).
Templates institucionais: AGENTS.md, snapshot, telemetria.
Descontaminação parcial de exemplos produto-específicos.
Separação formal: caos-core (infraestrutura reutilizável) ↔ afeto-em-forma (projeto de origem).

**Primitivo central:** separação produto/infraestrutura como pré-condição de replicabilidade.

**Insight central:**
"Infraestrutura cognitiva só é infraestrutura
quando pode ser adotada por qualquer projeto
sem carregar a história de quem a criou."

**Problema revelado:**
Dependências nominais residuais persistem em contratos ativos:
módulos referenciam CODEX/CLAUDE como identidades fixas em vez de papéis institucionais.
Se o executor muda de ferramenta, contratos estruturais quebram.
O modo degradado (mesmo agente acumulando papéis de orquestrador e executor)
existe na prática mas não está documentado como protocolo formal.
O princípio "papéis sobre identidades" não está formalizado como invariante.

---

### v5.0 — Runtime Session-Bound e Papéis sobre Identidades
**Período:** 2026-05-21 (hardening nominal: 2026-05-25)

**Problema motivador:**
Hardening nominal revelou: 25+ módulos ainda continham referências a CODEX/CLAUDE
como identidades em contratos ativos — não apenas em evidências históricas.
Se o executor mudasse de ferramenta, o protocolo deixava de funcionar.
O modo degradado (único agente acumulando papéis) operava sem contrato formal.
Separação repositórios declarada mas não governada por artefato.

**Resposta arquitetural:**
Formalização do runtime session-bound como axioma:
"agente sem artefatos = agente genérico; agente + artefatos = runtime institucional temporário."
Descontaminação nominal de 25+ módulos (identidades → papéis institucionais).
r-continuidade-cognitiva (4 contratos de continuidade, níveis, protocolo de promoção).
r-executor-contingencia (4 modos operacionais, taxonomia 3D, T-GEM.0).
r-restauracao-orquestrador (restauração sem dependência nominal).
r-handoff-executor como substituto agnóstico de r-handoff-codex.
k-sys-governanca-repositorios (separação institucional produto ↔ caos-core governada por artefato).
AGENTE-EXECUTOR-BOOTSTRAP.md (rename de CODEX-BOOTSTRAP.md).
Fundamentação formal: Lewis et al. (2020) — generalização de RAG para governança institucional.

**Primitivo central:** papéis institucionais como contrato; ferramentas como implementação contingente.

**Insight central:**
"O protocolo pertence aos artefatos, não às ferramentas.
Papéis são estrutura institucional permanente.
Ferramentas são implementações possíveis — transitórias por natureza."

**Problema revelado:**
O sistema sabe o que existe (módulos, domínios, snapshots).
Não sabe como esses elementos se relacionam entre si.
Dependências entre módulos são implícitas — detectáveis apenas por leitura humana completa.
Módulos órfãos são invisíveis até auditoria manual.
Lineage arquitetural requer reconstrução a partir de 13+ arquivos de telemetria.
O índice linear (index.md, TAREFA→MÓDULOS) não captura a topologia do conhecimento institucional.
Drift entre instâncias de módulos (caos-core ↔ produto) é descoberto, não rastreado.
O sistema possui memória de primeira ordem — sabe o que existe.
Precisa de memória de segunda ordem — saber como e por que as coisas existem.

---

### v6.0 — Indexação Relacional Institucional
**Período:** proposta formalizada em 2026-05-28
**Status:** PROPOSTA — aprovada para formalização institucional

**Problema motivador:**
Auditoria cognitiva estrutural completa (2026-05-28) revelou:
- 5 dependências invisíveis confirmadas (não declaradas em nenhum artefato)
- 8 módulos órfãos ou parcialmente integrados identificados
- 1 lineage arquitetural que exige leitura de 13 telemetrias para ser reconstruído
- Drift entre instâncias detectado manualmente, sem artefato dedicado
- Backlog implícito não consolidado em nenhum artefato único
- Snapshot base de v2.1 vazio (vestigial)

A informação já existe no sistema — mas está fragmentada.
O sistema não consegue navegar sua própria topologia institucional sem intervenção humana.

**Resposta arquitetural proposta:**
Camada de grafo institucional (rag/graph/) como síntese dos artefatos existentes.
Frontmatter relacional em módulos (depends_on, activates, supersedes, related_to).
SBERT local como serviço de indexação offline — nunca no runtime operacional.
Registry v3.0 com campos relacionais e contador formal de ciclos.
Snapshots com cabeçalho relacional (supersede, ciclos_geradores, modulos_afetados).
k-sys-lineage-arquitetural como artefato de arqueologia institucional (este documento).
drift-map.yaml como evidência de divergência de primeira classe.

Ver: k-sys-proposta-indexacao-relacional para arquitetura detalhada.

**Primitivo central (proposto):** relações entre artefatos como dado de primeira classe.

**Insight central (proposto):**
"Um sistema que sabe apenas o que existe opera com memória de primeira ordem.
Um sistema que sabe como as coisas se relacionam, por que existem e como evoluíram
opera com consciência institucional."

**Problema previsto para v7.0:**
Escala do grafo pode superar capacidade de representação em YAML estruturado.
Possível necessidade de backend de grafo leve (SQLite, DuckDB)
para corpus com mais de 50 domínios ou 100+ módulos.
Automação parcial de promoção/arquivamento de módulos com base em grafo e SBERT.

---

## CADEIA CAUSAL CONDENSADA

```
v1.0  Sessões não preservam decisões
  ↓   resolve: snapshots como memória
v2.1  Snapshots existem mas não são encontrados autonomamente
  ↓   resolve: auto-recuperação por detecção de gatilho
v2.2  Matching frágil + handoff informal entre agentes
  ↓   resolve: registry + protocolo estruturado + estados formais
v3.0  Execução sem evidência verificável
  ↓   resolve: Git como ledger + commit institucional + verificação de diff
v3.5  Entropia silenciosa — módulos stale, sem telemetria, sem guard de concorrência
  ↓   resolve: staleness detection + telemetria + concurrency guard
v3.9  Acoplamento ao produto de origem — sistema não é replicável
  ↓   resolve: extração do core + bootstrap + templates
v4.0  Dependências nominais em contratos — fragilidade quando ferramenta muda
  ↓   resolve: papéis sobre identidades + runtime session-bound + contingência
v5.0  Conhecimento fragmentado sem topologia — dependências invisíveis, órfãos indetectáveis
  ↓   resolve (proposto): grafo institucional + frontmatter relacional + SBERT offline
v6.0  [problema previsto] Escala do grafo além da capacidade YAML
  ↓   resolve (hipotético): backend de grafo leve para corpus > 50 domínios
v7.0
```

---

## PADRÃO EVOLUTIVO OBSERVADO

Cada versão do C.A.O.S seguiu um padrão consistente:

1. **Um ciclo real revela uma lacuna** — a evolução é sempre empírica, nunca puramente teórica
2. **A lacuna é local mas a solução é sistêmica** — um problema de ciclo vira protocolo de sistema
3. **O protocolo é imediatamente modularizado** — o RAG absorve o aprendizado como artefato
4. **A versão seguinte já está implícita no problema que a atual não resolveu**

Este padrão tem uma implicação operacional:
a v6.0 também revelará o problema da v7.0 antes que a v7.0 seja concebida explicitamente.

A auditoria que motivou a v6.0 é, ela mesma, evidência do padrão:
foi um ciclo real (auditoria cognitiva estrutural) que revelou a lacuna (topologia invisível).

---

## RELAÇÃO COM OUTROS MÓDULOS

- k-sys-proposta-indexacao-relacional: arquitetura detalhada da v6.0
- k-sys-principios-fundamentais: invariantes que governaram toda a evolução
- k-sys-nucleo-minimo: o núcleo que permanece estável através de toda a evolução
- k-sys-governanca-repositorios: separação produto/core formalizada em v5.0
- CHANGELOG.md: registro operacional das versões (complementar a este documento)
