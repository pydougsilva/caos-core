# r-snapshots-incrementais
versao: 1.0

## OBJETIVO

Definir como snapshots de um mesmo domínio evoluem ao longo do tempo
através de deltas incrementais sobre uma base imutável.

Responde à pergunta:
"quando criar snapshot completo vs incremental e como reconstituir o estado?"

---

## PRINCÍPIO CENTRAL

O histórico de um domínio é uma cadeia:
um snapshot base completo seguido de deltas que registram apenas o que mudou.

A base é imutável após criação.
Deltas são imutáveis após criação.
O estado atual é sempre reconstituível pela cadeia.

---

## DOIS TIPOS DE SNAPSHOT

### Snapshot Base

Registro completo e autossuficiente.
Não depende de nenhum snapshot anterior para ser interpretado.
Pode existir isoladamente.

Campo identificador: `tipo: base`
Snapshots criados antes de v3.0 (formato v2.x) são tratados como base implícita —
mesmo sem campo `tipo`, são bases por definição.

### Snapshot Incremental (delta)

Registro parcial que contém apenas os campos que mudaram.
Depende de uma base para ser interpretado.
Referencia o `id` da sua base obrigatoriamente.
Não pode existir sem base válida.

Campo identificador: `tipo: incremental`
Campo obrigatório: `base_ref` — ID do snapshot base da cadeia.

---

## QUANDO CRIAR CADA TIPO

### Criar snapshot BASE quando:

1. **Primeira operação no domínio** — não existe snapshot anterior.
2. **Após estado REVERTIDO** — o estado pós-revert é documentado como nova base.
   A base anterior permanece no histórico como registro do estado revertido.
3. **Mudança arquitetural fundamental** — o domínio mudou de forma que
   o delta seria > 80% de um snapshot completo.
   Critério prático: se mais de 5 campos mudam simultaneamente → novo base.
4. **Cadeia de deltas ≥ 5** — para evitar cadeias longas que dificultam
   reconstituição. Ao criar o 6º delta, criar novo base consolidado.
5. **Decisão explícita do usuário ou Claude** — quando a limpeza de histórico
   for operacionalmente necessária.

### Criar snapshot INCREMENTAL quando:

1. **Operações subsequentes no mesmo domínio** — segunda operação em diante.
2. **Mesmo tipo de tarefa com resultado diferente** — ex: segunda auditoria RLS.
3. **Atualização de riscos ativos** — riscos resolvidos ou novos riscos identificados.
4. **Mudança de estado_atual do domínio** — ex: estável → degradado.
5. **Ciclo que não altera decisão arquitetural principal** — apenas resultado ou
   dados operacionais.

---

## ESTRUTURA DO SNAPSHOT BASE

```yaml
id: [domínio-abreviado]-[NNN]        # ex: audit-logs-001
tipo: base
dominio: public.audit_logs
familia: banco/segurança
data: 2026-05-09

tarefa: hotfix-rls
modulos_usados:
  - r/r-rls-padrao
  - k/banco/k-db-funcoes

decisao: |
  [decisão arquitetural completa]

resultado: sucesso | falha | pendente
estado_atual: CONCLUÍDO
historico_estados:
  - estado: DETECTADO   timestamp: [ISO 8601]
  - estado: ANALISADO   timestamp: [ISO 8601]
  - estado: PROPOSTO    timestamp: [ISO 8601]
  - estado: VALIDADO    timestamp: [ISO 8601]
  - estado: EXECUTANDO  timestamp: [ISO 8601]
  - estado: CONCLUÍDO   timestamp: [ISO 8601]

riscos_vistos:
  - [risco 1]
  - [risco 2]
riscos_ativos: []

artefatos_alterados:
  - [artefato 1]
  - [artefato 2]

agente_orquestrador: Claude
agente_executor: Codex

# Campos opcionais — v3.5 Git (inativos em v3.0)
commit_hash: null
branch: null
```

---

## ESTRUTURA DO SNAPSHOT INCREMENTAL

```yaml
id: [domínio-abreviado]-[NNN]        # ex: audit-logs-002
tipo: incremental
base_ref: audit-logs-001             # ID do snapshot base desta cadeia
dominio: public.audit_logs
data: [data do ciclo incremental]
tarefa: [classificação do novo ciclo]

delta:
  # Incluir APENAS os campos que mudaram
  # Campos ausentes = sem alteração em relação ao estado reconstituído

  estado_atual: [novo estado se mudou]
  resultado: [novo resultado se mudou]

  riscos_ativos:
    adicionar: [lista de novos riscos]
    remover: [lista de riscos resolvidos]

  nova_decisao: |
    [nova decisão arquitetural, se houver]

  artefatos_alterados:
    - [novos artefatos deste ciclo]

  historico_estados:
    - estado: [estado]  timestamp: [ISO 8601]
    # apenas estados DO NOVO CICLO, não histórico completo

# Rastreabilidade (obrigatório mesmo em incremental)
agente_orquestrador: Claude
agente_executor: Codex

# Campos opcionais — v3.5 Git
commit_hash: null
branch: null
```

### Campos obrigatórios no incremental

| Campo | Obrigatório | Motivo |
|---|---|---|
| `id` | sim | identificação única |
| `tipo: incremental` | sim | distingue de base |
| `base_ref` | sim | referência à base da cadeia |
| `dominio` | sim | confirma o domínio |
| `data` | sim | ordenação da cadeia |
| `delta` | sim | sem delta = incremental vazio, inválido |
| `agente_orquestrador` | sim | rastreabilidade |

---

## ESTRUTURA DO DELTA

O delta contém apenas os campos que mudaram.
Campos ausentes no delta = valor do campo não mudou.

### Campos que podem aparecer no delta

```yaml
delta:
  # Estado do domínio
  estado_atual: estável | degradado | em_manutenção | desconhecido

  # Resultado do novo ciclo
  resultado: sucesso | falha | pendente

  # Decisão (somente se houve nova decisão arquitetural)
  nova_decisao: [texto]

  # Riscos — operação de conjunto, não substituição
  riscos_ativos:
    adicionar: [lista]
    remover: [lista]

  # Artefatos do novo ciclo
  artefatos_alterados: [lista]

  # Estados do novo ciclo
  historico_estados: [lista]

  # Atualização de tarefa (se ciclo tem tipo diferente)
  tarefa: [novo tipo]
```

### Campos que NÃO aparecem no delta

```yaml
# Estes campos são definidos na base e não mudam:
dominio:          # imutável — pertence à base
familia:          # imutável — pertence à base
modulos_usados:   # registrado por ciclo no historico_estados
decisao:          # campo base — nova_decisao adiciona, não substitui
riscos_vistos:    # imutável — registro histórico de todos os riscos já vistos
```

---

## RECONSTRUÇÃO DE ESTADO COMPLETO

Para obter o estado atual de um domínio:

```
1. Localizar snapshot base mais recente do domínio
   (base mais recente = base com maior data na cadeia)

2. Coletar todos os deltas que referenciam esse base_ref,
   ordenados por data crescente

3. Inicializar estado = cópia completa do snapshot base

4. Para cada delta na ordem:
   a. Se delta.estado_atual presente → estado.estado_atual = delta.estado_atual
   b. Se delta.resultado presente → estado.resultado = delta.resultado
   c. Se delta.nova_decisao presente → estado.decisao += "\n---\n" + delta.nova_decisao
   d. Se delta.riscos_ativos.adicionar → estado.riscos_ativos += lista
   e. Se delta.riscos_ativos.remover → estado.riscos_ativos -= lista
   f. Se delta.artefatos_alterados → estado.artefatos_alterados += lista
   g. Se delta.historico_estados → estado.historico_estados += lista

5. Retornar estado reconstituído
```

### Estado num ponto histórico

Para reconstruir o estado em data específica:
- Executar mesma lógica, mas parar de aplicar deltas quando `delta.data > data_alvo`

---

## INTEGRIDADE DE REFERÊNCIA

### Regras obrigatórias

**R1 — Base deve existir antes do delta:**
Antes de criar snapshot incremental, verificar que `base_ref` existe no sistema.
Se base não existe: criar snapshot base antes do incremental.

**R2 — Base não pode estar REVERTIDA:**
Se o snapshot base referenciado tem `resultado: revertido` ou
`estado_atual: REVERTIDO` → não criar delta sobre essa base.
Criar nova base documentando o estado pós-revert.

**R3 — Domínio deve ser consistente:**
`delta.dominio` deve ser idêntico a `base.dominio`.
Delta não pode ser criado para domínio diferente da base.

**R4 — Data do delta ≥ data da base:**
Um delta não pode ser anterior à sua base.
`delta.data >= base.data`

**R5 — ID único:**
ID de snapshot incremental deve ser único no sistema,
seguindo mesma sequência do domínio.

---

## LIMITE DE CADEIA

Máximo de 5 deltas por base antes de consolidar nova base.

Quando o 6º delta seria criado:
1. Reconstituir estado completo pela cadeia atual (base + 5 deltas)
2. Criar novo snapshot base com o estado reconstituído
3. Criar delta referenciando a nova base (em vez do 6º delta sobre a base antiga)
4. A base antiga e seus deltas permanecem no histórico — não são deletados

Justificativa: cadeias longas aumentam o custo de reconstituição e aumentam
o risco de erro acumulado. Novo base a cada 5 deltas mantém operação eficiente.

---

## COMPATIBILIDADE COM v2.x

Snapshots criados antes de v3.0 (formato v2.x):

- Não possuem campo `tipo`
- Tratados como `tipo: base` implícito
- Podem ser usados como `base_ref` por deltas v3.0
- Não precisam ser migrados ou alterados
- r-recuperacao-contextual continua funcionando sobre eles sem modificação

Snapshot v2.x existente (audit-logs-001):
```yaml
# Snapshot v2.x — sem campo tipo
id: audit-logs-001
dominio: public.audit_logs
# [...campos v2.x...]
# tipo: ausente → tratado como base implícita
```

Primeira operação futura em audit_logs com v3.0 ativo:
```yaml
# Primeiro delta sobre base v2.x
id: audit-logs-002
tipo: incremental
base_ref: audit-logs-001   # referencia snapshot v2.x como base
# [...delta...]
```

A ausência do campo `tipo` na base não impede a criação de delta sobre ela.

---

## EXEMPLOS

### Exemplo 1 — Cadeia completa em public.audit_logs

```
audit-logs-001 (base v2.x — hotfix-rls, 2026-05-09, resultado: sucesso)
audit-logs-002 (incremental, base_ref: audit-logs-001, 2026-06-15)
  delta:
    estado_atual: degradado
    riscos_ativos:
      adicionar: ["policy platform_admin retornou obsoleta após deploy"]
audit-logs-003 (incremental, base_ref: audit-logs-001, 2026-06-20)
  delta:
    estado_atual: estável
    resultado: sucesso
    riscos_ativos:
      remover: ["policy platform_admin retornou obsoleta após deploy"]
    nova_decisao: "Adicionar migration idempotente para recriar policy
                   platform_admin se ausente após deploy"
    artefatos_alterados:
      - "public.audit_logs — policy: audit_logs_platform_admin (recreada)"
```

Estado reconstituído de audit_logs em 2026-06-20:
- estado_atual: estável (de 003)
- resultado: sucesso (de 003)
- riscos_ativos: [] (adicionado em 002, removido em 003)
- decisao: [decisão original de 001] + "\n---\n" + [nova_decisao de 003]

---

### Exemplo 2 — Snapshot inválido (delta sem base)

```yaml
id: fornadas-002
tipo: incremental
base_ref: fornadas-001      ← base_ref que não existe ainda
dominio: public.fornadas
```

Violação de R1 — base_ref não existe.
Ação: criar fornadas-001 como base antes de criar fornadas-002.

---

### Exemplo 3 — Criação de nova base após 5 deltas

```
produtos-001 (base)
produtos-002 (incremental, base_ref: produtos-001)
produtos-003 (incremental, base_ref: produtos-001)
produtos-004 (incremental, base_ref: produtos-001)
produtos-005 (incremental, base_ref: produtos-001)
produtos-006 (incremental, base_ref: produtos-001) ← 5º delta — limite
```

Próximo ciclo (que seria produtos-007):
1. Reconstituir estado completo: produtos-001 + todos os deltas
2. Criar produtos-007 como nova base com estado completo
3. Próximos deltas referenciam base_ref: produtos-007

---

## PROIBIÇÕES

Nunca:
- Criar snapshot incremental sem `base_ref` válido
- Criar delta com `base_ref` de snapshot em estado REVERTIDO
- Editar snapshot base ou delta após criação (imutabilidade)
- Criar delta com `data` anterior à data da sua base
- Usar campo `decisao` no delta (usar `nova_decisao`)
- Substituir `riscos_vistos` no delta (campo imutável da base)
- Criar cadeia com mais de 5 deltas sem consolidar nova base

---

## RESULTADO ESPERADO

O sistema de snapshots incrementais deve:
- preservar histórico completo e imutável de cada domínio
- permitir reconstituição de estado em qualquer ponto temporal
- reduzir redundância de dados em ciclos frequentes no mesmo domínio
- manter compatibilidade total com snapshots v2.x existentes
- impedir estados inconsistentes por referência inválida
