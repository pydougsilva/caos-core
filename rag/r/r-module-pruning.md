# r-module-pruning
versao: 1.0

## OBJETIVO

Definir critérios claros para arquivamento, descontinuação,
consolidação e simplificação de módulos RAG.

Responde à pergunta:
"quando e como reduzir a memória institucional para que ela
continue precisa e utilizável?"

---

## PRINCÍPIO CENTRAL

Um RAG que cresce indefinidamente se torna inutilizável.
Módulos stale são mais custosos do que módulos ausentes.
Simplicidade é uma feature — não um trade-off.

---

## QUATRO AÇÕES POSSÍVEIS

### ARQUIVAR

Mover módulo para `rag/arquivo/` preservando histórico.

**Quando arquivar:**
- módulo descreve feature, fase ou padrão que foi completamente substituído
- o domínio que o módulo cobre foi removido do produto
- módulo era experimental e o experimento foi encerrado

**Como arquivar:**
1. mover arquivo para `rag/arquivo/[tipo]/[nome].md`
2. adicionar nota no topo: `status: ARQUIVADO | data: [YYYY-MM-DD] | motivo: [texto]`
3. remover do index.md (MÓDULOS EXISTENTES e TAREFA → MÓDULOS)
4. criar commit `[rag](arquivo): arquivar [nome] — [motivo breve]`

---

### DESCONTINUAR

Marcar módulo como obsoleto sem remover imediatamente.

**Quando descontinuar:**
- módulo referencia padrão LEGADO mas ainda pode ter valor histórico
- módulo está sendo substituído por outro mais recente em construção
- módulo tem informação parcialmente correta e substituição ainda não está pronta

**Como descontinuar:**
1. adicionar ao topo do arquivo:
   ```
   status: DESCONTINUADO
   substituido_por: [nome-do-novo-módulo ou "em construção"]
   data: [YYYY-MM-DD]
   motivo: [texto]
   ```
2. manter no index.md mas adicionar `[DESCONTINUADO]` ao lado do nome
3. não usar como referência primária em novos ciclos

---

### CONSOLIDAR

Fundir dois ou mais módulos em um único quando há sobreposição.

**Quando consolidar:**
- dois módulos respondem à mesma pergunta com abordagens levemente diferentes
- um módulo cresceu e seu escopo se tornou idêntico a outro
- manutenção dos dois separados gera inconsistência

**Como consolidar:**
1. criar novo módulo consolidado com nome canônico
2. arquivar os originais referenciando o novo
3. atualizar index.md
4. commit `[rag](consolidação): unificar [A] + [B] em [C]`

---

### SIMPLIFICAR

Reduzir conteúdo de módulo sem alterar seu propósito.

**Quando simplificar:**
- módulo cresceu além de sua pergunta original
- módulo acumulou regras que pertencem a outros módulos
- módulo tem exemplos redundantes que ocupam contexto sem adicionar valor

**Como simplificar:**
- editar o arquivo diretamente
- incrementar versão
- commit `[rag](simplificação): reduzir escopo de [nome]`

---

## SINAIS DE ALERTA — QUANDO AUDITAR

Auditar os módulos quando:

```
- novo agente assume o projeto e encontra ambiguidade nos módulos
- index.md referencia módulo que não existe no filesystem
- dois módulos têm objetivos sobrepostos detectados em ciclo real
- módulo referencia arquivo do produto que foi renomeado ou deletado
- ciclo real ignorou um módulo "obrigatório" porque estava inaplicável
```

---

## MÓDULOS PROBLEMÁTICOS CONHECIDOS (2026-05-12)

| Módulo | Problema | Ação recomendada |
|---|---|---|
| `hotfix-padrao.md` (sem r-) | duplicata de r-hotfix-padrao.md com conteúdo diferente | Arquivar |
| `r-design-padrao` | referenciado no index.md mas não existe | Criar stub ou remover referência |
| `r-relatorio-padrao` | referenciado no index.md mas não existe | Criar stub ou remover referência |
| `k/frontend/k-fe-admin-*` | wildcard irresolvível no index.md | Remover referência ou criar arquivos concretos |

---

## FREQUÊNCIA DE REVISÃO

Revisão de módulos recomendada:
- a cada troca de agente (entrada de novo executor)
- após cada 10 ciclos operacionais
- quando um ciclo falhar por módulo stale ou ausente
- quando o produto avançar uma fase major

---

## LIMITE RECOMENDADO DE MÓDULOS

Sinal de alerta: mais de 20 módulos /r ativos.
Sinal de alerta: mais de 20 módulos /k ativos.
Sinal de alerta: tabela TAREFA → MÓDULOS com mais de 20 linhas.

Acima desses limites, revisar antes de adicionar novos.

---

## O QUE NUNCA FAZER

Nunca:
- deletar módulo sem arquivar (perde histórico)
- criar novo módulo sem verificar se já existe um cobrindo o mesmo escopo
- deixar módulos fantasma no index.md (referências sem arquivo)
- simplificar um módulo removendo informação que ainda é a única fonte daquele conhecimento
