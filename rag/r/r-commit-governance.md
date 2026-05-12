# r-commit-governance
versao: 1.0

## OBJETIVO

Definir as regras que Codex deve seguir para criar commits
institucionais válidos no C.A.O.S.

Responde à pergunta:
"como Codex deve produzir um commit institucional correto?"

Para as convenções de nomenclatura e estratégia de branches,
ver k-sys-governanca-git.

---

## PRINCÍPIO CENTRAL

Um commit só é institucional quando rastreável até uma decisão homologada.

Toda execução que altera o sistema deve ser rastreável:
commit → snapshot → decisão → usuário que aprovou.

Se a cadeia for interrompida em qualquer ponto,
o commit não é reconhecido como evento institucional do C.A.O.S.

---

## RESPONSABILIDADE EXCLUSIVA DE CODEX

Apenas Codex cria commits. Claude nunca cria commits.

Claude prepara o contexto do commit (commit_type, branch_sugerido, snapshot_ref)
e o inclui no handoff. Codex recebe essas informações e as usa na criação.

Se Claude tentasse criar commits, violaria a separação
raciocínio ↔ execução que é o princípio central do protocolo de handoff.

---

## SEQUÊNCIA OBRIGATÓRIA

A criação de commit segue sequência estrita. Desviar desta ordem
é violação arquitetural.

```
Passo 1   Receber handoff válido (estado: VALIDADO)
          → commit_type presente e não null
          → branch_sugerido presente e não null
          → snapshot_ref presente

Passo 2   Criar branch ops/ conforme branch_sugerido
          → nunca executar diretamente em main
          → branch nomeada conforme k-sys-governanca-git

Passo 3   Executar instrução do handoff
          → apenas no escopo do domínio declarado
          → somente os artefatos descritos em artefatos_alterados

Passo 4   Verificar correspondência pré-commit
          → arquivos alterados == artefatos_alterados do handoff?
          → se divergência: retornar FALHOU antes de commitar

Passo 5   Criar commit institucional com todos os campos obrigatórios

Passo 6   Retornar resultado para Claude com commit_hash
          → Claude valida (etapa 7b) antes de qualquer merge
          → Codex não faz merge diretamente
```

Codex não executa merge. Merge é gate humano (etapa 8a).

---

## FORMATO OBRIGATÓRIO DO COMMIT

```
[tipo](domínio): descrição em imperativo, presente, sem ponto final

snapshot: ID-do-snapshot
agent-executor: Codex
agent-orchestrator: Claude
risks-addressed: N

Co-Authored-By: Codex <noreply@codex>
Orchestrated-By: Claude <noreply@claude>
```

---

## VALIDAÇÃO DE CAMPOS

Codex valida todos os campos antes de criar o commit.
Se qualquer campo falhar, o commit não é criado — retornar FALHOU.

| Campo | Obrigatório | Regra de validação |
|---|---|---|
| `[tipo]` | sim | valor deve estar na tabela de tipos válidos |
| `(domínio)` | sim | deve ser domínio do registry — não inventar |
| `descrição` | sim | imperativo presente, sem ponto final, ≤ 52 chars |
| linha de título completa | sim | ≤ 72 caracteres |
| `snapshot:` | sim | deve referenciar snapshot existente |
| `agent-executor: Codex` | sim | valor literal "Codex" |
| `agent-orchestrator: Claude` | sim | valor literal "Claude" |
| `risks-addressed:` | sim | inteiro ≥ 0 |
| `Co-Authored-By: Codex <noreply@codex>` | sim | exato |
| `Orchestrated-By: Claude <noreply@claude>` | sim | exato |

### Comprimento da linha de título

```
[tipo](domínio): descrição
└──────────────────────────┘
      máximo 72 caracteres
```

Se a linha ultrapassar 72 caracteres, encurtar a descrição.
Nunca cortar `[tipo]`, `(domínio)` ou `snapshot:`.

---

## TIPOS VÁLIDOS

| Tipo | Usar quando |
|---|---|
| `[hotfix]` | correção em código de aplicação (src/, public/) |
| `[migration]` | alteração de schema SQL (DDL, policies, functions) |
| `[rls]` | alteração exclusiva em Row Level Security |
| `[rag]` | criação ou atualização de módulos RAG (rag/) |
| `[docs]` | criação ou atualização de documentação (rag/docs/) |
| `[snapshot]` | quando um snapshot é criado ou atualizado via commit |
| `[handoff]` | registro de transferência entre agentes |
| `[revert]` | rollback técnico de commit anterior |

Se o tipo do ciclo não se encaixar em nenhuma categoria:
→ usar o tipo mais próximo
→ incluir contexto adicional no corpo da descrição (não na linha de título)
→ nunca inventar tipo fora da tabela

---

## ESCOPO ÚNICO POR COMMIT

Cada commit representa exatamente um ciclo operacional num único domínio.

### O que isso significa

- Um commit por handoff executado
- Um domínio por commit
- Os arquivos alterados no commit são exatamente os listados em `artefatos_alterados`
- Nenhum arquivo adicional — nem "de passagem", nem "por conveniência"

### Multi-domínio — proibido sem aprovação

Se a instrução impactar mais de um domínio:
→ não commitar tudo junto
→ retornar FALHOU com campo `escopo_excedido: true` e listar domínios impactados
→ Claude cria ciclos separados por domínio

Exceção: aprovação explícita de multi-domínio no handoff.
Se handoff.dominio lista múltiplos domínios com aprovação explícita do usuário:
→ commit multi-domínio é permitido — mencionar todos na descrição

---

## RELAÇÃO COMMIT ↔ ARTEFATOS_ALTERADOS

O commit deve espelhar exatamente o campo `artefatos_alterados` do handoff.

### Verificação pré-commit (Passo 4)

```
artefatos_handoff = handoff.artefatos_alterados
artefatos_reais   = git diff --name-only HEAD

se artefatos_reais ⊄ artefatos_handoff:
    arquivos_extras = artefatos_reais - artefatos_handoff
    → retornar FALHOU
    → erro: "artefatos não autorizados: " + arquivos_extras

se artefatos_handoff ⊄ artefatos_reais:
    arquivos_faltando = artefatos_handoff - artefatos_reais
    → avaliar: arquivos não encontrados = instrução não executada completamente
    → retornar FALHOU
    → erro: "artefatos esperados não alterados: " + arquivos_faltando
```

### O que fazer com a divergência

Codex não decide o que fazer com arquivos extras ou faltantes.
Codex retorna FALHOU com detalhes completos.
Claude analisa e reavalia a instrução se necessário.
Novo handoff pode ser emitido após análise.

---

## COMMITS PROIBIDOS

| Commit | Motivo | Consequência |
|---|---|---|
| Sem campo `snapshot:` | execução sem decisão — não rastreável | FALHOU antes de commitar |
| Sem `agent-executor:` | autoria indeterminada | FALHOU antes de commitar |
| Criado diretamente em `main` | bypassa governança — sem gate humano | violação arquitetural grave |
| Multi-domínio sem aprovação | viola escopo único | FALHOU com escopo_excedido |
| Contendo `.env`, credentials, tokens | violação de segurança | FALHOU — jamais commitar |
| Sem `[tipo](domínio):` na linha de título | não reconhecível como institucional | FALHOU antes de commitar |
| `git commit --amend` de commit já mergeado | altera histórico verificável | proibido — use git revert |
| Commit vazio (`--allow-empty`) | ruído institucional | proibido sem justificativa explícita |
| Arquivos fora do escopo de artefatos_alterados | execução além do autorizado | FALHOU |

---

## O QUE CODEX NÃO FAZ

Codex nunca:

- Cria commit sem handoff válido correspondente
- Decide o conteúdo do commit — segue handoff
- Interpreta instrução condicional — retorna HANDOFF_INVALIDO
- Faz merge de branch ops/ → main — isso é gate humano
- Faz push para repositório remoto sem instrução explícita
- Cria mais de um commit por ciclo sem instrução de múltiplos ciclos
- Usa `git commit --amend` em commits já reportados para Claude
- Altera `snapshot:` no commit após sua criação

---

## RETORNO PARA CLAUDE APÓS COMMIT

Após a criação do commit, Codex inclui o commit_hash no retorno:

```yaml
retorno_codex:
  ciclo_id: [mesmo ciclo_id do handoff]
  estado: CONCLUÍDO
  commit_hash: [hash completo do commit criado]
  branch: [nome da branch ops/]
  artefatos_alterados:
    - [lista dos arquivos commitados]
```

O commit_hash é o que Claude usa na etapa 7b para validar correspondência.
Sem commit_hash no retorno, Claude não pode executar a validação.
Commit sem hash no retorno = retorno inválido.

---

## INTEGRAÇÃO COM r-handoff-codex

Este módulo é complementar ao r-handoff-codex.

r-handoff-codex define: como o handoff é estruturado e validado.
r-commit-governance define: como o commit é criado a partir do handoff.

O ponto de integração:

```yaml
# Handoff (r-handoff-codex)
commit_type: rls                              → [tipo] do commit
branch_sugerido: ops/audit-logs-20260509     → nome da branch
snapshot_ref: audit-logs-001                 → campo snapshot:
```

Codex usa os três campos do handoff para criar o commit correto.

---

## PROIBIÇÕES (RESUMO)

Nunca:
- Criar commit sem todos os campos obrigatórios
- Criar commit em main diretamente
- Criar commit com arquivos fora do escopo do handoff
- Criar commit antes de verificar correspondência pré-commit (Passo 4)
- Omitir commit_hash do retorno para Claude
- Usar tipos de commit fora da tabela de tipos válidos
- Resolver divergência de artefatos por conta própria

---

## RESULTADO ESPERADO

Todo commit criado por Codex deve:
- ser rastreável até o handoff que o originou
- ser rastreável até o snapshot que originou o handoff
- ser rastreável até o usuário que validou o ciclo
- conter exatamente os artefatos autorizados e nenhum outro
- ter formato reconhecível por r-git-operacional como commit institucional
