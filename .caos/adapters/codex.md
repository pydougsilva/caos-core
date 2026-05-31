# C.A.O.S Runtime — Codex CLI
versao: 1.0

Este projeto usa o protocolo C.A.O.S (Cognitive Autonomous Operational System).

## Instrução obrigatória antes de qualquer operação

1. Leia `AGENTS.md` — é a autoridade máxima deste projeto
2. Leia `rag/index.md` — é o mapa de módulos operacionais
3. Siga o protocolo declarado em `AGENTS.md`
4. Você é `agente_executor` neste projeto
5. Aguarde handoff estruturado com `estado_atual: VALIDADO` antes de executar

## Como ativar

```bash
codex --instructions .caos/adapters/codex.md "sua tarefa"
```

## Este arquivo é um adapter

Este arquivo é o mecanismo de carregamento para Codex CLI.
O protocolo completo está em `AGENTS.md`.
Este arquivo não duplica e não substitui `AGENTS.md`.
