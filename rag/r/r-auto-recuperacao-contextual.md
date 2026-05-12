# r-auto-recuperacao-contextual
versao: 1.0

## OBJETIVO

Definir como o C.A.O.S detecta automaticamente domínios conhecidos
e dispara a recuperação de snapshots antes do carregamento do RAG.

Este módulo é a camada de detecção.
A camada de execução está em r-recuperacao-contextual.md.

---

## PRINCÍPIO CENTRAL

Antes de classificar a tarefa e carregar módulos:
verificar se o prompt menciona domínio com histórico de snapshot.

Se detecção positiva:
acionar r-recuperacao-contextual antes de qualquer outro módulo.

---

## POSIÇÃO NO FLUXO OPERACIONAL

Fluxo padrão do C.A.O.S:

  1. Classificar tarefa
  2. Consultar index.md
  3. Carregar /r → /k
  4. Executar

Com auto-recuperação ativa:

  0. Detectar domínio (este módulo)
  0a. Se match → acionar r-recuperacao-contextual
  1. Classificar tarefa
  2. Consultar index.md
  3. Carregar /r → /k
  4. Executar com contexto histórico ativo

A detecção ocorre na etapa zero — antes de qualquer carregamento.

---

## GATILHOS AUTOMÁTICOS

A detecção é disparada quando qualquer uma das condições abaixo
for verdadeira no prompt da sessão:

| Gatilho | Condição | Exemplo |
|---|---|---|
| G1 — Domínio explícito | nome de tabela ou componente mencionado textualmente | "audit_logs", "checkout", "profiles" |
| G2 — Classificação coincidente | tipo de tarefa coincide com tarefa de snapshot existente | "hotfix RLS" → snapshot de hotfix-rls existe |
| G3 — Regressão sinalizada | palavras-chave de reincidência presentes no prompt | "voltou", "novamente", "regressão", "antes funcionava", "o mesmo problema" |
| G4 — Auditoria de domínio conhecido | solicitação de auditoria em domínio com histórico | "audite audit_logs" → snapshot-001 existe |

Basta um gatilho ativo para iniciar o processo de matching.

---

## ALGORITMO DE MATCHING

Ao detectar gatilho, aplicar matching nesta ordem de prioridade:

### Nível 1 — Exact match
confidence_level: alto

Domínio mencionado textualmente == domínio de snapshot existente.

Ação: recuperar snapshot imediatamente via r-recuperacao-contextual.

---

### Nível 2 — Substring match
confidence_level: médio

Substring do domínio está presente no prompt.

Exemplos:
- "logs" → "public.audit_logs"
- "fornada" → "public.fornadas"
- "produto" → "public.produtos"

Condição obrigatória:
a substring deve corresponder a exatamente 1 domínio no registry de snapshots.

Se múltiplos domínios forem compatíveis com a substring:
não disparar recuperação automática — avançar para Nível 3.

Ação (somente com 1 domínio compatível): recuperar snapshot do domínio correspondente.

---

### Nível 3 — Classification match
confidence_level: baixo

Tipo de tarefa atual == tarefa registrada em snapshot existente.

Exemplos:
- tarefa classificada como "hotfix-rls" → existe snapshot com tarefa = hotfix-rls
- tarefa classificada como "policy-rls" → existe snapshot com tarefa = policy-rls

Ação: recuperar snapshot mais relevante pela hierarquia de r-recuperacao-contextual.

---

### Sem match
Nenhum dos níveis produziu correspondência.

Ação: prosseguir sem recuperação contextual — fluxo padrão.

---

## LIMITES DE DETECÇÃO

| Limite | Valor |
|---|---|
| Domínios avaliados por execução | máximo 3 candidatos |
| Snapshots recuperados por execução | máximo 1 (delegado a r-recuperacao-contextual) |
| Níveis de matching avaliados | todos os 3, na ordem definida |
| Substring match com múltiplos domínios | não dispara recuperação automática |
| Inferência sem menção explícita | proibida |

Nunca inferir domínio que não foi mencionado ou sinalizado no prompt.

---

## FALLBACK OPERACIONAL

| Situação | Ação |
|---|---|
| Match com resultado = sucesso | recuperar e usar normalmente |
| Match com resultado = falha sem resolução | recuperar, sinalizar ao usuário, não bloquear execução |
| Match com riscos_ativos preenchidos | recuperar, incluir riscos na análise da tarefa atual |
| Match ambíguo (múltiplos domínios) | avançar para próximo nível de matching; se persistir, sem recuperação |
| Nenhum match | prosseguir sem recuperação — registrar ausência de histórico |
| Erro na leitura do snapshot | log silencioso, prosseguir sem bloquear |

A auto-recuperação nunca bloqueia a execução.
Falha na detecção é fallback silencioso, não erro crítico.

---

## GOVERNANÇA HUMANA

A auto-recuperação não substitui validação humana.

O snapshot recuperado informa a análise.
Não autoriza execução automática.
A proposta gerada com base no histórico ainda requer aprovação explícita
antes de qualquer apply_migration, alter, drop ou mudança estrutural.

---

## COMPATIBILIDADE COM r-recuperacao-contextual

Este módulo é complementar, não substituto.

| Responsabilidade | Módulo |
|---|---|
| Detectar domínio automaticamente | r-auto-recuperacao-contextual (este) |
| Definir o que é snapshot | r-recuperacao-contextual |
| Hierarquia de prioridade entre snapshots | r-recuperacao-contextual |
| Limites de carregamento de snapshot | r-recuperacao-contextual |
| Registro e atualização de snapshots | r-recuperacao-contextual |

Quando este módulo detecta match, delega imediatamente para r-recuperacao-contextual.

---

## PROIBIÇÕES

Nunca:
- inferir domínio não mencionado no prompt
- recuperar snapshot de domínio diferente do detectado
- carregar mais de 1 snapshot por execução
- bloquear execução por falha na detecção
- substituir validação humana pelo histórico recuperado
- duplicar responsabilidades de r-recuperacao-contextual
- disparar Nível 2 com múltiplos domínios compatíveis

---

## RESULTADO ESPERADO

O C.A.O.S deve:
- detectar automaticamente domínios com histórico sem intervenção manual
- disparar recuperação contextual na etapa zero do fluxo operacional
- reduzir tempo de análise em domínios conhecidos
- sinalizar riscos_ativos pendentes antes da execução
- operar sem exigir que o usuário mencione snapshots anteriores explicitamente
