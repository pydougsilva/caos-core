# k-sys-proposta-indexacao-relacional
versao: 1.0
data: 2026-05-28
tipo: proposta_arquitetural
status: APROVADA — aguardando sprint de implementação
linha_evolutiva: v6.0
origem: auditoria cognitiva estrutural completa (2026-05-28)

---

## MOTIVAÇÃO

A auditoria cognitiva estrutural de 2026-05-28 confirmou que o C.A.O.S v5.0
resolve completamente o problema de continuidade cognitiva entre sessões efêmeras.

Métricas empíricas validadas:
- Tempo de retomada: 8 minutos
- Cobertura de snapshots: 100%
- Nível de continuidade: Pleno
- Confiança de retomada: 94–97%

Mas a auditoria também revelou um limite estrutural do modelo atual:

**O sistema sabe o que existe. Não sabe como as coisas se relacionam.**

Dependências entre módulos são implícitas.
Módulos órfãos são invisíveis até auditoria manual.
Lineage arquitetural requer leitura sequencial de 13+ arquivos de telemetria.
Drift entre instâncias de módulos é descoberto por acidente, não rastreado como dado.
O índice linear (index.md, TAREFA→MÓDULOS) não captura a topologia real do conhecimento.

---

## DIAGNÓSTICO — OS TRÊS LIMITES CONFIRMADOS

### Limite 1 — Dependências invisíveis

`r-estados-ciclo v2.0` depende de `r-git-operacional` para ativar estados
COMMITADO/VERIFICADO/DIVERGENTE. Sem `r-git-operacional`, v2.0 silenciosamente
degrada para comportamento v1.0. Isso não está declarado em nenhum artefato
de forma estruturada — é conhecimento implícito distribuído entre index.md e
a leitura atenta dos módulos individuais.

Confirmadas pela auditoria:
- r-estados-ciclo v2.0 → r-git-operacional (condicional não declarada)
- r-matching-conceito → k-sys-registry-dominios (implícita no fluxo)
- r-recuperacao-contextual → k-sys-registry-dominios (implícita no matching)
- Sprint 5C → contagem manual de ciclos (sem automação, sem artefato de controle)
- afetoeforma módulos /r → versões caos-core (sem versionamento cruzado declarado)

### Limite 2 — Órfãos invisíveis

`k-proj-cooperacao-agentes.md` existe, tem conteúdo válido, não está indexado.
`snapshot-v2.1-inicial.md` existe, está vazio.
`hotfix-padrao.md` (sem prefixo) existe ao lado do artefato canônico.
`prompts/` directory existe sem governança formal.
Nenhum mecanismo no sistema atual detecta isso automaticamente.

### Limite 3 — Lineage como reconstrução, não como recuperação

Para entender como o sistema chegou ao estado atual, é necessário ler 13 arquivos
de telemetria em sequência cronológica e inferir a cadeia causal.
Essa cadeia não existe como artefato — existe apenas como emergência da leitura humana.
k-sys-lineage-arquitetural (criado nesta sprint) é o primeiro passo para resolver isso.

---

## MUDANÇA CONCEITUAL CENTRAL

O C.A.O.S atual é um sistema de **primeira ordem**:
sabe o que existe e serve lookup de módulos por tarefa.

```
TAREFA → [módulo_1, módulo_2, módulo_3]
```

O C.A.O.S v6.0 proposto é um sistema de **segunda ordem**:
sabe como os elementos se relacionam, por que existem e como evoluíram.

```
MÓDULO → DEPENDE_DE → [módulos]
MÓDULO → ATIVA → [módulos condicionais]
MÓDULO → SUPERSEDE → [versão anterior]
CICLO → MODIFICOU → [módulos]
SNAPSHOT → CAPTURA → [estado de domínio]
SNAPSHOT → SUPERSEDE → [snapshot anterior]
DOMÍNIO → DEPENDE_DE → [domínios]
MÓDULO → DIVERGE_DE → [versão no core]
```

Primeira ordem responde: "o que carregar?"
Segunda ordem responde: "por que isso existe?", "o que quebra se eu mudar isso?",
"o que se tornou irrelevante?", "como este estado foi atingido?"

---

## ARQUITETURA PROPOSTA — OS QUATRO PLANOS

```
PLANO 4 — SEMÂNTICA (SBERT)
  Embeddings, similaridade semântica, clustering, drift semântico quantificado

PLANO 3 — GRAFO INSTITUCIONAL (novo)
  Nós, arestas, lineage explícito, drift declarado, órfãos detectados

PLANO 2 — FRONTMATTER RELACIONAL (novo, aditivo)
  Metadados de relação declarados em cada módulo individualmente

PLANO 1 — ESTRUTURA ATUAL (preservada, intocável)
  AGENTS.md, index.md, /r, /k, snapshots, telemetria, registry
```

Cada plano superior é **opcional** em relação ao plano inferior.
O Plano 1 é obrigatório e inalterado.
O sistema opera corretamente com apenas o Plano 1 — fallback estrutural sempre ativo.

---

## PLANO 2 — FRONTMATTER RELACIONAL

O menor passo com o maior impacto estrutural:
adicionar declarações de relação ao cabeçalho de módulos existentes.

### Padrão proposto

```yaml
---
id: r-estados-ciclo
versao: 2.0
tipo: /r
status: ativo                          # ativo | arquivado | deprecado
depends_on:
  hard: []                             # dependências obrigatórias
  soft: [r-git-operacional]            # ativa funcionalidades v2.0
activates:
  condition: r-git-operacional presente
  then: estados COMMITADO | VERIFICADO | DIVERGENTE
supersedes: r-estados-ciclo@1.0
related_to:
  - r-handoff-executor
  - r-commit-governance
  - r-rollback-contextual
populated_by: []                       # domínios cujo estado este módulo monitora
caos_core_version: "2.0"              # versão no caos-core (para detecção de drift)
drift_status: none                     # none | known | unknown
---
```

### Regras de adoção

- Frontmatter mínimo obrigatório: apenas `id:`, `versao:`, `tipo:`.
- Todos os campos relacionais são **opcionais para módulos existentes**.
- **Obrigatório para módulos criados a partir de v5.2**.
- Ausência de relações declaradas não é erro: é sinal para SBERT detectar relações implícitas.
- Aplicação incremental: começar pelos 7 módulos de maior tráfego operacional.

### O que isso resolve

- Dependência de r-estados-ciclo → r-git-operacional: declarada, não inferida
- Qualquer módulo sem `id:` no frontmatter: detectável como candidato a órfão
- `supersedes:` cria cadeia de lineage recuperável sem leitura humana

---

## PLANO 3 — GRAFO INSTITUCIONAL

### Estrutura de diretório proposta

```
rag/graph/
├── relations.yaml          ← grafo de adjacência (derivado do frontmatter)
├── lineage.md              ← cadeia causal da evolução arquitetural
├── drift-map.yaml          ← divergências conhecidas entre instâncias
└── orphans.yaml            ← módulos detectados como órfãos (auditoria + SBERT)
```

Este diretório não existe ainda.
Será criado na sprint de implementação v6.0.
É uma camada de síntese — não substitui nenhuma estrutura existente.

### `relations.yaml` — O grafo como dado estruturado

```yaml
# Formato: adjacência declarativa derivada do frontmatter + curadoria manual

nodes:
  - id: r-estados-ciclo
    type: module
    layer: /r
    status: active
    version: "2.0"
  - id: r-git-operacional
    type: module
    layer: /r
    status: active
    version: "1.0"
  - id: public.pedidos
    type: domain
    family: banco/operacional
    state: estavel

edges:
  - from: r-estados-ciclo
    to: r-git-operacional
    type: soft_depends_on
    note: "estados v2.0 inativos sem r-git-operacional"
  - from: r-matching-conceito
    to: k-sys-registry-dominios
    type: hard_depends_on
  - from: public.pedidos
    to: public.tenants
    type: domain_depends_on
```

**Tipos de aresta canônicos:**

| Tipo | Descrição |
|---|---|
| hard_depends_on | dependência obrigatória — sem o alvo, o módulo não funciona |
| soft_depends_on | dependência condicional — ativa funcionalidades quando presente |
| activates | o módulo origem ativa comportamentos no módulo alvo |
| supersedes | evolução de versão — o origem substitui o alvo |
| related_to | relação semântica — co-carregados frequentemente |
| populated_by | o domínio alvo popula/alimenta o módulo origem |
| domain_depends_on | dependência entre domínios operacionais |
| drifts_from | divergência conhecida entre instância local e versão core |

**O que isso habilita:**
- Busca: "quais módulos dependem de r-git-operacional?" → resposta estrutural
- Análise de impacto: "se remover r-sql-idiomatico, o que é afetado?"
- Detecção de dependências circulares
- Ordenação correta de carregamento por topologia, não por convenção

### `lineage.md` — Arqueologia institucional como artefato

k-sys-lineage-arquitetural.md (criado nesta sprint) é o protótipo deste artefato.
O arquivo `rag/graph/lineage.md` será a versão consolidada, mantida sincronizada
com a evolução do sistema, no formato de grafo temporal.

### `drift-map.yaml` — Divergências como dado de primeira classe

```yaml
# Formato: registro estruturado de divergências conhecidas

drifts:
  - module: k-proj-caos-metodo
    caos_core_version: "1.0"
    instancia_version: "1.0"
    drift_type: nomenclatural
    detail: "usa CODEX/CLAUDE em vez de papéis institucionais"
    severity: low
    action_pending: hardening-nominal
    detected: 2026-05-25

  - module: r-handoff-codex
    caos_core_status: preserved (historical + complementary)
    instancia_status: absent
    drift_type: structural
    detail: "afetoeforma usa exclusivamente r-handoff-executor"
    severity: medium
    action_pending: none  # divergência intencional documentada
    detected: 2026-05-28
```

Elimina o padrão atual: "drift descoberto por leitura de telemetria".
Drift passa a ser artefato de primeira classe, não efeito colateral de auditoria.

### `orphans.yaml` — Órfãos como dado estruturado

```yaml
orphans:
  detected: 2026-05-28
  method: auditoria_estrutural  # auditoria_estrutural | sbert | ambos
  items:
    - path: rag/r/hotfix-padrao.md
      reason: duplicata não removida — cópia arquivada em rag/arquivo/r/
      action: remover da pasta ativa
      severity: low

    - path: rag/docs/ciclos/snapshots/snapshot-v2.1-inicial.md
      reason: arquivo vazio (1 linha) — vestigial não populado
      action: popular com conteúdo ou remover
      severity: low

    - path: rag/k/projeto/k-proj-cooperacao-agentes.md
      reason: não indexado em rag/index.md — conteúdo sobrepõe AGENTS.md e k-proj-caos-metodo
      action: indexar ou arquivar
      severity: medium
```

---

## EVOLUÇÃO DO REGISTRY — v1.0 → v3.0

O registry atual (v1.0) é um catálogo de domínios com aliases e pesos.
O Sprint 5C planejado (v2.0) adiciona dependências e staleness budget.
A visão v3.0 integra o grafo e formaliza o contador de ciclos.

### Extensão proposta por entrada de domínio

```yaml
id: public.pedidos
canonico: public.pedidos
aliases:
  - pedidos    (peso: 1.0)
  - pedido     (peso: 0.9)
  - orders     (peso: 0.8)
familia: banco/operacional
modulos_padrao:
  - r/r-rls-padrao
  - k/banco/k-db-tabelas-core

# CAMPOS NOVOS — v3.0

depende_de:
  - public.tenants     # FK estrutural
  - public.perfis      # user_id reference
afeta:
  - public.audit_logs  # operações geram entradas
modulos_relacionados:
  - r/r-sql-idiomatico
  - r/r-hotfix-padrao

ciclos_acumulados: 5   # contador formal (substitui contagem manual)
ultimo_ciclo: T-MT.1a
estado_atual: estavel
drift_status: none
staleness_budget: 90d

embedding_path: graph/embeddings/domain-pedidos.npy  # reservado (já existe v1.0)
```

### Sequência de evolução

- v2.0 (Sprint 5C — gate: 30 ciclos): adiciona depends_on, staleness_budget, ciclos_acumulados
- v3.0 (sprint de implementação v6.0): adiciona afeta, modulos_relacionados, drift_status, embedding_path ativo

---

## EVOLUÇÃO DOS SNAPSHOTS

Snapshots não se tornam nós do grafo no sentido de substituição —
eles **adquirem cabeçalho relacional** como extensão não-destrutiva do template.

### Extensão proposta do template

```yaml
# Seção adicional ao final do cabeçalho YAML de snapshots

relacoes:
  supersede: snapshot-[dominio]-N          # snapshot que este substitui
  supersedido_por: null                    # null = snapshot mais recente
  ciclos_geradores:
    - id: ciclo-id
      data: YYYY-MM-DD
      tipo: operacional | auditoria | rollback
  modulos_validados:
    - r-rls-padrao
    - r-sql-idiomatico
  estado_pos_ciclo: estavel | degradado | em_manutencao
  tipo_snapshot: base | incremental | auditoria | rollback
```

**Regra de compatibilidade:**
Snapshots históricos **não são retroativamente modificados**.
Imutabilidade após homologação é princípio inviolável.
O novo cabeçalho se aplica apenas a snapshots criados a partir de v5.4.
A cadeia histórica permanece como evidência arqueológica na forma original.

**O que isso habilita:**
Um agente que carrega um snapshot sabe imediatamente:
- sua posição na cadeia (supersede/supersedido_por)
- quais módulos foram validados no ciclo que o produziu
- se existe snapshot mais recente
- que tipo de operação gerou este estado

---

## PLANO 4 — SBERT COMO SERVIÇO OFFLINE

### Posicionamento correto — o erro a evitar

Erro: integrar SBERT no **runtime operacional**.
Cada consulta de módulo passaria por um modelo de embedding.
Isso cria dependência de infraestrutura Python em cada sessão operacional.
Viola o princípio de isolamento de sessão e o princípio de economia cognitiva.

**Correto: SBERT como serviço de indexação offline.**

```
RUNTIME (sem SBERT — operação normal)
├── AGENTS.md
├── index.md
├── modules /r e /k
└── graph/relations.yaml   ← consulta resultados pré-computados
        ↑
        │ consulta
        │
INDEXAÇÃO SBERT (offline, sob demanda, explícita)
└── python caos-index.py --target rag/
    ├── gera/atualiza graph/embeddings/
    ├── gera/atualiza graph/orphans.yaml
    └── gera/atualiza drift-map.yaml (componente semântico)
```

A indexação ocorre quando:
- Novo módulo é criado
- Módulo existente sofre modificação substancial
- Auditoria de staleness é solicitada explicitamente
- Usuário invoca explicitamente

O runtime consulta **resultados** pré-computados (JSON/YAML), não o modelo diretamente.

### As 5 funções do SBERT no C.A.O.S

**1. Retrieval semântico — augmentação do matching por alias**
Quando r-matching-conceito retorna SEM_MATCH por alias,
o SBERT encontra o domínio/módulo mais semanticamente próximo da descrição da tarefa.
Resolve o caso em que o usuário descreve um problema com vocabulário diferente dos aliases.

**2. Detecção de órfãos semânticos**
Módulos sem vizinhos semânticos no grafo declarado → candidatos a órfãos.
k-proj-cooperacao-agentes.md teria sido detectado automaticamente:
conteúdo semanticamente próximo de AGENTS.md e k-proj-caos-metodo.md,
mas sem conexão no grafo. Flag: "módulo potencialmente redundante ou não indexado."

**3. Detecção de drift semântico quantificado**
Comparação de embeddings entre versão local de um módulo e versão caos-core.
Divergência acima de threshold → drift semântico registrado no drift-map.yaml.
Não apenas "existe drift" — mas "o conteúdo divergiu em N% de similaridade semântica".

**4. Lineage por similaridade histórica**
Dado um snapshot de domínio, encontrar os snapshots historicamente mais similares.
Quando novo agente precisa de contexto sobre um domínio, o SBERT ranqueia snapshots
por similaridade semântica com a situação atual — não apenas por data.
Resolve o caso em que o snapshot mais recente é menos informativo que um base anterior.

**5. Clustering institucional automático**
SBERT revela famílias de módulos implícitas que a taxonomia manual não capturou.
Exemplo esperado: {r-rls-padrao, r-sql-idiomatico, k-db-funcoes} → cluster banco-operacional.
{r-git-operacional, r-commit-governance, r-rollback-contextual, r-replay-operacional} → cluster ledger.
Alimenta a estrutura de famílias no registry e pode revelar famílias implícitas.

### Infraestrutura mínima

```
Modelo: sentence-transformers/all-MiniLM-L6-v2
Tamanho: ~80MB (download único)
Dependências: sentence-transformers, numpy, pyyaml
Execução: script local, sem servidor, sem porta, sem daemon
Output: arquivos JSON/YAML pré-computados em rag/graph/
```

O script `caos-index.py` será criado na sprint de implementação v6.0.
Não é um serviço. Executa, produz arquivos, termina.
Nenhuma dependência de rede durante a operação.

---

## IMPACTO POR COMPONENTE EXISTENTE

### index.md — sobrevive com papel mais preciso

O index.md atual acumula três responsabilidades:
- Tabela de roteamento (TAREFA→MÓDULOS) — permanece no index.md
- Registro de módulos existentes — permanece no index.md
- Registro de evolução arquitetural — migra gradualmente para graph/lineage.md

O index.md não é substituído. Torna-se mais focado no seu papel primário:
**lookup operacional rápido para agentes com contexto mínimo.**

Em v6.0, o index.md pode ser **validado** contra relations.yaml (não gerado por ele).
Discrepâncias entre o que index.md declara e o que relations.yaml contém
geram flags em orphans.yaml — mecanismo de coerência bidireccional.

### AGENTS.md — sem mudança estrutural, extensão possível

Em v6.0, o fluxo operacional pode ganhar uma etapa opcional:

```
Etapa 0c → Consultar grafo institucional (graph/relations.yaml)
           se grafo presente → enriquecer contexto de dependências
           se grafo ausente → continuar com comportamento atual (fallback)
```

A prioridade de autoridade permanece inalterada:
AGENTS.md > Prompt de Sessão > Skills > index.md > Inferência própria.

### Snapshots — extensão não-destrutiva

Snapshots históricos: imutáveis, preservados como evidência arqueológica.
Novos snapshots (a partir de v5.4): template com cabeçalho relacional.
A cadeia de snapshots passa a ser navegável sem leitura sequencial.

### Telemetria — sem mudança estrutural

Telemetria continua como está.
O ganho vem de graph/lineage.md que sintetiza o que estava distribuído nas telemetrias.
Não de modificar as telemetrias em si.

### Registry — evolução em duas etapas

v2.0 = Sprint 5C (gate: 30 ciclos): campos operacionais (staleness, ciclos_acumulados)
v3.0 = sprint implementação v6.0: campos relacionais completos

### Continuidade cognitiva — reforçada, não substituída

A continuidade atual (8 min, 94–97%, AGENTS.md + index.md) permanece o protocolo padrão.

O grafo adiciona uma modalidade de retomada mais rica:
em vez de apenas carregar o snapshot mais recente por domínio,
o agente pode consultar graph/relations.yaml para entender quais módulos precisam
ser co-carregados dado o snapshot ativo — reduzindo risco de carregamento incompleto
em sessões que atravessam múltiplos domínios com dependências.

---

## RISCOS E MITIGAÇÕES

### Risco 1 — Complexidade cresce mais rápido que o corpus

Um grafo de 25 módulos é trivial.
Um grafo de 100+ módulos pode gerar overhead de manutenção maior que o benefício.

**Mitigação:** O grafo é **derivado**, não primário.
O frontmatter dos módulos é source of truth.
O relations.yaml é gerado ou validado a partir do frontmatter — não mantido manualmente.
Manutenção humana do grafo é exceção, não regra.

### Risco 2 — Burocracia por módulo criado

Se criar módulo requer declarar 8 campos de relação, a fricção de criação aumenta.
Vai contra o princípio de adoção incremental.

**Mitigação:** Frontmatter mínimo obrigatório = apenas `id:`, `versao:`, `tipo:`.
Todos os campos relacionais são opcionais.
Ausência de relações declaradas → sinal para SBERT detectar relações implícitas.

### Risco 3 — Grafo como single point of failure

Se agentes passam a depender de relations.yaml e ele diverge da realidade,
o sistema pode tomar decisões baseadas em relações stale.

**Mitigação:** Fallback estrutural obrigatório.
Se `graph/` não existir ou estiver flagado como stale, o sistema opera em modo estrutural
(index.md apenas) — comportamento atual.
O grafo é enriquecimento, nunca requisito.

### Risco 4 — Perda da elegância de "7 arquivos"

O núcleo mínimo pode parecer mais complexo.
Adoção em uma tarde pode parecer impossível.

**Mitigação:** A sequência de adoção é preservada e estendida:

```
Nível 0 — Núcleo (7 arquivos)        → sem mudança
Nível 1 — Continuidade estruturada   → sem mudança
Nível 2 — Governança operacional     → sem mudança
Nível 3 — Persistência verificável   → sem mudança
Nível 4 — Frontmatter relacional     → novo, opcional
Nível 5 — Grafo institucional        → novo, opcional
Nível 6 — SBERT                      → novo, opt-in, requer Python
```

Projetos novos e simples nunca precisam dos Níveis 4–6.
O C.A.O.S permanece adotável em uma tarde.

### Risco 5 — Grafo cross-repo: qual é canônico?

caos-core e instâncias de produto terão grafos com nós compartilhados.
Qual grafo é autoritativo para módulos compartilhados?

**Mitigação:** O grafo do caos-core é canônico para módulos genéricos.
O grafo de instâncias de produto estende o grafo do core com nós produto-específicos.
Seguindo o mesmo princípio de k-sys-governanca-repositorios.
O campo `caos_core_version:` no frontmatter é o elo formal entre instâncias.

### Risco 6 — SBERT como dependência de infraestrutura

Requer Python 3.x + sentence-transformers (~80–500MB, download único).
Não disponível em todos os ambientes.

**Mitigação:** SBERT é totalmente opt-in.
Sistema opera sem SBERT em modo estrutural.
r-matching-conceito continua como fallback para matching de domínio.
O Plano 4 (SBERT) nunca é pré-requisito para os Planos 1–3.

---

## ROADMAP EVOLUTIVO

### v5.x — Consolidação e Fundação Relacional

| Versão | Marco | Gate |
|---|---|---|
| v5.1 | Limpeza de órfãos confirmados + promoção de artefatos pendentes | nenhum |
| v5.2 | Frontmatter relacional: especificação + aplicação nos 7 módulos de maior tráfego | nenhum |
| v5.3 = Sprint 5C | Registry v2.0: staleness_budget, ciclos_acumulados, depends_on | 30 ciclos reais acumulados |
| v5.4 | Template de snapshot com cabeçalho relacional | v5.3 concluído |
| v5.5 | SBERT piloto: apenas módulos /r, script caos-index.py | v5.2 concluído |

### v6.0 — Grafo Institucional Completo

**Estrutura implementada:**

```
rag/graph/
├── relations.yaml      ← grafo declarativo completo
├── lineage.md          ← arqueologia institucional como artefato
├── drift-map.yaml      ← divergências como dado de primeira classe
├── orphans.yaml        ← órfãos detectados (SBERT + estrutural)
└── embeddings/
    ├── index.json
    ├── modules/
    └── snapshots/
```

**Novos módulos institucionais:**

| Módulo | Tipo | Finalidade |
|---|---|---|
| k-sys-grafo | /k/sistema | estrutura do grafo, schemas, convenções |
| r-grafo-institucional | /r | regras para construir, validar e consultar o grafo |
| k-sys-sbert | /k/sistema | integração SBERT: como indexar, consultar, interpretar |

**AGENTS.md evolui para v6.0:**
Adiciona Etapa 0c (consulta ao grafo) como etapa opcional com fallback explícito.

**index.md:**
Validado contra relations.yaml. Discrepâncias geram flags em orphans.yaml.
Continua sendo o documento humano-legível primário.
O grafo é a contraparte estrutural — não o substituto.

**Registry v3.0:**
Campos relacionais completos + embedding_path ativo.

### v6.5 — Grafo Cross-Repo

- caos-core como grafo canônico para módulos genéricos
- Protocol de extensão para instâncias de produto
- drift-map.yaml unificado com evidência quantitativa (SBERT) e qualitativa (declarada)
- Sincronização de grafo seguindo o mesmo fluxo de k-sys-governanca-repositorios

### v7.0 — Hipotético (não projetar ainda)

Condicional: corpus > 50 domínios / 100+ módulos por projeto.

Possíveis direções:
- SQLite com extensões de grafo para travessia eficiente
- API de consulta local (HTTP) para ferramentas externas
- Automação parcial de promoção/arquivamento com base em grafo + SBERT

Não implementar antes de validar empiricamente que YAML é insuficiente.
O princípio "filesystem como source of truth" permanece até ser refutado por necessidade real.

---

## O QUE PERMANECE, O QUE MORRE, O QUE NASCE

### Permanece (intocável)

- AGENTS.md como autoridade máxima
- index.md como lookup operacional primário
- A separação /r (regras) / /k (conhecimento)
- O protocolo de ciclo em 10 etapas
- A governança humana como único gate obrigatório
- O axioma de isolamento de sessão
- Os 7 artefatos do núcleo mínimo
- A sequência de adoção incremental
- A compatibilidade histórica com snapshots existentes
- O princípio "fallback é degradação controlada, não falha"
- O princípio "filesystem como source of truth"

### Morre (precisa ser encerrado antes de v6.0)

- `rag/r/hotfix-padrao.md` duplicata em afetoeforma — arquivo ativo sem ser o canônico
- `snapshot-v2.1-inicial.md` vazio — vestigial sem valor, populate ou remova
- O contador informal de ciclos para Sprint 5C — substituído por `ciclos_acumulados:` no registry
- `prompts/` como camada não-governada — formalizar em skills/ ou arquivar
- A distinção "o index.md é o mapa" / "relações não existem como artefato" — inválida em v6.0

### Nasce

- Frontmatter relacional como padrão para novos módulos (v5.2)
- `rag/graph/` como camada de síntese institucional (v6.0)
- `caos-index.py` como serviço de indexação SBERT offline (v5.5)
- `k-sys-grafo.md` — conhecimento da estrutura do grafo (v6.0)
- `r-grafo-institucional.md` — regras de governança do grafo (v6.0)
- `k-sys-sbert.md` — integração SBERT (v6.0)
- `graph/lineage.md` — arqueologia institucional como artefato de grafo (v6.0)
- `graph/drift-map.yaml` — evidência de divergência de primeira classe (v6.0)
- Registry v3.0 com campos relacionais completos (v5.3 → v6.0)
- Template de snapshot com cabeçalho relacional (v5.4)

---

## CRITÉRIOS DE READINESS PARA SPRINT DE IMPLEMENTAÇÃO

A sprint de implementação v6.0 pode ser iniciada quando:

```
□ v5.1 concluído: órfãos limpos, artefatos pendentes promovidos
□ v5.2 concluído: frontmatter relacional aplicado em ≥ 7 módulos
□ v5.3 = Sprint 5C concluído: registry v2.0 operacional (gate: 30 ciclos)
□ v5.4 concluído: template de snapshot com cabeçalho relacional
□ v5.5 concluído: SBERT piloto validado em módulos /r

Meta mínima alternativa (fast path):
□ v5.1 + v5.2 + v5.5 concluídos
□ graph/ pode ser inicializado com relações manuais antes do SBERT
□ v5.3 e v5.4 podem ser feitos em paralelo com v6.0
```

---

## NOTA ARQUITETURAL FINAL

O maior risco desta evolução não é técnico. É epistêmico.

Um sistema que sabe demais sobre si mesmo pode tornar-se mais interessante para manter
do que para usar. O r-anti-burocracia.md e o princípio de economia cognitiva precisam ser
aplicados à própria camada de grafo com o mesmo rigor que são aplicados aos módulos operacionais.

O grafo serve ao sistema — nunca o contrário.

Se o overhead de manutenção do grafo superar o benefício de retomada,
o grafo deve ser simplificado ou arquivado sem hesitação.
Degradar controladamente continua sendo preferível a falhar sofisticadamente.

---

## RELAÇÃO COM OUTROS MÓDULOS

- k-sys-lineage-arquitetural: lineage completo que motivou esta proposta
- k-sys-principios-fundamentais: invariantes que esta proposta deve respeitar
- k-sys-nucleo-minimo: núcleo que permanece inalterado pela proposta
- k-sys-governanca-repositorios: modelo de separação que o grafo cross-repo herda
- r-anti-burocracia: limites que a camada de grafo deve respeitar
- r-staleness-detection: responsável por detectar staleness no grafo (extensão futura)
