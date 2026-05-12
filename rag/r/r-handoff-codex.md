# r-handoff-codex
versao: 2.0

## OBJETIVO

Definir as regras operacionais do protocolo de handoff entre
Claude (orquestrador) e Codex (executor) no C.A.O.S.

Responde à pergunta:
"como Claude emite e Codex recebe, valida e responde a um handoff?"

Formato dos documentos: ver k-sys-handoff-format.

---

## PRINCÍPIO CENTRAL

O handoff é o único canal formal de comunicação entre Claude e Codex.

Claude não executa.
Codex não decide arquiteturalmente.
A separação é absoluta e não pode ser invertida.

---

## PARTE 1 — REGRAS PARA CLAUDE (emissão)

### Quando emitir handoff

Claude emite handoff quando e somente quando:
- o ciclo está em estado VALIDADO (usuário aprovou)
- a instrução está completa e sem ambiguidade
- o snapshot de referência está identificado
- o domínio canônico foi verificado no registry

### O que verificar antes de emitir

```
1. estado_atual do ciclo = VALIDADO?
2. instrucao está preenchida e completa?
3. snapshot_ref referencia snapshot existente?
4. dominio está no k-sys-registry-dominios?
5. ciclo_id é novo (não reutilização de ciclo anterior)?
6. Se r-git-operacional carregado: commit_type não null?
7. Se r-git-operacional carregado: branch_sugerido não null?
```

Se qualquer verificação falhar: não emitir handoff.
Resolver a inconsistência e apresentar ao usuário se necessário.

### O que Claude não faz no handoff

- Não inclui raciocínio interno ou justificativas na instrucao
- Não inclui campos especulativos ("talvez" / "pode ser")
- Não emite handoff com instrução condicional ("se X, então Y, senão Z")
  → instrução deve ser determinística
- Não altera handoff após emissão sem novo ciclo completo
- Não re-emite handoff com novo ciclo_id para o mesmo ciclo
  → mesmo ciclo = mesmo ciclo_id

### Instrução determinística — regra

A instrução deve descrever ação específica e verificável.

Inválido:
```
"Corrija o problema de RLS na tabela de logs se possível."
```

Válido:
```
"Aplicar DROP POLICY IF EXISTS audit_logs_select_tenant ON public.audit_logs.
Criar policy audit_logs_select_tenant_admin com USING (tenant_id = get_tenant_id()
AND is_tenant_admin()). Validar: SELECT policyname FROM pg_policies
WHERE tablename = 'audit_logs' — resultado esperado: 2 policies."
```

### Re-emissão de handoff (retomada de ciclo)

Quando ciclo está em VALIDADO e sessão foi interrompida:
- Claude recria handoff com o mesmo ciclo_id do ciclo original
- Não gera novo ciclo_id — é retomada, não novo ciclo
- Codex usa ciclo_id para verificar idempotência

---

## PARTE 2 — REGRAS PARA CODEX (recepção e validação)

### Etapa 1 — Validação obrigatória antes de executar

Codex deve validar TODOS os campos obrigatórios antes de iniciar qualquer execução.

Checklist de validação:

```
□ ciclo_id presente e não vazio?
□ versao_protocolo = "3.0"?
□ estado_atual = VALIDADO?
□ timestamp_validacao presente?
□ instrucao presente e não vazia?
□ snapshot_ref presente?
□ dominio presente?
□ agente_executor corresponde a este agente?
□ ciclo_id não foi processado anteriormente?
□ Se r-git-operacional ativo: commit_type presente e não null?
□ Se r-git-operacional ativo: branch_sugerido presente e não null?
```

Se qualquer item falhar → retornar HANDOFF_INVALIDO imediatamente.
Não executar parcialmente e depois rejeitar.

### Etapa 2 — Verificação de idempotência

Antes de executar, verificar se ciclo_id já foi processado.

Se ciclo_id foi processado anteriormente:
- Retornar o resultado anterior sem re-executar
- Incluir campo `idempotente: true` no retorno

Se ciclo_id é novo:
- Prosseguir para execução

### Etapa 3 — Execução da instrução

Codex executa exatamente a instrução recebida.

O que Codex não faz durante execução:
- Não modifica escopo da instrução (nem reduz nem expande)
- Não toma decisões arquiteturais não previstas na instrução
- Não ignora etapas de validação definidas na instrução
- Não altera domínios fora do especificado em `dominio`

Se a instrução for ambígua ou incompleta:
- Não assumir — retornar HANDOFF_INVALIDO com motivo "instrução ambígua"
- Descrever especificamente o ponto de ambiguidade em `motivo_rejeicao`

### Etapa 4 — Construção do retorno

Após execução (sucesso ou falha), Codex constrói retorno estruturado.

Regras do retorno:

- `ciclo_id` deve ser idêntico ao recebido no handoff
- `estado` deve ser um de: CONCLUÍDO, FALHOU, HANDOFF_INVALIDO
- Campos condicionais por estado devem ser preenchidos (ver k-sys-handoff-format)
- `commit_hash`: null em modo v3.0 | hash real obrigatório em modo v3.5 (r-git-operacional)
- Retorno vazio ou parcial é inválido — Claude deve rejeitar

### Campos obrigatórios por estado de retorno

**Se CONCLUÍDO:**
```
resultado:          obrigatório — não vazio
artefatos_alterados: obrigatório — lista dos artefatos modificados
```

**Se FALHOU:**
```
erro_descricao:        obrigatório
artefatos_afetados:    obrigatório
reversao_parcial:      obrigatório (true ou false)
estado_atual_sistema:  obrigatório — descrever estado pós-falha
```

**Se HANDOFF_INVALIDO:**
```
campo_invalido:      obrigatório
motivo_rejeicao:     obrigatório
acao_recomendada:    obrigatório
```

---

## PARTE 3 — REGRAS PARA CLAUDE (recepção do retorno)

### O que Claude faz com cada estado de retorno

**Retorno CONCLUÍDO — modo v3.0 (sem r-git-operacional):**
1. Verificar se ciclo_id do retorno corresponde ao handoff emitido
2. Registrar snapshot com estado = CONCLUÍDO
3. Preencher artefatos_alterados no snapshot
4. Preencher commit_hash no snapshot como null
5. Apresentar resultado ao usuário

**Retorno CONCLUÍDO — modo v3.5 (com r-git-operacional):**
1. Verificar se ciclo_id do retorno corresponde ao handoff emitido
2. Verificar commit_hash e branch presentes no retorno
3. Registrar snapshot com estado = COMMITADO + commit_hash + branch
4. Carregar diff do commit via r-git-operacional (situação 1 — divergência etapa 7b)
5. Comparar diff ↔ instrução autorizada:
   - Correspondência: estado → VERIFICADO → CONCLUÍDO
   - Divergência: estado → DIVERGENTE → apresentar ao usuário para decisão

**Retorno FALHOU:**
1. Verificar ciclo_id
2. Registrar snapshot com estado = FALHOU
3. Preencher erro_descricao, artefatos_afetados, reversao_parcial
4. Se reversao_parcial = true: sinalizar ao usuário o estado inconsistente
5. Retornar ciclo para estado ANALISADO com contexto da falha
6. Nova proposta deve incluir contexto da falha anterior

**Retorno HANDOFF_INVALIDO:**
1. Ciclo permanece em VALIDADO (gate não foi invalidado)
2. Corrigir campo apontado em campo_invalido
3. Re-emitir handoff corrigido (mesmo ciclo_id)
4. Não criar novo ciclo

**Retorno parcial ou corrompido (campos faltando):**
1. Claude rejeita o retorno como inválido
2. Solicita que Codex reenvie retorno completo
3. Não atualiza snapshot com retorno incompleto

### Validação de consistência Claude

Antes de marcar ciclo como CONCLUÍDO, Claude verifica:

```
□ ciclo_id do retorno = ciclo_id do handoff emitido?
□ estado = CONCLUÍDO?
□ resultado não vazio?
□ artefatos_alterados presentes?
```

Se qualquer verificação falhar: não marcar CONCLUÍDO — investigar.

---

## PARTE 4 — TRATAMENTO DE CASOS ESPECIAIS

### Caso 1 — Codex não responde (timeout operacional)

Se Codex não retorna após período razoável:
1. Claude não assume CONCLUÍDO nem FALHOU
2. Claude sinaliza ao usuário: "Codex não retornou resultado para ciclo_id X"
3. Usuário decide: aguardar, verificar manualmente o estado, ou cancelar ciclo
4. Ciclo permanece em EXECUTANDO no snapshot até retorno ou decisão humana

### Caso 2 — Dois retornos para o mesmo ciclo_id

Se Claude recebe dois retornos com o mesmo ciclo_id:
1. Primeiro retorno válido é aceito
2. Segundo retorno é descartado (idempotência)
3. Se os dois retornos divergem: sinalizar ao usuário antes de qualquer ação

### Caso 3 — Instrução que ultrapassa escopo do domínio

Se durante execução Codex identifica que a instrução afetaria domínios
além do especificado em `dominio`:
1. Não executar a parte fora do escopo
2. Retornar FALHOU com erro_descricao: "instrução ultrapassa escopo do domínio"
3. Listar em artefatos_afetados os domínios que seriam impactados
4. Claude reavalia e, se necessário, cria ciclos separados por domínio

---

## INTEGRAÇÃO COM r-estados-ciclo

| Evento | Transição de estado |
|---|---|
| Claude emite handoff válido | VALIDADO → EXECUTANDO |
| Codex retorna CONCLUÍDO (modo v3.0) | EXECUTANDO → CONCLUÍDO |
| Codex retorna CONCLUÍDO com commit_hash (modo v3.5) | EXECUTANDO → COMMITADO |
| Claude valida diff sem divergência (modo v3.5) | COMMITADO → VERIFICADO |
| Claude detecta divergência no diff (modo v3.5) | COMMITADO → DIVERGENTE |
| Claude confirma VERIFICADO (modo v3.5) | VERIFICADO → CONCLUÍDO |
| Codex retorna FALHOU | EXECUTANDO → FALHOU |
| Claude retorna ciclo de FALHOU | FALHOU → ANALISADO |
| Codex retorna HANDOFF_INVALIDO | ciclo permanece em VALIDADO |
| Claude corrige e re-emite | VALIDADO → EXECUTANDO (nova tentativa) |

---

## CAMPOS CONDICIONAIS — v3.5

Estes campos têm comportamento diferente em modo v3.0 e modo v3.5.

### Modo v3.0 (sem r-git-operacional)

| Campo | Comportamento |
|---|---|
| `commit_type` no handoff | null — Codex ignora |
| `branch_sugerido` no handoff | null — Codex ignora |
| `commit_hash` no retorno | null — Claude ignora |

### Modo v3.5 (com r-git-operacional carregado)

| Campo | Comportamento |
|---|---|
| `commit_type` no handoff | obrigatório — Codex cria commit `[tipo](domínio)` |
| `branch_sugerido` no handoff | obrigatório — Codex cria branch ops/ antes de executar |
| `commit_hash` no retorno | obrigatório — hash real do commit criado |

Violações em modo v3.5:

```
commit_type = null com r-git-operacional ativo → HANDOFF_INVALIDO
branch_sugerido = null com r-git-operacional ativo → HANDOFF_INVALIDO
commit_hash ausente no retorno em v3.5 → retorno inválido → Claude investiga
```

---

## PROIBIÇÕES

Claude nunca:
- Emite handoff com estado_atual ≠ VALIDADO
- Emite handoff com instrução vazia ou condicional
- Altera handoff após emissão sem novo ciclo
- Marca CONCLUÍDO sem retorno de Codex
- Aceita retorno parcial como válido

Codex nunca:
- Executa handoff sem validar todos os campos obrigatórios
- Toma decisões arquiteturais não previstas na instrução
- Modifica domínios fora do escopo definido em `dominio`
- Retorna resultado sem ciclo_id correspondente
- Omite campos condicionais obrigatórios do retorno

---

## RESULTADO ESPERADO

O protocolo de handoff deve garantir que:
- toda execução de Codex tem origem em instrução validada
- todo ciclo tem estado rastreável antes e depois do handoff
- falhas são documentadas com contexto suficiente para retomada
- retomada de ciclos interrompidos é possível sem perda de estado
- a separação Claude/Codex é verificável e auditável
