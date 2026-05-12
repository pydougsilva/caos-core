# r-rollback-contextual
versao: 1.0

## OBJETIVO

Definir como ciclos operacionais são revertidos preservando
integridade técnica e institucional.

Responde à pergunta:
"como reverter um ciclo sem destruir histórico nem perder causalidade?"

---

## PRINCÍPIO CENTRAL

Um rollback no C.A.O.S é sempre duplo:
técnico (Git) e institucional (Snapshot).

Os dois são inseparáveis.
Nunca executar um sem o outro.

---

## DOIS TIPOS DE ROLLBACK

### Rollback Técnico — Git

Desfaz as alterações de arquivos introduzidas por um commit.

Mecanismo: `git revert <commit>`

O `git revert` cria um novo commit que inverte as mudanças do original.
O histórico é preservado — o commit revertido continua visível no log.
A operação é rastreável, auditável e reversível.

```
Antes:       A → B → C (commit a reverter)
Após revert: A → B → C → C' (C' desfaz C)
```

Nunca usar `git reset --hard` ou `git push --force` para rollback.
Isso destrói histórico verificável — violação arquitetural grave.

### Rollback Institucional — Snapshot

Registra que uma decisão homologada foi desfeita.

Mecanismo: atualizar snapshot + criar novo snapshot de reversão

O snapshot do ciclo revertido recebe:
```yaml
estado_atual: REVERTIDO
revertido_em: [timestamp]
motivo_reversao: [texto]
commit_revert_hash: [hash do commit C']
```

Um novo snapshot incremental documenta a reversão:
```yaml
tipo: incremental
base_ref: [snapshot-ID do ciclo revertido]
delta:
  estado_atual: REVERTIDO
  resultado: revertido
  nova_decisao: "[motivo e contexto da reversão]"
  artefatos_alterados: [artefatos do commit de revert]
```

---

## VÍNCULO OBRIGATÓRIO

Os dois rollbacks devem ocorrer no mesmo ciclo operacional.

| Situação | Consequência |
|---|---|
| Git revert sem rollback institucional | snapshot reporta CONCLUÍDO, Git reporta revertido — divergência |
| Rollback institucional sem Git revert | snapshot reporta REVERTIDO, sistema ainda tem as alterações ativas |
| Ambos executados no mesmo ciclo | integridade preservada — causalidade técnica e institucional alinhadas |

Codex executa o Git revert.
Claude executa o rollback institucional.
Usuário autoriza ambos.

---

## SEQUÊNCIA COMPLETA DE ROLLBACK

```
Passo 1   Pedido de rollback chega a Claude
          → identificar: commit_hash do ciclo a reverter
          → identificar: snapshot_ref do ciclo a reverter
          → identificar: artefatos que serão desfeitos

Passo 2   Claude prepara apresentação dual:

          IMPACTO TÉCNICO:
          "Reverter commit [hash] desfará as seguintes alterações:
           [lista de artefatos afetados]"

          IMPACTO INSTITUCIONAL:
          "O snapshot [ID] será marcado como REVERTIDO.
           Histórico do domínio [domínio] registrará a reversão."

Passo 3   GATE 1 — Usuário aprova impacto técnico + institucional
          → se rejeitado: rollback cancelado, ciclo permanece CONCLUÍDO
          → se aprovado: prosseguir

Passo 4   Codex cria branch de reversão:
          ops/revert-[domínio-abreviado]-[YYYYMMDD]

Passo 5   Codex executa: git revert <commit_hash>
          → commit criado com formato institucional:
            [revert](domínio): reverter [descrição do ciclo original]
            snapshot: [novo-snapshot-revert-ID]
            agent-executor: Codex
            agent-orchestrator: Claude
            risks-addressed: 0

Passo 6   Claude atualiza snapshot do ciclo revertido:
          → estado_atual: REVERTIDO
          → commit_revert_hash: [hash do commit de revert]
          → registrar delta incremental com motivo da reversão

Passo 7   Claude apresenta o resultado ao usuário:
          → diff do commit de revert
          → snapshot atualizado com REVERTIDO
          → novo snapshot incremental criado

Passo 8   GATE 2 — Usuário autoriza merge da branch de reversão → main
          → se rejeitado: revert fica na branch, não em main ainda
          → se aprovado: Codex faz merge e deleta branch

Passo 9   Codex: merge ops/revert-[...] → main
          Codex: deleta branch de reversão
          Claude: registra snapshot como concluído
```

**Dois gates humanos obrigatórios.**
Nenhum rollback automático. Nenhuma exceção.

---

## ESTADOS ENVOLVIDOS

Transições de estado durante rollback:

```
Ciclo original (após CONCLUÍDO):
  CONCLUÍDO → REVERTIDO

Ciclo de reversão (novo ciclo):
  DETECTADO → ANALISADO → PROPOSTO → VALIDADO (Gate 1)
  → EXECUTANDO → COMMITADO (v3.5) → VERIFICADO → CONCLUÍDO
  → merge autorizado (Gate 2)
```

O rollback é tratado como um ciclo operacional completo —
não como uma operação especial fora do fluxo.

---

## ROLLBACK PARCIAL

Quando apenas parte de um commit precisa ser revertida:

```
Situação: commit alterou artefatos A, B e C.
          Apenas A precisa ser revertido.

Comportamento:
  git revert reverte A, B e C juntos.
  Não é possível reverter apenas A via git revert seletivo
  sem recriar B e C.

Abordagem correta:
  1. Não usar git revert
  2. Criar novo ciclo de hotfix para desfazer apenas A
  3. Novo handoff: instrução para corrigir apenas A
  4. Novo commit: [hotfix](domínio) com escopo único A
```

Rollback parcial não é tecnicamente um `git revert`.
É um novo ciclo de correção com escopo restrito.
Claude deve apresentar essa distinção ao usuário antes de prosseguir.

---

## ROLLBACK DE CICLOS PRÉ-V3.5

Ciclos executados antes de v3.5 não possuem `commit_hash`.

```yaml
snapshot.commit_hash: null   ← ciclo v3.0 ou anterior
```

Nesse caso, rollback técnico via `git revert` não é possível.
Não há commit para reverter.

Abordagem para ciclos pré-v3.5:

```
1. Claude identifica: não há evidência Git rastreável para este ciclo
2. Claude apresenta ao usuário:
   "Este ciclo (v3.0 pré-Git) não possui commit verificável.
    Rollback técnico não é possível via git revert.
    Reversão deve ser feita manualmente."
3. Claude propõe: novo ciclo de correção manual
4. Usuário valida a abordagem [GATE]
5. Novo handoff emitido com instrução de reversão manual
6. Codex executa, cria commit institucional do ciclo de correção
7. Snapshot original: estado_atual = REVERTIDO (rollback institucional feito)
8. Novo snapshot documenta a reversão
```

O rollback institucional ainda ocorre — apenas o técnico não é automático.

---

## PRESERVAÇÃO DA TRILHA HISTÓRICA

O histórico Git nunca deve ser reescrito durante rollback.

| Operação | Permitida | Motivo |
|---|---|---|
| `git revert` | ✅ | cria novo commit, histórico preservado |
| `git reset --soft` | ❌ | move HEAD, não altera arquivos, histórico comprometido |
| `git reset --hard` | ❌ | destrói commits e alterações de arquivo |
| `git push --force` | ❌ | destrói histórico remoto verificável |
| `git rebase` com squash | ❌ | reescreve histórico, commits originais perdidos |

O commit de revert deve ter sua própria entrada no log.
Isso significa: a decisão original + sua reversão são ambas rastreáveis.

Resultado: dado qualquer commit em `main`, é possível reconstruir
exatamente o que foi feito e o que foi desfeito — sem gaps.

---

## PROIBIÇÕES

Nunca:
- Executar `git reset --hard` para rollback
- Executar `git push --force` em qualquer branch
- Fazer rollback técnico sem rollback institucional correspondente
- Marcar snapshot como REVERTIDO sem commit de revert em Git
- Automatizar rollback sem os dois gates humanos
- Tratar rollback como ciclo de menor prioridade — tem os mesmos gates do ciclo original
- Usar rollback para esconder execuções que deveriam ter sido ciclos separados

---

## RESULTADO ESPERADO

Todo rollback executado deve:
- preservar o histórico completo: ciclo original + ciclo de reversão
- ter evidência verificável em ambos os sentidos
- registrar institucionalmente a decisão de reverter
- permitir auditoria futura: "o que foi feito, quando foi desfeito e por quê"
