---
id: dependencia-nominal-residual
tipo: auditoria
estado: registrado
data: 2026-05-25
escopo: hardening institucional definitivo
---

# Dependencia Nominal Residual

## Objetivo

Auditar referencias nominais de ferramenta apos a institucionalizacao dos papeis
`agente_orquestrador` e `agente_executor`.

## Regra Aplicada

Quando a referencia representava contrato ativo, foi substituida por papel
institucional.

Quando a referencia representava evidencia historica, foi preservada.

## Vestigios Preservados Como Historia

- `rag/docs/validacoes/T2.1-rejeicao-handoff-invalido.md`
- `rag/docs/validacoes/T2.2-aceitacao-handoff-valido.md`
- `rag/docs/validacoes/T0.1-retomada-fria.md`
- `rag/docs/validacoes/T0.1B-retomada-pos-ajustes.md`
- snapshots historicos em `rag/r/r-recuperacao-contextual.md`

Esses vestigios registram ciclos reais e nao devem ser reescritos
retroativamente.

## Hardcodes Estruturais Eliminados

- `AGENTS.md`: responsabilidades nominais convertidas em papeis.
- `rag/index.md`: handoff e matriz de responsabilidades convertidos em papeis.
- `r-handoff-codex.md`: renomeado para `r-handoff-executor.md`.
- `CODEX-BOOTSTRAP.md`: renomeado para `AGENTE-EXECUTOR-BOOTSTRAP.md`.
- Governanca Git: metadados passam a registrar ids concretos de papeis, nao
  fornecedores fixos.
- Templates: snapshots e telemetria passam a aceitar identidades concretas em
  campos de papel.

## Modo Degradado

modo_degradado_validado: SIM

Documentacao validada:

- `rag/r/r-executor-contingencia.md`
- `rag/r/r-restauracao-orquestrador.md`
- snapshots historicos `pedidos-004` e `pedidos-005`

Contrato atual:

- `modo_operacao: degradado`
- mesma identidade concreta exerce `agente_orquestrador` e `agente_executor`
- telemetria obrigatoria
- gate humano reforcado
- revisao de diff antes de merge
- limite de 5 ciclos consecutivos antes de auditoria de rastreabilidade

Lacuna corrigida neste ciclo:

- o modo degradado deixou de depender de uma ferramenta nominal especifica e
  passou a representar acumulacao temporaria de papeis.

## Riscos Remanescentes

- Documentos historicos ainda contem nomes de ferramentas por fidelidade
  institucional.
- Commits antigos mantem trailers nominais e nao devem ser alterados.
- Ferramentas externas ou prompts fora do repositorio podem ainda usar nomes
  antigos sem que o C.A.O.S consiga auditar automaticamente.

## Resultado

dependencia_nominal_residual: NAO em contratos ativos.

dependencia_nominal_historica: SIM, preservada como evidencia.
