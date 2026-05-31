# k-sys-principios-fundamentais
versao: 1.0
data: 2026-05-28
tipo: principios_imutaveis
status: canônico

## OBJETIVO

Registrar os invariantes arquiteturais do C.A.O.S:
as restrições que toda versão futura deve respeitar,
independentemente de qual problema está resolvendo.

Este documento é constitucional, não operacional.
Não descreve como o sistema funciona — descreve o que o sistema não pode violar.

Toda futura RFC, ADR ou proposta arquitetural deve ser avaliada contra estes princípios.
Um princípio só pode ser revisado com justificativa explícita de por que a restrição
deixou de ser válida — e com aprovação humana formal.

---

## OS PRINCÍPIOS

---

### P1 — Filesystem como source of truth absoluto

O estado canônico do sistema reside em arquivos de texto.
Markdown e YAML são os formatos primários de memória institucional.

Implicações:
- Nenhum banco de dados, nenhuma API, nenhum serviço externo substitui arquivos
- O sistema deve operar corretamente em qualquer ambiente com Git e um editor de texto
- Formatos binários (embeddings, índices) são **derivados** dos arquivos — nunca primários
- Se um arquivo e um banco divergem, o arquivo é verdade

Violação típica que P1 proíbe:
"Vamos mover snapshots para PostgreSQL para performance."
Resposta: snapshots em banco são um cache, nunca o source of truth.

---

### P2 — Markdown e YAML são os formatos primários

Toda memória institucional que precisa ser lida por humanos ou agentes
deve existir em Markdown ou YAML estruturado.

Implicações:
- SQL é para dados de produto — nunca para artefatos institucionais C.A.O.S
- Formatos proprietários (Word, PDF) não são formatos institucionais
- Embeddings (.npy, .bin) são indexação — nunca memória primária
- JSON é aceitável para outputs de ferramentas, não para artefatos que humanos mantêm

Violação típica que P2 proíbe:
"Vamos usar DuckDB para armazenar o registry de domínios."
Resposta: registry em DuckDB é um cache derivável. O YAML é o source of truth.

---

### P3 — SBERT nunca participa do runtime operacional

O modelo de embedding processa módulos e snapshots offline, sob demanda explícita.
Os resultados (arquivos JSON/YAML pré-computados) são o que o runtime consulta.

Implicações:
- Nenhum agente inicia execução de Python durante uma sessão operacional
- A indexação SBERT é um ciclo separado, não uma etapa do ciclo operacional
- Ausência de embeddings pré-computados → fallback para matching estrutural (r-matching-conceito)
- SBERT é opt-in total: o sistema funciona sem ele

Violação típica que P3 proíbe:
"Vamos chamar sentence-transformers em tempo real para cada matching de domínio."
Resposta: isso cria dependência de Python no runtime — viola P3.

---

### P4 — Grafo é camada derivada, nunca camada primária

O grafo institucional (relations.yaml) é derivado dos frontmatters dos módulos
e de curadoria manual explícita.
Os módulos são a fonte de verdade — o grafo é síntese.

Implicações:
- Se um módulo e o grafo divergem sobre uma relação, o módulo prevalece
- O grafo pode ser reconstruído a partir dos módulos — não o contrário
- Agentes que não encontrarem o grafo continuam operando (fallback: index.md)
- Módulos devem ser compreensíveis sem o grafo

Violação típica que P4 proíbe:
"Vamos declarar relações apenas no grafo, não no frontmatter dos módulos."
Resposta: isso inverte a dependência — o grafo passaria a ser primário.

---

### P5 — Fallback estrutural sempre obrigatório

Para cada capacidade de nível superior (matching semântico, grafo, telemetria avançada),
deve existir um comportamento de fallback definido e testado.

Implicações:
- O sistema nunca falha totalmente — apenas degrada de forma controlada
- O fallback deve ser documentado no mesmo módulo que descreve a capacidade
- A degradação deve ser explicitamente comunicada ao agente (sinalização, não silêncio)
- Fallback de Nível 0: AGENTS.md + index.md são suficientes para operação básica

Violação típica que P5 proíbe:
"Se o grafo não existir, lançar erro e interromper o ciclo."
Resposta: operar em modo estrutural (index.md apenas) sem erro.

---

### P6 — Simplicidade operacional é prioridade

A capacidade de iniciar um projeto com o C.A.O.S em uma tarde
deve ser preservada em todas as versões.

Implicações:
- O núcleo mínimo de 7 artefatos não pode crescer
- Cada novo módulo deve justificar sua existência por necessidade operacional real
- Documentação não deve crescer mais rápido que o corpus operacional que documenta
- Overhead de manutenção de artefatos institucionais não deve superar o benefício

Violação típica que P6 proíbe:
"Vamos adicionar 5 campos obrigatórios ao template de snapshot."
Resposta: campos obrigatórios são custo de adoção. Só adicionar se o benefício for maior.

---

### P7 — Degradar controladamente supera falhar sofisticadamente

Um sistema em modo degradado que continua operando é superior a um sistema
sofisticado que falha quando uma dependência está ausente.

Implicações:
- Módulos avançados ausentes → comportamento reduzido, nunca erro total
- O valor de uma capacidade é proporcional ao custo de sua ausência, não à sua complexidade
- Modos de operação devem ser explicitamente definidos: pleno, estrutural, mínimo
- A contingência de executor (r-executor-contingencia) é um exemplo deste princípio em ação

Violação típica que P7 proíbe:
"Sem o grafo, não podemos garantir carregamento correto de módulos."
Resposta: sem o grafo, o sistema usa index.md como sempre fez — e funciona.

---

### P8 — Continuidade cognitiva é o núcleo inviolável

A capacidade de um novo agente retomar trabalho de sessões anteriores
é o objetivo primário do sistema. Todas as outras capacidades são secundárias.

Implicações:
- Nenhuma adição arquitetural pode comprometer o tempo de retomada (meta: < 15 min)
- Snapshots e telemetria de sessão são os artefatos mais críticos do sistema
- Qualquer módulo que aumenta overhead de sessão sem benefício de continuidade é candidato a pruning
- A métrica `tempo_retomada_estimado_min` em telemetria é indicador de saúde do sistema

Violação típica que P8 proíbe:
"Vamos exigir leitura do grafo completo antes de iniciar qualquer sessão."
Resposta: isso aumentaria o tempo de retomada para O(módulos) — viola P8.

---

### P9 — Isolamento de sessão como axioma

O estado operacional existe exclusivamente nos artefatos carregados na sessão ativa.
Nenhum estado persiste implicitamente entre sessões.

Implicações:
- AGENTS.md deve ser carregado em toda sessão — sem exceção
- Agente sem artefatos = agente genérico, independentemente de sessões anteriores
- Embeddings, grafos e caches são pré-computados — não são estado de sessão
- Fine-tuning de modelo com conhecimento institucional é violação deste axioma

Violação típica que P9 proíbe:
"O modelo já foi fine-tuned com os snapshots — não precisamos carregá-los."
Resposta: fine-tuning institucional é anti-padrão explicitamente proibido.

---

### P10 — Governança humana é o único gate obrigatório

Nenhuma alteração estrutural pode ser executada sem validação humana explícita.
Automação pode preparar, propor e executar — mas não pode aprovar em nome do usuário.

Implicações:
- Ciclos sem gate humano são inválidos, mesmo em modo automatizado
- O gate humano não pode ser substituído por "confiança em sessão anterior"
- Modo degradado (único agente acumulando papéis) reforça o gate, não o elimina
- Automação de grafo, SBERT e indexação não são gates — são preparação para o gate

Violação típica que P10 proíbe:
"Como o agente executou ciclos anteriores corretamente, pode pular o gate humano."
Resposta: histórico de acerto não substitui governança humana — cada ciclo tem seu gate.

---

### P11 — Adoção incremental sobre instalação completa

O C.A.O.S é adotado gradualmente — nunca instalado de uma vez.

Implicações:
- A sequência de níveis (0 → 6) define ordem natural de adoção
- Projetos novos começam no Nível 0 e adicionam capacidades por demanda real
- A documentação de um nível não deve depender de compreender todos os níveis superiores
- "Overhead de setup proporcional ao problema que resolve" é critério de validação de cada nível

Violação típica que P11 proíbe:
"Para usar o C.A.O.S, é necessário configurar o grafo e o SBERT desde o início."
Resposta: Nível 0 é suficiente para iniciar. Grafo e SBERT são Níveis 5 e 6.

---

### P12 — Aditivo sobre substitutivo

Cada versão do C.A.O.S adiciona capacidades sem remover as anteriores.
Arquivos são arquivados, nunca deletados sem decisão explícita.

Implicações:
- Novos protocolos coexistem com protocolos anteriores durante período de transição
- Módulos deprecated recebem status, não são removidos imediatamente
- Compatibilidade retroativa com snapshots históricos é obrigatória
- Remoção requer r-module-pruning como guia e decisão humana explícita

Violação típica que P12 proíbe:
"Vamos substituir r-handoff-codex por r-handoff-executor e deletar o antigo."
Resposta: r-handoff-codex foi preservado como evidência histórica e protocolo complementar.

---

### P13 — Compatibilidade histórica como restrição, não sugestão

Artefatos homologados não são reescritos retroativamente.
O histórico de ciclos é evidência institucional permanente.

Implicações:
- Snapshots após homologação são imutáveis
- Commits institucionais não são alterados (--amend é proibido em commits homologados)
- Telemetria de sessões passadas não é editada, mesmo que revele erros
- Erros históricos são documentados em artefatos novos, não apagados nos originais

Violação típica que P13 proíbe:
"Vamos reescrever os commits antigos para seguir o novo formato."
Resposta: commits anteriores à governança formal são evidência histórica — preservar.

---

### P14 — Papéis sobre identidades

O sistema depende de papéis institucionais, nunca de identidades de ferramentas.
`agente_orquestrador` e `agente_executor` são papéis — qualquer ferramenta pode implementá-los.

Implicações:
- Contratos ativos (handoff, snapshot, telemetria) usam nomes de papéis
- Evidências históricas podem usar nomes de ferramentas (fidelidade arqueológica)
- Um novo fornecedor de IA adota os papéis sem exigir atualização de protocolos
- A descontaminação nominal (v5.0) é um exemplo permanente deste princípio

Violação típica que P14 proíbe:
"Nosso workflow requer que o orquestrador seja Claude especificamente."
Resposta: o orquestrador é um papel. Claude é uma implementação possível.

---

### P15 — Módulos respondem uma única pergunta

Cada módulo /r ou /k possui escopo isolado e responde exatamente uma pergunta.

Implicações:
- Um módulo não deve mesclar regras operacionais com conhecimento estrutural
- Se um módulo responde duas perguntas diferentes, deve ser dividido
- Sobreposição de conteúdo entre módulos é candidata a pruning
- O tamanho do módulo é consequência do escopo — não é critério primário

Violação típica que P15 proíbe:
"Vamos consolidar r-rls-padrao e r-sql-idiomatico em um único módulo de banco."
Resposta: são perguntas diferentes — RLS é sobre isolamento, SQL é sobre qualidade de migrations.

---

## RELAÇÃO COM OUTROS MÓDULOS

- k-sys-lineage-arquitetural: mostra como estes princípios emergiram empiricamente
- k-sys-proposta-indexacao-relacional: proposta que deve ser avaliada contra estes princípios
- r-anti-burocracia: implementação operacional de P6 (simplicidade) e P7 (degradação controlada)
- k-sys-nucleo-minimo: manifestação concreta de P6, P11 e P15
- r-continuidade-cognitiva: implementação operacional de P8 e P9

---

## QUANDO REVISAR ESTE DOCUMENTO

Um princípio pode ser revisado quando:

1. Uma versão do sistema demonstra empiricamente que a restrição gera mais custo que benefício
2. Um novo problema operacional real não pode ser resolvido sem violar o princípio
3. A revisão é proposta formalmente, explicada e aprovada com gate humano explícito

A revisão documenta:
- qual princípio foi alterado
- por que a restrição original deixou de ser válida
- qual ciclo ou evidência motivou a revisão
- qual a nova formulação
