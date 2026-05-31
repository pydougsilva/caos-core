# r-executor-contingencia
versao: 1.0

## OBJETIVO

Definir o protocolo de contingência operacional do C.A.O.S
quando o executor nominal está indisponível.

Responde à pergunta:
"como o sistema opera quando o executor designado não está disponível?"

Referência: Lewis et al. (2020) — Retrieval-Augmented Generation.
O executor, como o modelo base, é substrate genérico.
O protocolo C.A.O.S vive nos artefatos — não no executor.
Qualquer agente com as capacidades requeridas pode ser executor.

---

## OS QUATRO MODOS DE EXECUTOR

### Nominal

Executor designado disponível e operando.
Declaração: modo_operacao: nominal | executor_designado: [nome]
Governança: padrão — handoff + checklist + retorno estruturado + gate humano.

---

### Fallback

Executor alternativo (outro LLM capaz) em lugar do nominal.
Requer validação institucional prévia antes de ciclos reais de produto.

Declaração obrigatória em todo commit, snapshot e telemetria:
  modo_operacao: fallback
  executor_designado: [nome]
  executor_nominal_ausente: [nome do nominal]
  motivo: [razão da indisponibilidade]

---

### Degradado

Orquestrador (Claude) acumula execução.

Declaração obrigatória:
  modo_operacao: degradado
  executor_temporario: Claude
  executor_nominal_ausente: [nome]
  motivo: [razão]

Governança adicional:
- Audit trail explícito por ciclo (decisão / execução / auditoria separados)
- Gate humano reforçado — humano revisa diff antes do merge
- Limite: máximo 5 ciclos consecutivos antes de revisão de rastreabilidade

---

### Suspenso

Nenhum executor disponível. Apenas ciclos de análise e proposta.
Nenhum ciclo de execução deve ser iniciado.

---

## PRÉ-REQUISITOS NÍVEL 0

Verificar ANTES de enviar handoff a qualquer executor:

□ Executor é invocável de forma confiável pelo operador?
□ SLA minimamente previsível (sem bloqueios opacos mid-cycle)?
□ Ambiente operacional disponível com as ferramentas necessárias?
□ Capacidades requeridas confirmadas (file-edit, git-commit, MCP se necessário)?
□ Risco comercial conhecido e aceitável?

Se qualquer item falhar: não enviar handoff. Declarar modo correspondente.

---

## TAXONOMIA DE EXECUTORES

Compatibilidade arquitetural:
  verificada    → completou ciclo real e seguiu protocolo corretamente
  parcial       → completou com desvios identificados e documentados
  conceitual    → compatível na teoria, não verificado empiricamente
  incompatível  → não consegue seguir o protocolo estruturalmente

Confiabilidade operacional:
  alta     → SLA formal, limites previsíveis, sem bloqueios externos
  média    → plano pago com limites conhecidos
  baixa    → plano gratuito, limites opacos
  inviável → bloqueios que impedem ciclos institucionais completos

Maturidade institucional:
  validado  → ciclos reais com rastreabilidade completa
  parcial   → parte do protocolo executada
  tentado   → teste iniciado, não concluído
  untested  → nenhuma tentativa feita

---

## REGISTRO DE EXECUTORES

### Codex (executor de referência)

```
compatibilidade_arquitetural: verificada
confiabilidade_operacional: média
maturidade_institucional: validado
status: executor de referência do C.A.O.S
notas: executor validado em múltiplos ciclos reais com rastreabilidade completa
```

### Gemini Code Assist (Agent Mode, free plan)

```
compatibilidade_arquitetural: conceitual-nao-verificada
confiabilidade_operacional: inviavel
maturidade_institucional: tentado
teste: T-GEM.0 (2026-05-17) — nao concluido (bloqueios de plano gratuito)
status: indisponivel operacionalmente
recomendacao: reavaliar com plano pago + MCP configurado
notas: incompatibilidade foi operacional, nao arquitetural
```

---

## LIMITES DO MODO DEGRADADO

| Limite | Valor |
|---|---|
| Máximo de ciclos consecutivos | 5 |
| Após 5 ciclos | revisão de rastreabilidade obrigatória |
| Após 10 ciclos | pausa até executor nominal disponível |

---

## RETORNO AO MODO NOMINAL

Quando o executor nominal retorna:

1. Primeiro ciclo: infraestrutura institucional (não produto)
2. Auditar commits em modo degradado/fallback — verificar invariantes
3. Verificar rastreabilidade completa dos ciclos em contingência
4. Declarar explicitamente nos artefatos: modo_operacao: nominal restabelecido

---

## INTEGRAÇÃO COM MÓDULOS EXISTENTES

| Este módulo | complementa |
|---|---|
| r-executor-contingencia | r-handoff-codex (protocolo de execução) |
| r-executor-contingencia | r-continuidade-cognitiva (contratos) |
| r-executor-contingencia | r-orquestracao-caos (responsabilidades) |
| r-executor-contingencia | AGENTS.md (axioma de isolamento operacional) |
