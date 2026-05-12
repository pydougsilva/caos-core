# r-matching-conceito
versao: 1.0

## OBJETIVO

Definir o algoritmo de identificação de domínios operacionais
a partir de termos presentes no prompt, usando o registry de domínios
com scores ponderados por aliases.

Responde à pergunta:
"dado este prompt, qual domínio operacional está sendo referenciado?"

Substituição do matching heurístico (v2.2) por matching estruturado (v3.0).
Matching semântico por embeddings é reservado ao v4.0.

---

## PRINCÍPIO CENTRAL

Um domínio é identificado pelo peso acumulado de seus aliases no prompt.
Nenhum domínio é assumido sem evidência ponderada suficiente.
Ambiguidade não resolvida é sinalizada ao usuário — nunca auto-resolvida.

---

## QUANDO ATIVAR

Este módulo é ativado na etapa 0a do fluxo operacional,
após a detecção de gatilho (etapa 0 — r-auto-recuperacao-contextual).

Pré-requisito: k-sys-registry-dominios carregado na sessão.

Se k-sys-registry-dominios não estiver carregado:
→ não tentar matching estruturado
→ aplicar fallback v2.2 imediatamente (ver seção FALLBACK)

---

## ALGORITMO — CINCO PASSOS

### Passo 1 — Extração de termos candidatos

Extrair do prompt os termos substantivos com potencial de domínio:
nomes de tabelas, objetos de negócio, entidades, ações sobre dados.

Exemplos de termos candidatos:
- "audit_logs" → candidato
- "políticas de acesso" → candidato (termo composto)
- "logs" → candidato
- "corrija" → não é candidato (verbo de ação genérico)
- "o" / "que" / "de" → não são candidatos (stopwords)

Limite: máximo 10 termos candidatos por avaliação.
Se o prompt tiver mais de 10 candidatos, priorizar termos mais específicos
(nomes compostos e técnicos antes de termos genéricos).

---

### Passo 2 — Consulta ao registry

Para cada termo candidato:

```
1. Verificar exact match contra aliases de cada domínio
2. Se não houver exact match: verificar substring match
   (termo candidato contido em alias, ou alias contido em termo candidato)
3. Registrar: { termo, domínio, peso, tipo_match: exato | substring }
```

Tipo de match afeta o peso aplicado:

| Tipo de match | Peso aplicado |
|---|---|
| exato | peso do alias conforme registry |
| substring | peso do alias × 0.7 |

Exemplo:
- Alias "audit_logs" com peso 1.0
- Match exato: score += 1.0
- Match substring (ex: "audit_log"): score += 0.7

---

### Passo 3 — Agregação de scores por domínio

Para cada domínio que apareceu no Passo 2:

```
score(domínio) = soma dos pesos de todos os aliases correspondentes
```

Exemplo:
- Prompt: "verificar isolamento nos logs de auditoria"
- Termos candidatos: "logs", "auditoria"
- Matches:
    "logs"      → public.audit_logs, peso 0.8 (exact) → 0.80
    "auditoria" → public.audit_logs, peso 0.8 (exact) → 0.80
- Score(public.audit_logs) = 1.60

---

### Passo 4 — Ordenação e aplicação de regras

Ordenar domínios por score decrescente: [score_1, score_2, score_3, ...]

Aplicar regras na ordem abaixo. Parar na primeira regra que produzir resultado.

**Regra A — Match único:**
Somente um domínio tem score > 0.
→ Retornar esse domínio diretamente.

**Regra B — Alias exclusivo de alta confiança:**
O maior score contém match de alias com peso ≥ 0.8
que é exclusivo de um único domínio no registry.
→ Retornar esse domínio independente do segundo score.

Alias exclusivo = alias que aparece na lista de apenas um domínio no registry.
Termos como "fornadas" (peso 1.0) e "audit_logs" (peso 1.0) são exclusivos.
Termos como "logs" (peso 0.8 em audit_logs) podem ser exclusivos se não
aparecerem em outros domínios com peso ≥ 0.5.

**Regra C — Ratio de desambiguação:**
score_1 / score_2 ≥ 1.5
→ Retornar domínio com score_1.

Exemplo:
- score(public.audit_logs) = 1.60
- score(public.profiles) = 0.40
- Ratio: 1.60 / 0.40 = 4.0 ≥ 1.5 → retornar public.audit_logs

**Regra D — Mais de 3 domínios com score > 0:**
Prompt é muito amplo ou ambíguo.
→ Tratar como AMBÍGUO (ver Passo 5).

**Regra E — Nenhuma regra resolveu:**
score_1 / score_2 < 1.5 e sem alias exclusivo e ≤ 3 domínios.
→ Tratar como AMBÍGUO (ver Passo 5).

---

### Passo 5 — Resultado

**Resultado ÚNICO (Regras A, B ou C):**
```
domínio: public.audit_logs
score: 1.60
confiança: alta | média  (alta se Regra A ou B, média se Regra C)
tipo_match: exato | substring
```
→ Prosseguir com recovery de snapshot e fluxo normal.

**Resultado AMBÍGUO (Regra D ou E):**
```
ambiguidade: true
candidatos:
  - { domínio: public.X, score: 0.90 }
  - { domínio: public.Y, score: 0.75 }
mensagem_usuario: "Domínio ambíguo. Identificados dois candidatos:
  [X] e [Y]. Qual é o domínio alvo desta operação?"
```
→ Parar e aguardar esclarecimento do usuário.
→ Não prosseguir com recovery sem domínio confirmado.

**Resultado SEM MATCH:**
```
match: false
```
→ Aplicar FALLBACK v2.2 (ver seção abaixo).

---

## LIMITE DE AVALIAÇÃO

| Limite | Valor | Motivo |
|---|---|---|
| Termos candidatos por prompt | 10 | contexto mínimo |
| Domínios com score ativo | máximo 3 antes de AMBÍGUO | mais de 3 = prompt muito genérico |
| Domínios retornados | 1 | nunca múltiplos domínios simultâneos |
| Chamadas ao registry por execução | 1 | leitura única do arquivo |

---

## FALLBACK v2.2

Quando ativar:
- k-sys-registry-dominios não carregado
- Nenhum alias produziu match (resultado SEM MATCH)
- Ambiguidade não resolvida (após sinalização ao usuário sem resposta)

O que fazer:
→ Usar comportamento de r-auto-recuperacao-contextual:
   matching heurístico por presença de termo no prompt.

O fallback não é erro — é degradação controlada para comportamento v2.2.
Claude deve sinalizar internamente que operou sem matching estruturado.

---

## INTEGRAÇÃO COM k-sys-registry-dominios

Este módulo lê k-sys-registry-dominios para consultar aliases e pesos.

O que lê:
- `id` de cada domínio (para identificar o canônico)
- `aliases` com pesos de cada domínio
- `família` (para contexto na mensagem de ambiguidade)

O que não lê:
- `snapshots` (delegado a r-recuperacao-contextual)
- `módulos_padrão` (usado depois, na seleção de módulos)
- `embedding_path` (inativo até v4.0)

Após identificação do domínio:
- r-recuperacao-contextual recebe o `id` canônico para buscar snapshots
- index.md usa `módulos_padrão` do registry para sugerir carregamento

---

## EXEMPLOS

### Exemplo 1 — Match único por alias exclusivo

Prompt: "verificar políticas da fornada de sábado"

Termos candidatos: ["políticas", "fornada"]

Consulta registry:
- "políticas" → sem match direto nos aliases (alias de nenhum domínio)
- "fornada" → public.fornadas, peso 1.0 (exact)

Score(public.fornadas) = 1.0

Aplicar regras:
- Regra B: "fornada" peso 1.0 é alias exclusivo de public.fornadas
→ Resultado ÚNICO: public.fornadas, confiança alta

---

### Exemplo 2 — Desambiguação por ratio

Prompt: "o histórico de logs do sistema está mostrando acesso negado"

Termos candidatos: ["histórico", "logs", "acesso"]

Consulta registry:
- "histórico" → public.audit_logs, peso 0.5 (exact)
- "logs"      → public.audit_logs, peso 0.8 (exact)
- "acesso"    → sem match nos aliases do registry

Score(public.audit_logs) = 1.3
Nenhum outro domínio tem score > 0.

Regra A: match único.
→ Resultado ÚNICO: public.audit_logs, confiança alta

---

### Exemplo 3 — Ambiguidade sinalizada

Prompt: "verificar o plano do usuário"

Termos candidatos: ["plano", "usuário"]

Consulta registry:
- "plano"   → public.subscriptions, peso 0.8 (exact)
- "usuário" → public.profiles, peso 0.8 (exact)

Score(public.subscriptions) = 0.8
Score(public.profiles) = 0.8

Regra C: ratio = 0.8/0.8 = 1.0 < 1.5
Regra E: ambíguo — sem alias exclusivo

→ Resultado AMBÍGUO:
"Domínio ambíguo. Identificados dois candidatos:
[public.subscriptions] e [public.profiles].
Qual é o domínio alvo desta operação?"

---

### Exemplo 4 — Fallback v2.2

Prompt: "como funciona o faturamento?"

Termos candidatos: ["faturamento"]

Consulta registry: nenhum alias corresponde a "faturamento" (não está registrado).

Resultado SEM MATCH.
→ Fallback v2.2: r-auto-recuperacao-contextual usa matching heurístico.

---

## PROIBIÇÕES

Nunca:
- Auto-selecionar domínio quando resultado é AMBÍGUO
- Retornar mais de um domínio como resultado único
- Usar matching semântico (embeddings) — reservado a v4.0
- Carregar k-sys-registry-dominios mais de uma vez por execução
- Ignorar o fallback v2.2 quando sem match — deve ser aplicado explicitamente

---

## RESULTADO ESPERADO

O matching por conceito deve:
- identificar domínios com reprodutibilidade determinística
- sinalizar ambiguidade antes de agir
- degradar graciosamente para v2.2 quando sem match
- operar sem modificar o registry consultado
