# k-sys-handoff-format
versao: 1.0

## OBJETIVO

Define a estrutura canônica dos documentos de comunicação
entre Claude (orquestrador) e Codex (executor) no C.A.O.S.

Responde à pergunta:
"qual é o formato completo de um handoff e de um retorno?"

---

## DOIS DOCUMENTOS ESTRUTURADOS

O protocolo de handoff consiste em dois documentos:

1. **Handoff** — emitido por Claude para Codex
2. **Retorno** — emitido por Codex para Claude

Cada documento possui campos obrigatórios e opcionais.
Campos ausentes tornam o documento inválido ou incompleto.

---

## DOCUMENTO 1 — HANDOFF (Claude → Codex)

### Estrutura completa

```yaml
handoff:
  # Identificação do ciclo
  ciclo_id: [uuid-v4]
  versao_protocolo: "3.0"
  timestamp_emissao: [ISO 8601]

  # Referência institucional
  snapshot_ref: [id-do-snapshot]
  dominio: [canônico do registry — ex: public.audit_logs]
  familia: [família do domínio — ex: banco/segurança]

  # Classificação
  tarefa: [SQL | hotfix | rls | frontend | arquitetura | documentacao]

  # Estado obrigatório
  estado_atual: VALIDADO
  timestamp_validacao: [ISO 8601]

  # Instrução
  instrucao: |
    [instrução completa, estruturada, sem ambiguidade]
    [inclui: o que fazer, onde, como validar resultado]
    [inclui: arquivos ou objetos a alterar]

  # Contexto histórico (opcional, mas recomendado)
  contexto_historico:
    snapshot_anterior: [id ou null]
    decisao_anterior: [texto ou null]
    resultado_anterior: [sucesso | falha | null]
    riscos_ativos: [lista ou vazio]

  # Rastreabilidade de agentes
  agente_orquestrador: Claude
  agente_executor: Codex

  # Módulos RAG usados na análise
  modulos_usados:
    - [r/modulo-1]
    - [k/modulo-2]

  # Campos opcionais — v3.5 Git (inativos em v3.0)
  commit_type: null
  branch_sugerido: null
```

### Campos obrigatórios

| Campo | Tipo | Restrição |
|---|---|---|
| `ciclo_id` | string (uuid-v4) | único por ciclo operacional |
| `versao_protocolo` | string | deve ser "3.0" em v3.0 |
| `snapshot_ref` | string | deve referenciar snapshot existente |
| `dominio` | string | deve estar no registry de domínios |
| `tarefa` | enum | ver valores válidos abaixo |
| `estado_atual` | enum | deve ser exatamente VALIDADO |
| `timestamp_validacao` | ISO 8601 | quando o usuário aprovou |
| `instrucao` | text | não pode ser vazio ou nulo |
| `agente_orquestrador` | string | Claude |
| `agente_executor` | string | Codex (ou agente designado) |

### Valores válidos — campo `tarefa`

```
SQL | hotfix | rls | frontend | arquitetura | documentacao
rag | multi-tenant | debugging | auditoria | snapshot
```

### Campos opcionais — v3.5 (inativos em v3.0)

| Campo | Uso futuro |
|---|---|
| `commit_type` | prefixo institucional do commit Git |
| `branch_sugerido` | nome do branch ops/ para a operação |

Em v3.0 esses campos devem ser enviados como `null`.
Codex ignora campos `null`. Não falha por presença de campos opcionais.

---

## DOCUMENTO 2 — RETORNO (Codex → Claude)

### Estrutura completa

```yaml
retorno_codex:
  # Linkagem com o handoff
  ciclo_id: [mesmo uuid do handoff recebido]
  versao_protocolo: "3.0"
  timestamp_execucao: [ISO 8601]

  # Estado final da execução
  estado: [CONCLUÍDO | FALHOU | HANDOFF_INVALIDO]

  # Rastreabilidade
  agente_executor: Codex

  # Resultado (obrigatório quando estado = CONCLUÍDO)
  resultado: [descrição do que foi feito]
  artefatos_alterados:
    - [arquivo ou objeto alterado 1]
    - [arquivo ou objeto alterado 2]

  # Falha (obrigatório quando estado = FALHOU)
  erro_descricao: [descrição do erro]
  artefatos_afetados: [lista]
  reversao_parcial: [true | false]
  estado_atual_sistema: [descrição do estado após a falha]

  # Rejeição (obrigatório quando estado = HANDOFF_INVALIDO)
  campo_invalido: [campo que falhou na validação]
  motivo_rejeicao: [descrição do problema]
  acao_recomendada: [o que Claude deve corrigir]

  # Campo opcional — v3.5 Git (inativo em v3.0)
  commit_hash: null
```

### Três estados possíveis do retorno

**CONCLUÍDO:**
Execução completada com sucesso.
Campos `resultado` e `artefatos_alterados` obrigatórios.

**FALHOU:**
Execução iniciada mas falhou durante operação.
Campos `erro_descricao`, `artefatos_afetados`, `reversao_parcial` e `estado_atual_sistema` obrigatórios.

**HANDOFF_INVALIDO:**
Codex não iniciou execução — handoff rejeitado na validação.
Campos `campo_invalido`, `motivo_rejeicao` e `acao_recomendada` obrigatórios.
Diferença crítica: FALHOU = execução começou e falhou. HANDOFF_INVALIDO = nunca começou.

---

## EXEMPLOS

### Exemplo 1 — Handoff válido (hotfix RLS)

```yaml
handoff:
  ciclo_id: "f47ac10b-58cc-4372-a567-0e02b2c3d479"
  versao_protocolo: "3.0"
  timestamp_emissao: "2026-05-09T10:45:00-03:00"

  snapshot_ref: "audit-logs-001"
  dominio: "public.audit_logs"
  familia: "banco/segurança"

  tarefa: rls
  estado_atual: VALIDADO
  timestamp_validacao: "2026-05-09T10:44:00-03:00"

  instrucao: |
    Aplicar migration SQL no projeto jekzznblpekavanxcfbu via MCP apply_migration.

    Nome da migration: hotfix_audit_logs_rls_align_get_tenant_id

    SQL a executar:
    1. DROP POLICY IF EXISTS audit_logs_select_tenant ON public.audit_logs;
    2. Criar policy audit_logs_select_tenant_admin:
       USING (tenant_id = get_tenant_id() AND is_tenant_admin())
    3. Criar policy audit_logs_platform_admin:
       USING (is_platform_admin()) WITH CHECK (is_platform_admin())

    Validar após execução:
    SELECT policyname FROM pg_policies
    WHERE tablename = 'audit_logs'
    ORDER BY policyname;

    Resultado esperado: 2 policies ativas (audit_logs_platform_admin,
    audit_logs_select_tenant_admin). Ausente: audit_logs_select_tenant.

  contexto_historico:
    snapshot_anterior: null
    decisao_anterior: null
    resultado_anterior: null
    riscos_ativos: []

  agente_orquestrador: Claude
  agente_executor: Codex

  modulos_usados:
    - r/r-rls-padrao
    - k/banco/k-db-funcoes

  commit_type: null
  branch_sugerido: null
```

---

### Exemplo 2 — Handoff inválido (estado incorreto)

```yaml
handoff:
  ciclo_id: "a1b2c3d4-0000-0000-0000-000000000001"
  versao_protocolo: "3.0"
  timestamp_emissao: "2026-05-09T11:00:00-03:00"

  snapshot_ref: "audit-logs-001"
  dominio: "public.audit_logs"
  familia: "banco/segurança"

  tarefa: rls
  estado_atual: PROPOSTO        ← INVÁLIDO: deve ser VALIDADO
  timestamp_validacao: null     ← INVÁLIDO: ausente

  instrucao: |
    [instrução presente]

  agente_orquestrador: Claude
  agente_executor: Codex
  modulos_usados: []
  commit_type: null
  branch_sugerido: null
```

Motivo da rejeição: `estado_atual` é PROPOSTO em vez de VALIDADO.
`timestamp_validacao` está ausente.
Codex retorna HANDOFF_INVALIDO sem executar.

---

### Exemplo 3 — Retorno CONCLUÍDO

```yaml
retorno_codex:
  ciclo_id: "f47ac10b-58cc-4372-a567-0e02b2c3d479"
  versao_protocolo: "3.0"
  timestamp_execucao: "2026-05-09T11:10:00-03:00"

  estado: CONCLUÍDO
  agente_executor: Codex

  resultado: |
    Migration aplicada com sucesso.
    Policy audit_logs_select_tenant removida.
    Policies audit_logs_select_tenant_admin e audit_logs_platform_admin criadas.
    Validação SQL confirmou 2 policies ativas no estado esperado.

  artefatos_alterados:
    - "public.audit_logs — policy: audit_logs_select_tenant (removida)"
    - "public.audit_logs — policy: audit_logs_select_tenant_admin (criada)"
    - "public.audit_logs — policy: audit_logs_platform_admin (criada)"

  erro_descricao: null
  artefatos_afetados: null
  reversao_parcial: null
  estado_atual_sistema: null

  commit_hash: null
```

---

### Exemplo 4 — Retorno FALHOU

```yaml
retorno_codex:
  ciclo_id: "f47ac10b-58cc-4372-a567-0e02b2c3d479"
  versao_protocolo: "3.0"
  timestamp_execucao: "2026-05-09T11:08:00-03:00"

  estado: FALHOU
  agente_executor: Codex

  resultado: null
  artefatos_alterados: null

  erro_descricao: |
    Função get_tenant_id() não encontrada no schema public.
    A migration falhou ao criar a policy audit_logs_select_tenant_admin.
    Erro PostgreSQL: function get_tenant_id() does not exist.

  artefatos_afetados:
    - "public.audit_logs — estado: sem alteração (DROP executado antes da falha)"

  reversao_parcial: true
  estado_atual_sistema: |
    audit_logs_select_tenant foi removida pelo DROP.
    Novas policies NÃO foram criadas.
    Tabela audit_logs está sem política de acesso ativa para authenticated.
    ATENÇÃO: RLS habilitado sem policy = zero acesso para usuários autenticados.

  commit_hash: null
```

---

### Exemplo 5 — Retorno HANDOFF_INVALIDO

```yaml
retorno_codex:
  ciclo_id: "a1b2c3d4-0000-0000-0000-000000000001"
  versao_protocolo: "3.0"
  timestamp_execucao: null

  estado: HANDOFF_INVALIDO
  agente_executor: Codex

  resultado: null
  artefatos_alterados: null
  erro_descricao: null
  artefatos_afetados: null
  reversao_parcial: null
  estado_atual_sistema: null

  campo_invalido: "estado_atual"
  motivo_rejeicao: |
    estado_atual é PROPOSTO. Handoff só pode ser executado com estado_atual = VALIDADO.
    Indica que o gate de validação humana não foi concluído.
  acao_recomendada: |
    Verificar se o usuário aprovou a proposta.
    Se aprovado: atualizar snapshot para VALIDADO e re-emitir handoff.
    Se não aprovado: aguardar aprovação antes de emitir handoff.

  commit_hash: null
```

---

## RELAÇÃO COM r-estados-ciclo

| Evento no handoff/retorno | Transição de estado (r-estados-ciclo) |
|---|---|
| Handoff emitido com estado_atual = VALIDADO | VALIDADO → EXECUTANDO |
| Retorno com estado = CONCLUÍDO | EXECUTANDO → CONCLUÍDO |
| Retorno com estado = FALHOU | EXECUTANDO → FALHOU |
| Retorno com estado = HANDOFF_INVALIDO | ciclo permanece em VALIDADO |

---

## RETOMADA DE CICLO COM HANDOFF

Se uma sessão é interrompida com ciclo em estado VALIDADO:

1. Nova sessão carrega snapshot com estado_atual = VALIDADO
2. Claude lê campos do snapshot: instrucao, dominio, snapshot_ref
3. Claude reconstrói handoff com mesmo ciclo_id
4. Codex verifica: ciclo_id já foi processado?
   - Sim → retornar resultado anterior (idempotente)
   - Não → processar normalmente
5. Fluxo continua a partir do ponto de interrupção

O ciclo_id é o mecanismo de idempotência do protocolo.
