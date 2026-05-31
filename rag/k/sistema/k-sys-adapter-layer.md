# k-sys-adapter-layer
versao: 1.0
data: 2026-05-31
origem: decisao A1.1-codex-revisao-decisao-final (2026-05-31)

## OBJETIVO

Descrever a Adapter Layer do C.A.O.S:
mecanismo que garante que qualquer ferramenta de IA
carregue o protocolo canônico (`AGENTS.md`) antes de operar.

Responde à pergunta:
"como garantir que qualquer agente prove que carregou o protocolo C.A.O.S
independentemente da ferramenta utilizada?"

---

## PROBLEMA QUE RESOLVE

O C.A.O.S v5.0 formalizou o axioma:
> agente sem artefatos = agente genérico.

O axioma pressupõe que os artefatos foram carregados.
Mas ferramentas diferentes têm mecanismos diferentes de inicialização.
Sem um mecanismo explícito, o agente pode operar sem `AGENTS.md`.

A Adapter Layer resolve isso:
cada ferramenta tem um arquivo de inicialização (adapter)
que instrui o carregamento de `AGENTS.md` antes de qualquer operação.

Este é o conceito de **Bootstrap Institucional Verificável**:
não "tornar agentes indiferentes", mas garantir que o agente
*prova* ter carregado o protocolo antes de operar.

---

## ESTRUTURA

```
[raiz do projeto]/
├── CLAUDE.md                    ← adapter para Claude Code (carregado automaticamente)
├── .caos/
│   ├── adapters/
│   │   ├── claude.md            ← documentação do adapter Claude
│   │   └── codex.md             ← adapter + instrução para Codex CLI
│   └── receipts/                ← context_receipts de sessão (efêmeros, em .gitignore)
└── AGENTS.md                    ← autoridade máxima (inalterada)
```

---

## PRINCÍPIOS DO ADAPTER

### O adapter É
- Ponteiro para `AGENTS.md`
- Instrução mínima de bootstrap
- Mecanismo de compatibilidade técnica específico por ferramenta

### O adapter NÃO É
- O protocolo C.A.O.S — isso é `AGENTS.md`
- Autoridade sobre operações
- Duplicata de `AGENTS.md`
- Requisito estrutural — o sistema funciona sem adapters (carregamento manual)

### Regra fundamental (P14)
Um adapter nunca usa o nome da ferramenta como requisito institucional.
`.caos/adapters/claude.md` é um mecanismo técnico — não um contrato nominal.
Se Claude Code for substituído por outra ferramenta, o adapter muda.
`AGENTS.md` permanece.

---

## ADAPTERS DISPONÍVEIS

### CLAUDE.md — Claude Code

Claude Code lê `CLAUDE.md` na raiz do projeto como instruções persistentes.
`CLAUDE.md` instrui: "leia `AGENTS.md` antes de operar."

```
Localização: CLAUDE.md (raiz do projeto)
Conteúdo: instrução de bootstrap + referência a AGENTS.md
Ativação: automática (Claude Code carrega CLAUDE.md por padrão)
Documentação: .caos/adapters/claude.md
```

### .caos/adapters/codex.md — Codex CLI

Codex CLI aceita `--instructions [arquivo]` como ponto de entrada.

```bash
codex --instructions .caos/adapters/codex.md "sua tarefa"
```

```
Localização: .caos/adapters/codex.md
Conteúdo: instrução de bootstrap + referência a AGENTS.md + como ativar
Ativação: explícita via flag --instructions
```

---

## COMO CRIAR ADAPTER PARA NOVA FERRAMENTA

```
1. Entender como a ferramenta aceita instruções persistentes
2. Criar .caos/adapters/[ferramenta].md
3. Conteúdo obrigatório:
   - instrução para ler AGENTS.md
   - instrução para ler rag/index.md
   - declaração explícita de que o adapter não é o protocolo
4. Conteúdo proibido:
   - cópia do protocolo C.A.O.S
   - regras operacionais
   - qualquer conteúdo que deveria estar em AGENTS.md
5. Se a ferramenta usa arquivo na raiz (como CLAUDE.md):
   - criar o arquivo na raiz
   - documentar em .caos/adapters/[ferramenta].md
6. Atualizar este documento com o novo adapter
7. Commit: [rag](sistema): adiciona adapter [ferramenta] v1.0
```

---

## context_receipt — RECIBO DE BOOTSTRAP

Ao iniciar uma sessão com um adapter ativo, o agente emite um `context_receipt`
na telemetria da sessão como prova de que o bootstrap ocorreu.

```yaml
context_receipt:
  agente: [identidade concreta — ex: Claude Sonnet 4.6]
  timestamp: [ISO 8601]
  adapter_usado: CLAUDE.md | codex | manual | nenhum
  agents_md_carregado: sim | nao
  agents_md_versao: [versao declarada em AGENTS.md]
  index_md_carregado: sim | nao
  nucleo_minimo_verificado: sim | nao
  modo_operacao: pleno | degradado | suspenso
  ciclo_ativo: [ciclo_id ou null]
```

O `context_receipt` é a evidência verificável do bootstrap.
Sessões sem `context_receipt` podem não ter carregado o protocolo.

---

## .gitignore

```
.caos/receipts/     # context_receipts são efêmeros — não rastrear
```

`.caos/adapters/` É rastreado — adapters são artefatos institucionais.
`.caos/receipts/` NÃO É rastreado — receipts são estado efêmero de sessão.

---

## RISCOS E MITIGAÇÕES

| Risco | Severidade | Mitigação |
|---|---|---|
| Adapter duplica AGENTS.md | Alta | Proibido por design — adapter só instrui o carregamento |
| Ferramenta ignora o adapter | Baixa | Fallback: carregamento manual de AGENTS.md (histórico) |
| CLAUDE.md usado como protocolo | Alta | CLAUDE.md declara explicitamente: "não é o protocolo" |
| Nova ferramenta sem adapter | Baixa | Sistema funciona sem adapter — adapter é opcional |
| context_receipt não emitido | Baixa | Sessão ainda válida — apenas sem evidência de bootstrap |

---

## RELAÇÃO COM OUTROS MÓDULOS

- `AGENTS.md`: o protocolo que os adapters referenciam — nunca substituído
- `k-sys-principios-fundamentais`: P14 governa o design dos adapters
- `A1.1-codex-revisao-decisao-final`: decisão institucional que aprovou esta camada
- `r-continuidade-cognitiva`: context_receipt reforça o contrato de continuidade
- `rag/templates/telemetria-template.md`: inclui seção context_receipt
