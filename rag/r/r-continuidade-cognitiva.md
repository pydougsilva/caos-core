# r-continuidade-cognitiva
versao: 1.0

## OBJETIVO

Formalizar como as capacidades do C.A.O.S se conectam para garantir
continuidade operacional entre sessões, agentes e projetos.

Responde à pergunta:
"o sistema tem continuidade suficiente para ser retomado sem reconstrução manual?"

---

## PRINCÍPIO CENTRAL

Continuidade não é memória.
Continuidade é a capacidade de retomar operação a partir do estado atual
sem depender de quem operou antes.

---

## OS QUATRO CONTRATOS DE CONTINUIDADE

### Contrato 1 — Toda sessão deixa rastro

Ao encerrar qualquer sessão com ciclo operacional:
- snapshot atualizado (se houve execução)
- telemetria registrada (sempre)
- locks removidos (se ciclo CONCLUÍDO)

Se qualquer dos três estiver faltando, o contrato foi violado.
A próxima sessão pagará o custo dessa violação.

### Contrato 2 — Todo domínio operado tem snapshot

Primeira operação em domínio sem histórico:
→ OBRIGATÓRIO registrar snapshot antes de encerrar a sessão.

O snapshot de primeiro ciclo pode ser simples:
- o que foi observado
- o que foi decidido ou proposto
- o estado atual do domínio
- riscos identificados

Não precisa ser perfeito. Precisa existir.

### Contrato 3 — Contexto crítico vive nos artefatos

Se um agente sabe algo operacionalmente relevante que não está em nenhum artefato,
essa informação não existe institucionalmente.

Regra: toda descoberta relevante deve ser externalizada
(em snapshot, telemetria, ou atualização de módulo) antes de encerrar a sessão.

### Contrato 4 — Retomada em menos de 15 minutos

O sistema está funcionando se um novo agente consegue:
- entender o projeto
- identificar o estado atual
- saber onde continuar

em menos de 15 minutos lendo os artefatos.

Se não consegue: o sistema tem lacuna de continuidade a corrigir.

---

## VERIFICAÇÃO DE CONTINUIDADE

Aplicar ao iniciar qualquer sessão em projeto existente:

```
□ AGENTS.md presente e personalizado (não template)?
□ Última telemetria < 7 dias?
□ rag/locks/ vazio (sem ciclo interrompido)?
□ Cobertura de snapshots > 80% dos domínios operados?
□ index.md sem referências fantasma?
□ Tempo estimado de retomada < 15 min?
```

Se qualquer item falhar: tratar antes de iniciar novos ciclos.

---

## NÍVEIS DE CONTINUIDADE

O sistema opera em quatro níveis.
Nível 4 é degradação grave — nunca deve ser o estado normal.

| Nível | Nome | Capacidade |
|---|---|---|
| 1 | Pleno | todos os módulos + Git institucional + snapshots |
| 2 | Completo | módulos v3.0+ + snapshots (sem verificação Git) |
| 3 | Estruturado | núcleo mínimo + snapshots (sem matching formal) |
| 4 | Básico | apenas AGENTS.md (sem continuidade formal) |

Ao identificar o nível atual, declarar explicitamente ao usuário antes de operar.

---

## PROTOCOLO DE PRIMEIRO SNAPSHOT

Quando operar em domínio sem histórico (`snapshots: []` no registry):

```
Ao encerrar a sessão:

1. Verificar: este domínio tinha snapshots antes desta sessão?
   → SIM: atualizar o snapshot existente
   → NÃO: criar snapshot base com conteúdo mínimo abaixo

Conteúdo mínimo do snapshot de primeiro ciclo:
  id: [domínio]-001
  tipo: base
  dominio: [canônico]
  data: [hoje]
  tarefa: [tipo da operação realizada]
  decisao: |
    [o que foi observado, analisado ou proposto]
    [mesmo que não tenha sido executado ainda]
  resultado: [sucesso | falha | pendente | auditoria]
  riscos_vistos: [lista — pode ser vazia se nenhum identificado]
  riscos_ativos: [lista dos não resolvidos]
  estado_atual: [do domínio — estável | degradado | desconhecido | etc.]
```

---

## PROTOCOLO DE PROMOÇÃO DE MÓDULO

Quando um módulo específico de projeto deve migrar para caos-core:

```
Critérios (todos devem ser verdadeiros):
  □ Usado em ≥ 2 projetos distintos (com adaptação mínima)
  □ Responde pergunta que qualquer projeto pode ter
  □ Sem referências a domínios, tecnologias ou negócios específicos
  □ Validado em ≥ 5 ciclos reais
  □ Não duplica módulo já existente no core

Processo:
  1. Identificar o módulo candidato
  2. Generalizar — remover especificidades
  3. Comparar com index.md do caos-core — há sobreposição?
  4. Propor ao usuário [gate humano]
  5. Criar PR no caos-core se aprovado
  6. Manter cópia local no projeto original
```

---

## MÉTRICAS DE SAÚDE DE CONTINUIDADE

A partir da Fase 5, a telemetria deve incluir estas métricas:

```yaml
metricas_continuidade:
  tempo_retomada_estimado_min: [minutos estimados para novo agente operar]
  cobertura_snapshots_pct: [domínios com snapshot / total de domínios operados]
  dias_desde_ultima_telemetria: [número]
  modulos_carregados_nesta_sessao: [lista]  # rastreia uso efetivo
  dominios_sem_snapshot_operados: [lista]   # contrato 2
  contratos_violados: [lista]               # transparência institucional
```

---

## INTEGRAÇÃO COM MÓDULOS EXISTENTES

| Este módulo... | ...complementa |
|---|---|
| r-continuidade-cognitiva | r-recuperacao-contextual (como recuperar) |
| r-continuidade-cognitiva | r-telemetria-cognitiva (o que registrar) |
| r-continuidade-cognitiva | r-staleness-detection (quando verificar) |
| r-continuidade-cognitiva | k-sys-handoff-institucional (protocolo de entrada) |

---

## PROIBIÇÕES

Nunca:
- Encerrar sessão com ciclo executado sem snapshot correspondente
- Operar em domínio sem snapshot sem registrar um ao final
- Declarar "continuidade garantida" sem verificar os quatro contratos
- Deixar conhecimento operacionalmente relevante apenas na memória da sessão
- Promover módulo para core sem gate humano

---

## RESULTADO ESPERADO

O sistema tem continuidade cognitiva operacional quando:
- Qualquer agente consegue retomar em < 15 minutos
- Todo domínio operado tem pelo menos 1 snapshot
- A última sessão deixou telemetria
- Nenhum ciclo ativo está em estado EXECUTANDO sem retorno
