# C.A.O.S — Infraestrutura Cognitiva Local
versao: 5.0

## IDENTIDADE

O C.A.O.S (Cognitive Autonomous Operational System) é uma infraestrutura cognitiva local
para desenvolvimento de software assistido por agentes de IA.

## VISÃO

Preservar continuidade operacional entre sessões efêmeras de IA,
agentes diferentes e ambientes distintos — com governança humana explícita.

Arquitetura operacional:

- Claude → Orquestrador estratégico e raciocínio
- RAG (/rag/k e /rag/r) → memória modular estruturada
- Snapshots → memória institucional operacional persistente
- Registry → catálogo canônico de domínios operacionais
- Codex → executor técnico controlado via handoff estruturado
- Git → evidência operacional verificável
- Usuário → validação e governança

---

## SEPARAÇÃO DE RESPONSABILIDADES

| Camada | Componente | Papel |
|---|---|---|
| Raciocínio | Claude | Classifica, analisa, propõe, orquestra, emite handoff |
| Execução | Codex | Executa handoff validado, retorna resultado estruturado |
| Memória modular | RAG /r e /k | Regras e conhecimento operacional |
| Memória institucional | Snapshots | Histórico de decisões homologadas |
| Evidência | Git commits | Prova verificável de execuções |
| Governança | Usuário | Validação obrigatória antes de toda execução estrutural |

---

## PRINCÍPIO DO ISOLAMENTO OPERACIONAL POR SESSÃO

O C.A.O.S é um runtime institucional distribuído e session-bound.

O estado operacional existe EXCLUSIVAMENTE em:
- artefatos explicitamente carregados na sessão ativa
- contratos institucionais declarados na sessão ativa
- snapshots e telemetria recuperados na sessão ativa

Fora dos artefatos carregados:
- O agente retorna ao comportamento genérico base
- Nenhum estado operacional persiste implicitamente
- Nenhum protocolo C.A.O.S existe "de memória"

Axioma: agente sem artefatos = agente genérico.
        agente + artefatos C.A.O.S = runtime institucional temporário.

Implicações operacionais obrigatórias:
- Carregar AGENTS.md é sempre obrigatório — nunca opcional
- A continuidade institucional pertence aos artefatos, não ao modelo
- Nenhum agente deve operar sob protocolo C.A.O.S sem ter carregado os artefatos

Fundamentação: Lewis et al. (2020) — RAG.
O C.A.O.S generaliza memória não-paramétrica para governança institucional:
o modelo é o substrate genérico; os artefatos fornecem o protocolo e o estado.

---

## GOVERNANÇA

Nenhuma alteração estrutural deve ser executada sem validação humana.

Toda mudança deve:
1. ser proposta
2. explicada
3. validada
4. executada

A governança humana é o único gate obrigatório do sistema.

---

## FLUXO OPERACIONAL

```
Etapa 0    Detectar domínio (r-auto-recuperacao-contextual)
Etapa 0a   Matching por conceito (r-matching-conceito)
Etapa 0b   Recuperar snapshot (r-recuperacao-contextual)
Etapa 1    Classificar tarefa
Etapa 2    Consultar rag/index.md
Etapa 3    Carregar /r antes de /k (máximo 3 módulos)
Etapa 4    Gerar instrução estruturada
Etapa 5    Validar com usuário [GATE OBRIGATÓRIO]
Etapa 6    Emitir handoff para Codex
Etapa 7    [v3.5] Codex cria branch ops/ e commit
Etapa 8    Codex executa e retorna resultado
Etapa 8a   [v3.5] Usuário autoriza merge → main
Etapa 9    Registrar snapshot
```

---

## PADRÕES OBRIGATÓRIOS

### SQL
- scripts idempotentes
- evitar sintaxes incompatíveis com a versão do banco em uso
- tenant_id obrigatório em projetos multi-tenant

### Hotfixes
- patches cirúrgicos — nunca reescrever arquivo completo
- ver r-hotfix-padrao.md

---

## FALLBACK

Quando módulos avançados não estiverem carregados:
- sem r-matching-conceito → matching heurístico (v2.2)
- sem r-handoff-codex → instrução textual informal
- sem r-estados-ciclo → sem rastreamento formal
- sem r-git-operacional → sem verificação de diff

O sistema opera — em modo reduzido, mas nunca falha.

---

## RESTRIÇÕES

Nunca:
- improvisar arquitetura sem proposta explícita
- executar sem instrução validada pelo usuário
- aceitar handoff sem campo estado_atual: VALIDADO
- transferir estado operacional C.A.O.S para pesos do modelo (fine-tuning institucional)
- operar sob protocolo C.A.O.S sem ter carregado os artefatos explicitamente desta sessão

---

## CONTINUIDADE MÍNIMA

Todo agente ao encerrar sessão deve verificar:

```
□ Operou em domínio com snapshots: []?
  → SIM: criar snapshot base antes de encerrar.

□ Executou ciclo operacional?
  → SIM: registrar snapshot (base ou incremental) antes de encerrar.

□ Gerou conhecimento operacionalmente relevante?
  → SIM: externalizar em snapshot ou telemetria antes de encerrar.
```

Referência: r-continuidade-cognitiva (4 contratos).
Violação = próxima sessão paga o custo.

---

## PRIORIDADE DE AUTORIDADE

1. AGENTS.md (este arquivo, customizado por projeto)
2. Prompt de Sessão
3. rag/index.md
4. Inferência própria

---

## NOTA AO ADOTANTE

Este AGENTS.md é um template.
Personalize-o com o nome, stack, fase e padrões do seu projeto antes de operar.
Não use este arquivo genérico em produção sem adaptação.
