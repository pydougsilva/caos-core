---
data: YYYY-MM-DD
sessao_id: [identificador curto — ex: hotfix-auth-001]
agente_orquestrador: [identidade concreta — ex: Claude Sonnet 4.6]
agente_executor: [identidade concreta — ex: Codex | Claude Sonnet 4.6 (modo degradado)]
papel: orquestrador | executor | orquestrador + executor
versao_protocolo: "5.0"
modo_operacao: nominal | degradado | suspenso
ciclo_id: [ciclo-id-YYYYMMDD]
tipo_ciclo: feature | hotfix | auditoria | documentacao | limpeza | arquitetural
---

## context_receipt

```yaml
context_receipt:
  agente: [identidade concreta]
  timestamp: [ISO 8601]
  adapter_usado: CLAUDE.md | codex | manual | nenhum
  agents_md_carregado: sim | nao
  agents_md_versao: [ex: "3.0"]
  index_md_carregado: sim | nao
  nucleo_minimo_verificado: sim | nao
  modo_operacao: pleno | degradado | suspenso
  ciclo_ativo: [ciclo_id ou null]
```

---

## ARTEFATOS CONSULTADOS

- AGENTS.md (vX.Y)
- rag/index.md (vX.Y)
- [listar módulos /r e /k carregados]
- [listar snapshots ou telemetrias consultadas]

## HIPÓTESES FEITAS

- [hipótese 1 — base para a hipótese]
- [hipótese 2]

## AMBIGUIDADES ENCONTRADAS

- [ambiguidade 1: o que estava ambíguo e como foi tratado]
- Nenhuma.

## MÓDULOS COM SUSPEITA DE STALENESS

- [módulo: motivo]
- Nenhum.

## CONFIANÇA DA RECONSTRUÇÃO

| Área | Confiança | Justificativa |
|---|---|---|
| Domínio de negócio | Alta (X%) | [por quê] |
| Estado atual do produto | Alta (X%) | [por quê] |
| Estado C.A.O.S | Alta (X%) | [por quê] |

## RISCOS ARQUITETURAIS ATIVOS

### Estruturais
- [risco de schema, RLS, Auth + severidade: alta/média/baixa]

### De Produto
- [entrega pendente que é blocker real]

### Operacionais
- [gap de protocolo C.A.O.S, staleness, risco de processo]
- Nenhum.

## PROBLEMAS IDENTIFICADOS

- [problema 1 + severidade: alta/média/baixa]
- Nenhum.

## MUDANÇAS EXECUTADAS NESTA SESSÃO

- [arquivo alterado + motivo]

## MUDANÇAS EXCLUÍDAS DO ESCOPO

- [o que ficou de fora e por quê]

## CICLOS EXECUTADOS

- [ciclo_id]: [CONCLUÍDO | FALHOU | PENDENTE]
  Branch: ops/[nome]
  Commit: [hash]
  Gate humano: [descrição da aprovação]
  Modo: nominal | degradado

## PRÓXIMA SESSÃO — CONTEXTO RECOMENDADO

1. [prioridade 1 — o que deve ser feito primeiro]
2. [prioridade 2]

[O que o próximo agente deve saber que NÃO está nos artefatos persistentes.]

## MÉTRICAS DE CONTINUIDADE

```yaml
metricas_continuidade:
  tempo_retomada_estimado_min: [número]
  cobertura_snapshots_pct: [número]
  dias_desde_ultima_telemetria: [número]
  modulos_carregados_nesta_sessao: [lista]
  dominios_sem_snapshot_operados: [lista ou vazio]
  snapshots_criados_na_sessao: [lista de IDs ou vazio]
  nivel_de_continuidade: Pleno | Estruturado | Base | Frágil
  confianca_de_retomada: X%
  contratos_violados: [lista ou vazio]
  locks_verificados: vazio | [lista]
  ciclos_degradados_consecutivos: [número]
  modo_operacao: nominal | degradado
  ciclo_concluido: [ciclo_id]
  merge_hash: [hash ou null]
  proxima_prioridade: [descrição]
  fase_atual_produto: [ex: Fase 3 — Sprint D concluída]
```

## DECLARAÇÃO DE MODO DEGRADADO (incluir apenas se modo_operacao: degradado)

```yaml
modo_operacao: degradado
agente_orquestrador: [identidade]
agente_executor: [identidade] (acumulacao temporaria de papeis)
motivo: [motivo da acumulação]
restricoes_aplicadas:
  - gate_humano_reforcado: sim
  - diff_revisado: sim
  - telemetria: registrada
ciclos_degradados_consecutivos: [número]
auditoria_requerida_apos: 5 ciclos
```
