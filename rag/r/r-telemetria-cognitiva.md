# r-telemetria-cognitiva
versao: 1.0

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
versao_protocolo: 3.5
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

## CAMPOS OBRIGATÓRIOS

Os seguintes campos são obrigatórios em toda telemetria:

- `data`
- `agente`
- `papel`
- `ARTEFATOS CONSULTADOS` (pelo menos AGENTS.md e index.md)
- `CONFIANÇA DA RECONSTRUÇÃO` (pelo menos domínio + estado produto)
- `MUDANÇAS PROPOSTAS NESTA SESSÃO`

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

"App.jsx tem duas versões do checkout: uma para móvel (linha ~800) e
 uma para desktop (linha ~1200). Sempre verificar ambas antes de hotfix."
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
