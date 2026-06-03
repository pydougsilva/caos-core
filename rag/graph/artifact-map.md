# Mapa Arquitetural — C.A.O.S
versao: 1.0
data: 2026-05-31
tipo: grafo_navegacao
camada: derivada
fallback: artefatos individuais são sempre a fonte de verdade

## OBJETIVO

Mapa semântico das relações arquiteturais do C.A.O.S.
Serve como orientação de navegação para novos agentes — especialmente
na etapa de retomada, para identificar rapidamente qual artefato responde o quê.

Este documento não substitui nenhum artefato.
É uma vista de cima do sistema para quem acabou de chegar.

---

## CAMADAS E RESPONSABILIDADES

```
AGENTS.md ─────────────── constituição
  │  define: papéis, fluxo, restrições, axioma de isolamento
  │  referenciado por: adapters, context_receipt, handoff, todos os módulos
  │  autoridade: máxima — nenhum artefato o substitui
  │
  ├─ context_receipt ──── prova de bootstrap
  │    emitido por agente ao iniciar sessão com adapter ativo
  │    registrado em: telemetria de sessão
  │    campos: adapter_usado, agents_md_carregado, modo_operacao
  │
  ├─ handoff VALIDADO ─── contrato de execução
  │    exige: estado_atual = VALIDADO (gate humano)
  │    formato: k-sys-handoff-format
  │    protocolo: r-handoff-executor
  │    retorno esperado: CONCLUÍDO | FALHOU | HANDOFF_INVALIDO
  │
  ├─ snapshot ─────────── memória de domínio
  │    estado capturado após ciclo homologado
  │    hierarquia de carga: último sucesso > riscos ativos > última falha > base
  │    protocolo: r-recuperacao-contextual
  │    indexado por: k-sys-registry-dominios
  │
  ├─ telemetria ────────── contexto de sessão
  │    registro operacional de cada sessão
  │    contém: context_receipt, módulos carregados, ciclos executados, riscos
  │    alimenta: resumption-index (derivação manual ou assistida)
  │
  ├─ RAG /r ────────────── regras operacionais
  │    como o agente deve agir
  │    carregado ANTES de /k (sempre)
  │    máximo 3 módulos por ciclo
  │
  ├─ RAG /k ────────────── conhecimento do projeto
  │    o que o agente precisa saber
  │    carregado APÓS /r (sempre)
  │    máximo 3 módulos por ciclo
  │
  └─ rag/graph/ ────────── navegação e recuperação (esta camada)
       artifact-map.md: este documento — mapa de alto nível
       resumption-index.yaml: caminho mínimo validado por domínio
       NÃO substitui nenhuma camada acima
       fallback: se ausente → comportamento histórico (Etapa 0b via r-recuperacao-contextual)
```

---

## FLUXO DE RETOMADA — COM CAMADA DE NAVEGAÇÃO

```
Etapa 0   Detectar domínio
          r-auto-recuperacao-contextual

Etapa 0a  Matching por conceito
          r-matching-conceito + k-sys-registry-dominios
          → ÚNICO: domínio identificado
          → AMBÍGUO: aguardar usuário
          → SEM MATCH: fallback v2.2

Etapa 0b  Consultar caminho mínimo [NOVO]
          rag/graph/resumption-index.yaml
          → condição: entrada existe E confianca >= média E origem em ciclo CONCLUÍDO
          → válido: carregar caminho_minimo + snapshot_minimo → pular Etapa 0c
          → snapshot_minimo null: carregar caminho_minimo + Etapa 0c (busca normal)
          → inválido ou ausente: Etapa 0c (comportamento histórico)

Etapa 0c  Recuperar snapshot (se 0b não encontrou caminho)
          r-recuperacao-contextual

Etapa 1-3 Carregar módulos
          /r antes de /k, máximo 3

Etapas 4+ Protocolo normal (AGENTS.md)
```

Ganho: para domínios com entrada no resumption-index, a Etapa 0c é pulada.
O agente carrega o caminho mínimo já validado — sem precisar explorar telemetria.

---

## RELAÇÕES QUE ESTE MAPA NÃO COBRE

As relações abaixo existem no sistema mas não estão neste mapa:
- Dependências entre módulos /r e /k entre si (planejado para v6.0)
- Drift entre versões caos-core ↔ instância de produto (planejado para v6.0)
- Grafo semântico de módulos via SBERT (planejado para v6.0)

Até que v6.0 seja implementado, essas relações vivem nos próprios artefatos.

---

## MANUTENÇÃO

- `artifact-map.md`: atualizar apenas quando a arquitetura mudar (evento raro)
- `resumption-index.yaml`: atualizar após ciclos que validem novo caminho mínimo
- Ambos são artefatos DERIVADOS — nunca são fonte de verdade primária
