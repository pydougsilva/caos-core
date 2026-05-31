# k-sys-handoff-institucional
versao: 1.0

## OBJETIVO

Definir o que um novo agente executor precisa para assumir
o projeto sem depender de memória implícita do agente anterior.

Responde à pergunta:
"o que todo agente novo deve verificar antes de operar?"

---

## PRINCÍPIO CENTRAL

Continuidade operacional não depende do agente.
Depende dos artefatos que o agente deixou.

Um projeto que não pode ser retomado por um novo agente
sem intervenção humana extensa não tem continuidade real.

---

## PROTOCOLO DE ENTRADA — NOVO AGENTE

### Etapa 1 — Ler a autoridade máxima

```
Arquivo: AGENTS.md
Verificar: versão, fluxo operacional, prioridade de autoridade
Tempo estimado: 2-3 minutos de leitura
```

### Etapa 2 — Mapear o conhecimento disponível

```
Arquivo: rag/index.md
Verificar: módulos existentes, tarefas mapeadas, evolução arquitetural
Sinalizar: referências a módulos que não existem no filesystem
```

### Etapa 3 — Verificar estado operacional atual

```
Verificar:
  - existe algum arquivo em rag/locks/?
    → SIM: há ciclo ativo — ver r-concurrency-guard
    → NÃO: nenhum ciclo ativo
  - há snapshots com estado_atual ≠ CONCLUÍDO?
    → verificar r-recuperacao-contextual
```

### Etapa 4 — Verificar o histórico Git

```
git log --oneline --all
Verificar:
  - há commits não-mergeados?
  - há branches ops/ abertas?
  - qual foi o último commit e o que ele representou?
```

### Etapa 5 — Verificar staleness dos artefatos principais

Usar r-staleness-detection como guia.

```
Principais verificações de entrada:
  - versão em AGENTS.md vs versão esperada
  - versão em rag/index.md vs módulos existentes no filesystem
  - snapshot mais recente de cada domínio: data e estado
  - k-proj-identidade do projeto: fase atual vs artefato de produto principal
```

### Etapa 6 — Registrar telemetria de entrada

Criar `rag/docs/ciclos/telemetria/telemetria-[data]-entrada.md`
usando r-telemetria-cognitiva.

Declarar explicitamente:
- o que o agente entende
- o que o agente NÃO entende
- hipóteses feitas
- nível de confiança por área

### Etapa 7 — Perguntar antes de assumir

Se qualquer área tem confiança < 50%, sinalizar ao usuário
antes de propor qualquer ciclo operacional nessa área.

---

## ARTEFATOS QUE DEVEM SEMPRE EXISTIR E ESTAR ÍNTEGROS

| Artefato | Localização | O que verifica |
|---|---|---|
| Autoridade operacional | AGENTS.md | fluxo, versão, prioridade |
| Mapa de módulos | rag/index.md | módulos existentes, tarefas |
| Identidade do projeto | rag/k/projeto/k-proj-identidade.md | produto, fase, stack |
| Registry de domínios | rag/k/sistema/k-sys-registry-dominios.md | domínios, aliases, snapshots |
| Snapshot mais recente | rag/r/r-recuperacao-contextual.md | decisões ativas |
| Telemetria da sessão anterior | rag/docs/ciclos/telemetria/ | contexto não-persistido |

---

## O QUE UM AGENTE ANTERIOR DEVE DEIXAR

Antes de encerrar qualquer sessão com ciclo operacional:

```
□ Snapshot atualizado com resultado do ciclo
□ Telemetria da sessão criada
□ Locks removidos (se ciclo CONCLUÍDO)
□ index.md atualizado se novos módulos foram criados
□ Commit institucional criado se houve execução
```

Se a sessão foi encerrada abruptamente (interrompida), o próximo agente
deve verificar locks, snapshots com estado EXECUTANDO e commits não-mergeados.

---

## PERGUNTAS QUE UM NOVO AGENTE DEVE CONSEGUIR RESPONDER DOS ARTEFATOS

Se qualquer uma dessas perguntas não tiver resposta nos artefatos,
é uma lacuna de continuidade institucional:

```
1. Qual é a fase atual do produto?
2. Qual foi o último ciclo operacional e qual foi o resultado?
3. Há algum domínio com ciclo ativo?
4. Quais são os riscos arquiteturais conhecidos?
5. Qual é a próxima entrega prioritária?
6. Quais domínios têm histórico de operações?
7. Há módulos RAG suspeitos de staleness?
```

---

## SINAIS DE BOA CONTINUIDADE INSTITUCIONAL

O sistema está funcionando bem como infraestrutura cognitiva quando:
- um novo agente consegue reconstruir o contexto do projeto em < 30 minutos
- a confiança de reconstrução está acima de 70% nas áreas críticas
- nenhuma hipótese crítica precisou ser feita sem base em artefatos
- o próximo ciclo pode ser iniciado sem intervenção humana extensa

---

## SINAIS DE DEGRADAÇÃO DE CONTINUIDADE

O sistema está falhando como infraestrutura quando:
- módulos referenciados não existem
- snapshots têm mais de 60 dias sem atualização em domínios ativos
- o histórico Git não tem commits institucionais por mais de 2 semanas de trabalho
- o agente precisa ler mais de 30% do código de produto para entender o contexto
- nenhuma telemetria de sessão anterior existe
