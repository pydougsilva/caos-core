# r-telemetria-cognitiva
versao: 1.2

## OBJETIVO

Definir o registro mínimo de telemetria por sessão operacional.

Responde à pergunta:
"o que este agente fez, como, com que confiança e quais problemas encontrou?"

---

## PRINCÍPIO CENTRAL

Um agente que não registra seu raciocínio não pode ser auditado.
Um agente que registra demais cria ruído que nenhum humano lê.

A telemetria cognitiva mínima existe no equilíbrio:
suficiente para reconstrução, insuficiente para sobrecarga.

---

## QUANDO REGISTRAR

Registrar telemetria ao final de toda sessão que:
- iniciou pelo menos um ciclo operacional, ou
- realizou auditoria do projeto, ou
- assumiu o projeto como novo agente (entry telemetry)

Não registrar telemetria para:
- consultas simples sem análise arquitetural
- perguntas sobre o produto sem implicação operacional

---

## LOCALIZAÇÃO

```
rag/docs/ciclos/telemetria/
  telemetria-[YYYY-MM-DD]-[sessao-id-curto].md
```

Exemplo:
```
rag/docs/ciclos/telemetria/telemetria-20260512-abc1.md
```

---

## ESTRUTURA DO DOCUMENTO

```yaml
---
data: [YYYY-MM-DD]
sessao_id: [identificador curto]
agente: Claude | Codex | outro
papel: orquestrador | executor | auditor | novo-agente
versao_protocolo: "5.0"
modo_operacao: nominal | fallback | degradado | suspenso
executor_designado: [nome]
executor_efetivo: [nome — pode ser diferente do designado]
---

## ARTEFATOS CONSULTADOS

- AGENTS.md (v3.0)
- rag/index.md (v8.0)
- [lista de módulos carregados]
- [snapshots consultados]
- [arquivos de produto lidos]

## HIPÓTESES FEITAS

- [hipótese 1 e sua base]
- [hipótese 2 e sua base]

## AMBIGUIDADES ENCONTRADAS

- [ambiguidade 1: descrição + como foi tratada]
- [ambiguidade 2]

## MÓDULOS COM SUSPEITA DE STALENESS

- [módulo ou snapshot + motivo da suspeita]

## CONFIANÇA DA RECONSTRUÇÃO

| Área | Confiança | Justificativa |
|---|---|---|
| Domínio de negócio | Alta/Média/Baixa | [por quê] |
| Estado atual do produto | Alta/Média/Baixa | [por quê] |
| Estado C.A.O.S | Alta/Média/Baixa | [por quê] |

## RISCOS ARQUITETURAIS ATIVOS

Centraliza riscos identificados nesta sessão. Não deixar riscos apenas em "PRÓXIMA SESSÃO"
ou "LIMITAÇÕES" — qualquer risco operacionalmente relevante deve aparecer aqui também.

### Estruturais
- [risco de arquitetura, schema, RLS, Auth + severidade: alta/média/baixa]

### De Produto
- [entrega pendente que é blocker real + impacto + domínio]

### Operacionais
- [gap de protocolo C.A.O.S, staleness, risco de processo + módulo relacionado]

## PROBLEMAS IDENTIFICADOS

- [problema 1 + severidade]
- [problema 2 + severidade]

## MUDANÇAS PROPOSTAS NESTA SESSÃO

- [mudança 1: arquivo + motivo]
- [mudança 2]

## MUDANÇAS REJEITADAS NESTA SESSÃO

- [mudança rejeitada 1 + motivo da rejeição]

## CICLOS EXECUTADOS

- [ciclo_id ou descrição + resultado]

## DIVERGÊNCIAS PERCEBIDAS

- [divergência entre artefato e realidade observada]

## LIMITAÇÕES DO C.A.O.S PERCEBIDAS NESTA SESSÃO

- [limitação 1]
- [limitação 2]

## PONTOS FORTES DO C.A.O.S OBSERVADOS

- [ponto forte 1]

## PRÓXIMA SESSÃO — CONTEXTO RECOMENDADO

[O que o próximo agente deve saber que não está nos artefatos persistentes]
```

---

## MÉTRICAS DE CONTINUIDADE (v5.0)

A partir da Fase 5, incluir bloco de métricas ao final da telemetria:

```yaml
metricas_continuidade:
  tempo_retomada_estimado_min: [número]
  cobertura_snapshots_pct: [número]
  dias_desde_ultima_telemetria: [número]
  modulos_carregados_nesta_sessao: [lista]
  dominios_sem_snapshot_operados: [lista]
  snapshots_criados_na_sessao: [lista de IDs]
  snapshots_historicos_ativos: [lista de IDs]
  nivel_de_continuidade: Pleno | Completo | Estruturado | Basico
  confianca_de_retomada: [ex: 85%]
  contratos_violados: [lista ou vazio]
  locks_verificados: vazio | [lista]
  proxima_entrega_prioritaria: [entrega + domínio] | indefinida
  fase_atual_produto: [Fase N (status)] | desconhecida
```

`fase_atual_produto`: preserva contexto do produto em sessões de auditoria/infra.
`locks_verificados`: vazio = confirmação explícita de rag/locks/ limpo ao encerrar.
`proxima_entrega_prioritaria`: retomada orientada sem inferência. Ausente → indefinida.

Níveis (referência r-continuidade-cognitiva):
- Pleno: todos os módulos + Git + snapshots
- Completo: módulos v3.0+ + snapshots (sem verificação Git)
- Estruturado: núcleo mínimo + snapshots (sem matching formal)
- Básico: apenas AGENTS.md (sem continuidade formal)

---

## CAMPOS OBRIGATÓRIOS

Os seguintes campos são obrigatórios em toda telemetria:

- `data`
- `agente`
- `papel`
- `ARTEFATOS CONSULTADOS` (pelo menos AGENTS.md e index.md)
- `CONFIANÇA DA RECONSTRUÇÃO` (pelo menos domínio + estado produto)
- `RISCOS ARQUITETURAIS ATIVOS` (mesmo que vazio por categoria — declarar explicitamente)
- `MUDANÇAS PROPOSTAS NESTA SESSÃO`
- `MÉTRICAS DE CONTINUIDADE` (a partir da Fase 5)
  - inclui obrigatoriamente: `locks_verificados`, `proxima_entrega_prioritaria`, `fase_atual_produto`

Os demais campos são incluídos quando relevantes.

---

## CAMPO ESPECIAL — PRÓXIMA SESSÃO

Este campo é o mais valioso para continuidade cognitiva.

Deve conter informação que:
- não está em nenhum artefato persistente
- ajudaria o próximo agente a começar sem repetir erros
- contextualiza decisões implícitas tomadas nesta sessão

Exemplos:
```
"O schema_completo.sql usa is_admin() legado — não usar como referência.
 O schema atual no banco é multi-tenant conforme as migrations de Fase 2."

"O módulo k-proj-identidade lista 'próximas entregas' que parecem já implementadas.
 Verificar App.jsx ou migrações SQL antes de usar o módulo como referência."
```

---

## NÃO COMMITAR AUTOMATICAMENTE

A telemetria NÃO é commitada automaticamente.
É commitada quando o ciclo completo da sessão é registrado
como parte do snapshot institucional.

Para sessões de auditoria ou onboarding (sem ciclo de produto),
a telemetria pode ser commitada separadamente:

```
[docs](sistema): telemetria de sessão [data] — [agente]
```

---

## LIMITES

- Máximo 1 arquivo de telemetria por sessão
- Máximo 2 páginas de conteúdo (~100 linhas)
- Sem gráficos, diagramas ou relatórios complexos
- Linguagem direta — não é relatório executivo
