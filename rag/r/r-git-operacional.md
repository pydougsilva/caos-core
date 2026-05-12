# r-git-operacional
versao: 1.0

## OBJETIVO

Definir como Claude lê e interpreta histórico Git operacionalmente
no fluxo do C.A.O.S.

Responde à pergunta:
"quando e como Claude usa histórico Git para enriquecer a análise?"

Este módulo define leitura — não escrita.
Claude nunca cria commits, branches ou altera o repositório.

---

## PRINCÍPIO CENTRAL

Git é consultado para enriquecer o contexto da análise —
não para substituir snapshot, não para tomar decisões,
não para alterar o fluxo de governança.

A leitura Git é sempre complementar ao snapshot.
Se o snapshot não existir, o histórico Git não substitui.

---

## QUANDO ATIVAR

Este módulo é ativado na etapa 0b do fluxo operacional,
após a identificação do domínio (etapa 0a).

Condições de ativação:

```
□ domínio canônico identificado (etapa 0a concluída)
□ domínio possui histórico de snapshot (v3.0 ou anterior)
```

Se ambas as condições forem verdadeiras:
→ executar leitura Git conforme este módulo

Se qualquer condição falhar:
→ não ativar leitura Git → prosseguir normalmente

---

## METADADOS PADRÃO — LEITURA POR DEFAULT

Por padrão, Claude lê apenas metadados — nunca diff completo.

Campos extraídos por commit:

```
tipo          : extraído do prefixo [tipo] na linha de título
domínio       : extraído do campo (domínio) na linha de título
snapshot_ref  : extraído do campo snapshot: no corpo
agent_executor: extraído do campo agent-executor:
data          : timestamp do commit
hash          : primeiros 8 caracteres do SHA
```

O que NÃO é carregado por padrão:
- diff completo (`git diff` ou `git show -p`)
- arquivos alterados individualmente
- conteúdo das linhas modificadas
- histórico de branches ou tags

---

## QUANDO CARREGAR DIFF COMPLETO

Diff completo é carregado apenas em três situações explícitas:

**Situação 1 — Divergência detectada em etapa 7b**
Claude compara instrução autorizada ↔ execução de Codex.
Identificada divergência → carregar diff para detalhar o que divergiu.

**Situação 2 — Replay operacional solicitado**
Usuário ou Claude solicita reconstrução de ciclo histórico.
r-replay-operacional requer o diff para reconstituir a instrução.

**Situação 3 — Auditoria histórica explícita**
Usuário solicita auditoria de execuções passadas num domínio.
Claude carrega diff dos commits relevantes para análise.

Em todos os outros casos: apenas metadados.

---

## LIMITES DE LEITURA

| Limite | Valor | Motivo |
|---|---|---|
| Commits por domínio por execução | máximo 10 | contexto mínimo |
| Leituras de histórico por sessão | máximo 1 por domínio | sem carregamento repetido |
| Diffs completos por execução | máximo 2 | somente situações explícitas |
| Histórico total carregado | metadados apenas, por padrão | sem inflação de contexto |

Nunca carregar o histórico completo do repositório.
Nunca iterar sobre todos os commits para todos os domínios.
Contexto mínimo é princípio arquitetural inviolável.

---

## DETECÇÃO DE DRIFT OPERACIONAL

Claude identifica drift ao comparar snapshot ativo com histórico Git do domínio.

### Tipo 1 — Snapshot sem commit correspondente

```
Condição:
  snapshot.commit_hash = null
  snapshot.estado_atual = CONCLUÍDO

Sinalização:
  "Ciclo [ID] marcado como CONCLUÍDO sem evidência de commit (pré-v3.5 ou ciclo v3.0)"

Contexto:
  Ciclo foi concluído antes de v3.5 estar ativo, ou execução ocorreu
  sem Codex criar commit institucional.

Consequência operacional:
  Registrar ausência de evidência. Não bloquear ciclo atual.
  Se domínio está em análise: informar ao usuário que histórico
  de execução não é verificável para esse ciclo específico.
```

### Tipo 2 — Commit sem snapshot_ref

```
Condição:
  commit encontrado no domínio
  commit não possui campo snapshot: no corpo

Sinalização:
  "Commit [hash] no domínio [domínio] sem referência institucional (snapshot:)"

Contexto:
  Execução foi realizada sem ciclo institucional correspondente.
  Commit técnico, não institucional.

Consequência operacional:
  Sinalizar ao usuário antes de prosseguir com qualquer análise.
  Não usar esse commit como referência de ciclo.
  Claude deve identificar se o commit foi intencional ou acidental.
```

### Tipo 3 — Decisão mais recente que última execução

```
Condição:
  snapshot.data > último_commit_institucional.data
  snapshot.commit_hash = null

Sinalização:
  "Decisão [snapshot-ID] é mais recente que a última execução verificada.
   Possível ciclo pendente de execução."

Contexto:
  Uma decisão foi homologada mas não há evidência de execução posterior.
  Pode indicar: ciclo aguardando execução, execução sem commit, ou ciclo v3.0.

Consequência operacional:
  Verificar estado do snapshot: VALIDADO (pendente) ou CONCLUÍDO (v3.0 pré-Git)?
  Se VALIDADO: handoff pode ser re-emitido (ciclo interrompido).
  Se CONCLUÍDO: registrar como ciclo sem evidência — informar ao usuário.
```

### Tipo 4 — Múltiplos commits sem snapshot correspondente no mesmo domínio

```
Condição:
  ≥ 2 commits no domínio sem campo snapshot:

Sinalização:
  "Domínio [domínio] possui [N] commits não-institucionais. Auditoria recomendada."

Contexto:
  Padrão de execuções fora do protocolo C.A.O.S.

Consequência operacional:
  Sinalizar ao usuário. Incluir na análise atual como risco de drift arquitetural.
  Não bloquear ciclo. Não alterar os commits.
```

### Resultado da detecção de drift

Toda sinalização de drift produz três elementos:

```
tipo_drift:          [tipo identificado acima]
evidência:           [hash do commit ou ID do snapshot envolvido]
consequência:        [ação recomendada ao usuário]
bloqueia_execução:   false  ← nunca bloquear automaticamente
```

Drift detectado é informação para o usuário — não é gate automático.
Somente o usuário decide como proceder diante de drift identificado.

---

## FALLBACK v3.0

Quando r-git-operacional não está carregado:

- Etapa 0b não é executada
- Histórico Git não é lido
- Nenhuma detecção de drift é feita
- Sistema opera normalmente em modo v3.0

O fallback é degradação controlada, não falha.
Ciclos v3.0 são válidos sem leitura Git.

Condição de fallback:

```
se r-git-operacional NÃO carregado:
  → pular etapa 0b integralmente
  → prosseguir para etapa 1 (classificar tarefa)
```

---

## O QUE CLAUDE NÃO FAZ

Claude nunca:

- Cria commits
- Cria ou deleta branches
- Executa `git push` ou `git pull`
- Faz `git checkout` ou `git rebase`
- Altera o histórico do repositório de qualquer forma
- Usa histórico Git para substituir análise humana
- Bloqueia execução automaticamente com base em drift detectado
- Carrega histórico completo do repositório
- Mantém leituras Git entre execuções (leitura é por ciclo, não cumulativa)

---

## INTEGRAÇÃO COM r-estados-ciclo

A leitura Git informa o estado atual do domínio antes de Claude classificar:

```
Leitura Git na etapa 0b
  ↓
Claude identifica: último ciclo está em qual estado?
  → CONCLUÍDO com commit_hash: ciclo completo e verificado
  → CONCLUÍDO sem commit_hash: ciclo v3.0 sem evidência
  → VALIDADO: ciclo pendente de execução (retomável)
  → commit sem snapshot: execução não autorizada detectada
  ↓
Contexto disponível para análise na etapa 1
```

---

## PROIBIÇÕES

Nunca:
- Ler diff completo sem situação explícita autorizada
- Carregar mais de 10 commits por domínio por execução
- Usar commit sem snapshot_ref como evidência institucional
- Bloquear ou pausar ciclo automaticamente por drift
- Propagar leitura Git para domínios não identificados na etapa 0a
- Substituir snapshot por histórico Git quando snapshot existe

---

## RESULTADO ESPERADO

Quando r-git-operacional está carregado:
- Claude chega à etapa 1 com contexto histórico real do domínio
- Drift identificado é explicitamente apresentado ao usuário antes da proposta
- A análise incorpora: o que foi decidido (snapshot) + o que foi executado (Git)
- Ciclos sem evidência são identificados — não encobertos
