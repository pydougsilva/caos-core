# r-anti-burocracia
versao: 1.0

## OBJETIVO

Definir limites operacionais para prevenir que o C.A.O.S se torne
mais custoso que os problemas que resolve.

Responde à pergunta:
"isso está reduzindo caos operacional real ou apenas adicionando sofisticação?"

---

## PRINCÍPIO CENTRAL

O C.A.O.S existe para reduzir caos, não para criar outro tipo de caos.

Toda melhoria, módulo, protocolo ou estrutura nova deve responder:

> "Qual problema operacional real isso resolve?"

Se a resposta não for imediata e clara: não implementar.

---

## OS CINCO SINAIS DE HIPERTROFIA

### Sinal 1 — Módulos que nunca são carregados

Um módulo que não foi referenciado em nenhum ciclo real nos últimos 30 dias
é um candidato a arquivamento — não a refinamento.

Antes de criar um novo módulo:
verificar se existe algum que cobre o mesmo escopo.

### Sinal 2 — Protocolos que adicionam etapas sem adicionar valor

Toda etapa nova no fluxo operacional deve ter uma razão de existir
verificável em produção. Uma etapa que "faz sentido em teoria"
mas nunca foi necessária em um ciclo real é overhead, não governança.

### Sinal 3 — Documentação sobre documentação

Quando o sistema tem mais documentação sobre seus próprios módulos
do que conhecimento sobre o produto que ele gerencia,
a prioridade está invertida.

Razão saudável: ≤ 3 módulos operacionais por módulo de produto.

### Sinal 4 — Rituais que a equipe ignora

Se um protocolo (telemetria, lock, snapshot) é sistematicamente
ignorado nas sessões reais, ele não está funcionando.
Opções: simplificar ou remover — nunca reforçar com mais burocracia.

### Sinal 5 — Automação que precisa de automação para funcionar

Se manter um mecanismo requer ferramentas adicionais, scripts de manutenção
ou configuração específica de ambiente, ele está além da complexidade justificável
para o contexto de pequenos negócios.

---

## LIMITES SAUDÁVEIS DE COMPLEXIDADE

| Componente | Máximo recomendado | Ação quando ultrapassado |
|---|---|---|
| Módulos /r ativos | 20 | revisar com r-module-pruning |
| Módulos /k ativos | 20 | revisar com r-module-pruning |
| Linhas em TAREFA → MÓDULOS | 25 | consolidar linhas similares |
| Snapshots por domínio sem nova operação | 5 deltas | consolidar nova base |
| Commits por semana apenas sobre C.A.O.S | > 5 | parar e trabalhar no produto |
| Módulos criados sem ciclo correspondente | > 3 | parar e executar ciclos reais |

---

## O QUE NUNCA CRIAR

**Nunca criar módulo para:**
- Documentar o óbvio (o que o nome já diz)
- Cobrir edge cases que nunca aconteceram em produção
- Antecipar problemas que ainda são hipotéticos
- Substituir uma conversa curta com o usuário

**Nunca criar protocolo para:**
- Situações que acontecem menos de 1 vez por mês
- Processos que levam mais tempo para documentar do que executar
- Casos onde a solução é "perguntar ao usuário"

**Nunca criar automação para:**
- Tarefas que levam < 2 minutos manualmente
- Operações que requerem julgamento contextual
- Processos onde falha silenciosa é pior que ausência de automação

---

## CRITÉRIO OBRIGATÓRIO ANTES DE CRIAR QUALQUER MÓDULO

Responder as três perguntas:

```
1. Qual problema operacional REAL este módulo resolve?
   → Deve ser um problema que já aconteceu, não hipotético.

2. Sem este módulo, o que exatamente falharia?
   → Se a resposta for "nada imediatamente", adiar.

3. Este módulo é menor que o problema que resolve?
   → Se o módulo for mais complexo que o problema, não criar.
```

Se qualquer resposta for insatisfatória: não criar.
Registrar a ideia como nota de contexto e revisar após 2 semanas de operação real.

---

## QUANDO O SISTEMA ESTÁ SAUDÁVEL

O C.A.O.S está operacional e proporcional quando:

- Um novo agente entende o projeto em < 20 minutos
- Um ciclo completo (proposta → execução → snapshot) leva < 15 min de overhead
- A maioria dos módulos é usada em pelo menos 1 ciclo por mês
- O produto tem mais snapshots do que o C.A.O.S tem documentação sobre si mesmo
- Qualquer membro do projeto consegue explicar o C.A.O.S em 3 frases

---

## QUANDO SIMPLIFICAR SEM PEDIR PERMISSÃO

Se qualquer uma das condições abaixo for verdadeira, simplificar imediatamente:

- Dois módulos respondem à mesma pergunta de forma quase idêntica
- Um protocolo foi ignorado em 3 ciclos consecutivos
- Um módulo tem mais exemplos do que regras efetivas
- Um documento tem mais de 200 linhas e responde mais de 1 pergunta
- O index.md tem referência a arquivo que não existe

Simplificar não requer ciclo operacional completo.
Requer apenas: edição mínima + commit `[rag](simplificação): [o que foi simplificado]`.
