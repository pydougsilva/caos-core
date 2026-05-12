# r-replay-operacional
versao: 1.0

## OBJETIVO

Definir como ciclos históricos podem ser reconstruídos e
reproduzidos de forma assistida e auditável.

Responde à pergunta:
"como reproduzir um ciclo histórico num novo contexto?"

---

## PRINCÍPIO CENTRAL

Replay é reconstrução assistida — não é automação.

Claude reconstrói o contexto. O usuário valida a reconstrução.
Codex executa apenas após validação humana explícita.

O replay nunca executa sem gate humano.

---

## O QUE É REPLAY

Replay é o processo de usar os artefatos de um ciclo histórico
(snapshot + commit) para reconstituir e re-executar aquele ciclo
num novo contexto operacional.

Casos de uso:
- reproduzir uma correção aplicada em produção no ambiente de staging
- re-executar uma migration em um novo tenant
- reconstruir o que aconteceu durante um incidente
- aplicar uma decisão homologada a um domínio semelhante

---

## REPLAY ≠ AUTOMAÇÃO

| Replay | Automação |
|---|---|
| Claude reconstrói com base em artefatos | sistema executa sem reconstrução |
| Usuário valida a reconstrução [GATE] | execução sem gate humano |
| Codex executa a instrução validada | sistema executa diretamente |
| Novo ciclo registrado independente | histórico pode ser sobrescrito |
| Equivalência ambiental avaliada | ambiente assumido como equivalente |

Replay requer julgamento humano em cada etapa.
Automação não requer. Por isso, replay — não automação.

---

## INPUTS NECESSÁRIOS

Para iniciar um replay, dois artefatos são necessários:

```
snapshot_id:    ID do snapshot do ciclo a reproduzir
commit_hash:    hash do commit institucional correspondente
```

Para replay de ciclo pré-v3.5 (sem commit_hash):
```
snapshot_id:    ID do snapshot do ciclo a reproduzir
commit_hash:    null   ← replay parcial — sem evidência de diff
```

Nenhum replay é iniciado sem ao menos o snapshot_id.
Replay sem commit_hash é possível mas com limitações documentadas.

---

## AVALIAÇÃO DE EQUIVALÊNCIA AMBIENTAL

Antes de reconstruir a instrução, Claude avalia se o ambiente alvo
é suficientemente equivalente ao ambiente original.

Equivalência ambiental não exige perfeição — exige suficiência.

### Dimensões a avaliar

```
1. Estado do domínio alvo:
   O domínio no ambiente alvo está no estado esperado pelo ciclo original?
   Ex: a policy que o ciclo original removeu existe no ambiente alvo?

2. Dependências:
   As funções, tabelas e objetos referenciados na instrução existem?
   Ex: get_tenant_id() existe no schema do ambiente alvo?

3. Versões:
   A versão do banco, framework ou runtime é compatível?

4. Dados:
   Há registros que a instrução pressupõe existir?
```

### Resultado da avaliação

```
EQUIVALENTE:
  Todas as dimensões críticas satisfeitas.
  Replay pode prosseguir para reconstrução.

PARCIALMENTE EQUIVALENTE:
  Algumas dimensões divergem mas não impedem execução.
  Claude documenta as divergências antes de apresentar ao usuário.
  Usuário decide se prossegue com as divergências conhecidas.

NÃO EQUIVALENTE:
  Uma ou mais dimensões críticas estão ausentes.
  Ex: tabela referenciada não existe, função dependente ausente.
  Claude apresenta bloqueadores ao usuário.
  Replay não deve prosseguir sem resolução dos bloqueadores.
```

---

## SEQUÊNCIA COMPLETA DE REPLAY

```
Passo 1   Pedido de replay chega a Claude
          → identificar: snapshot_id + commit_hash (se disponível)
          → identificar: ambiente alvo

Passo 2   Claude carrega snapshot:
          → decisão original
          → riscos identificados
          → módulos usados
          → artefatos alterados no ciclo original

Passo 3   Claude carrega diff do commit (se commit_hash disponível):
          → o que foi realmente alterado
          → quais arquivos, quais linhas
          → confirma consistência com snapshot

Passo 4   Claude avalia equivalência ambiental:
          → estado do domínio alvo
          → dependências disponíveis
          → resultado: EQUIVALENTE | PARCIALMENTE EQUIVALENTE | NÃO EQUIVALENTE

Passo 5   Claude reconstrói instrução:
          → instrução baseada em snapshot.decisao + diff (quando disponível)
          → instrução adaptada para o ambiente alvo (se necessário)
          → documenta diferenças em relação ao ciclo original

Passo 6   Claude apresenta ao usuário:
          → contexto do ciclo original
          → instrução reconstruída
          → avaliação de equivalência
          → divergências identificadas (se houver)

Passo 7   GATE — Usuário valida:
          → a reconstrução corresponde à intenção?
          → a avaliação de equivalência está correta?
          → as divergências são aceitáveis?
          → aprovação explícita antes de qualquer execução

Passo 8   Claude emite handoff para Codex:
          → instrução validada pelo usuário
          → snapshot_ref: novo snapshot do ciclo de replay
          → campo adicional: replay_de: [snapshot-ID original]

Passo 9   Codex executa no ambiente alvo
          → novo ciclo operacional independente
          → commit com [tipo](domínio) correspondente

Passo 10  Novo snapshot registrado com referência ao ciclo original:
          replay_de: [snapshot_id original]
          commit_hash: [novo commit do replay]
```

Um gate humano obrigatório em Passo 7.
Sem exceções — mesmo para ciclos considerados simples.

---

## O QUE REPLAY GARANTE

- O contexto do ciclo original está disponível e estruturado
- A instrução é reconstruída a partir de artefatos verificáveis
- O usuário valida explicitamente antes da execução
- O novo ciclo é registrado de forma independente com referência ao original
- A rastreabilidade é bidirecional: replay → original, original → replay(s)

---

## O QUE REPLAY NÃO GARANTE

- Resultado idêntico ao ciclo original (ambiente pode divergir)
- Ausência de efeitos colaterais não presentes no ciclo original
- Equivalência perfeita de estado após execução
- Que o ciclo original foi a abordagem correta para o novo contexto

Claude deve explicitar essas não-garantias ao usuário em Passo 6.
O usuário decide com conhecimento dos limites — não com expectativa de reprodução exata.

---

## REPLAY PARCIAL

Quando apenas parte do ciclo original é reproduzível:

```
Situação: ciclo original alterou artefatos A, B, C.
          No ambiente alvo, B não existe — apenas A e C são reproduzíveis.

Comportamento:
  Claude reconstrói instrução apenas para A e C.
  Documenta explicitamente: "B não reproduzível no ambiente alvo — [motivo]"
  Usuário valida o replay parcial [GATE]
  Codex executa apenas A e C
  Snapshot registra: replay parcial, B excluído, motivo documentado
```

Replay parcial é válido.
Deve ser documentado explicitamente — não silenciado.

---

## REPLAY DE CICLOS PRÉ-V3.5

Ciclos sem `commit_hash` (executados antes de v3.5) podem ser reproduzidos
apenas com base no snapshot — sem diff verificável.

```
Limitações documentadas:
  - Instrução reconstruída a partir de snapshot.decisao apenas
  - Sem verificação por diff do que foi realmente executado
  - Menor precisão na reconstrução
  - Reconstrução "snapshot-only"

Comportamento:
  Passo 3 (carregar diff) é pulado com nota explícita:
  "Ciclo pré-v3.5: sem commit verificável. Reconstrução baseada apenas em snapshot."

  Usuário deve avaliar se a reconstrução a partir do snapshot
  é suficientemente precisa antes de validar [GATE].
```

O replay de ciclos pré-v3.5 é válido — com transparência sobre a limitação.

---

## VÍNCULO ENTRE CICLO ORIGINAL E REPLAY

O novo snapshot do ciclo de replay referencia o ciclo original:

```yaml
# Novo snapshot do replay
id: audit-logs-replay-001
tipo: incremental
base_ref: [último base do domínio]
replay_de: audit-logs-001          ← referência ao ciclo original
data: [data do replay]
...
```

Isso cria rastreabilidade em dois sentidos:

```
Dado snapshot original (audit-logs-001):
  → ver se existe replay: procurar snapshots com replay_de: audit-logs-001

Dado snapshot de replay (audit-logs-replay-001):
  → replay_de: audit-logs-001 → ciclo original com contexto completo
```

---

## PROIBIÇÕES

Nunca:
- Executar replay sem gate humano em Passo 7
- Assumir equivalência ambiental sem avaliação explícita
- Omitir as não-garantias ao usuário
- Criar replay como "cópia" do ciclo original sem novo ciclo independente
- Usar replay para automatizar execuções recorrentes
- Silenciar divergências entre ambiente original e alvo
- Omitir o campo `replay_de` no snapshot de replay
- Re-executar automaticamente ciclos com base em "semelhança" sem reconstrução explícita

---

## RESULTADO ESPERADO

Todo replay executado deve:
- partir de artefatos verificáveis (snapshot + commit quando disponível)
- avaliar equivalência ambiental antes de reconstruir
- ter instrução reconstruída explicitamente por Claude
- ter validação humana explícita antes da execução
- gerar novo ciclo independente referenciado ao ciclo original
- documentar limitações quando replay for parcial ou pré-v3.5
