# CONVENCOES — Repositório de Validações C.A.O.S
versao: 1.0

## PROPÓSITO

Preservar descobertas arquiteturais verificáveis como memória institucional permanente.

Não é: sistema de QA, pipeline de testes, substituto de telemetria, substituto de snapshots.
É: registro de hipóteses comprovadas, refutadas ou homologadas — com evidência rastreável.

---

## O QUE MERECE REGISTRO

Registrar quando:
- Hipótese arquitetural foi testada e produziu aprendizado não-óbvio
- Regressão foi encontrada em artefato que estava funcionando
- Protocolo foi validado empiricamente pela primeira vez
- Mudança em módulo foi feita como resultado direto de um teste
- Padrão identificado pode futuramente ser promovido ao caos-core

---

## O QUE NÃO MERECE REGISTRO

Não registrar:
- Resultados sem implicação arquitetural
- Ciclos operacionais normais → pertencem ao snapshot
- Detalhes de execução → pertencem à telemetria
- Análises sem conclusão → pertencem ao contexto conversacional
- Repetições de descobertas já registradas

---

## FORMATO DO ARQUIVO

Frontmatter:
```
id: [prefixo][número]-[nome-curto]
tipo: protocolo | artefato | modulo | multiagente | degradacao | produto
estado: experimental | em-teste | validado | refutado | homologado | deprecated
data: YYYY-MM-DD
hipotese: "[uma frase — o que estava sendo verificado]"
resultado: validado | validado-parcialmente | refutado | homologado | em-avaliacao
```

Seções:

**DESCOBERTA** — o que foi encontrado (máx 5 linhas)
**IMPACTO ARQUITETURAL** — o que mudou ou deve mudar (máx 3 linhas)
**REFERÊNCIAS** — telemetria, commit, módulos, snapshot
**CAOS-CORE** — padrão promotável: sim/não + qual, se sim

Limite absoluto: 50 linhas por arquivo.

---

## ESTADOS

| Estado | Significado |
|---|---|
| experimental | Hipótese formulada, teste não iniciado |
| em-teste | Teste em andamento |
| validado | Confirmado com evidência — mudança pendente |
| refutado | Hipótese falhou — aprendizado documentado |
| homologado | Validado E mudança aplicada e commitada |
| deprecated | Supersedido por validação mais recente |

---

## CONVENÇÃO DE NOMENCLATURA

Prefixo por categoria:
- `T` → teste de protocolo ou handoff (T0.1, T2.1...)
- `H` → hipótese arquitetural pura
- `R` → regressão documentada
- `P` → padrão candidato a promoção no caos-core

Formato: `[prefixo][número]-[nome-curto].md`

Exemplos:
- `T0.1-retomada-fria.md`
- `R001-contexto-produto-perdido.md`
- `P001-locks-verificados.md`

---

## MODO DE EXECUÇÃO — HANDOFFS E PROMPTS FASE 5

Todo prompt ou handoff emitido para agente_executor deve incluir ao final:

```
---
MODO DE EXECUÇÃO: [modo]

MOTIVO:
- [razão operacional principal]
- [o que deve ou não estar no contexto do executor]
```

**NOVA SESSÃO FRIA**
Zero contexto conversacional anterior.
Usar quando: teste mede continuidade cognitiva, T_ret real, ou ausência de contaminação.

**NOVA SESSÃO APÓS LEITURA DOS ARTEFATOS**
Contexto limpo, mas executor lê artefatos específicos antes do handoff.
Usar quando: ciclo real onde o executor precisa do snapshot do domínio antes de executar.

**SESSÃO ATUAL**
Continuidade explícita do contexto da sessão em andamento.
Usar quando: teste depende de estado conversacional previamente construído
(ex: T2.2 que segue T2.1 — agente_executor já conhece o handoff inválido e recebe o válido).

Esta convenção é obrigatória a partir da Fase 5 para todos os prompts de teste e handoffs formais.

---

## LIMITES ANTI-HIPERTROFIA

- Máximo 20 arquivos neste diretório (excl. CONVENCOES.md e index-validacoes.md)
- Máximo 50 linhas por arquivo de validação
- Nenhum subdiretório
- Arquivo sem referência a commit ou telemetria após 90 dias: candidato a `deprecated`
- Ao atingir 15 arquivos: aplicar r-module-pruning aqui antes de criar novos

---

## VÍNCULO COM OUTROS ARTEFATOS

| Artefato | Papel |
|---|---|
| Telemetria | registra quem executou, como, confiança |
| Snapshot | registra o que foi decidido |
| Git commit | registra o que foi executado |
| **Validação** | **registra o que foi aprendido arquiteturalmente** |

Cada validação deve referenciar pelo menos telemetria OU commit.
Validação sem referência = especulação, não memória institucional.

---

## CAMINHO PARA CAOS-CORE

Uma validação `homologado` pode ser proposta ao caos-core quando:
1. O padrão aparece em 2+ validações distintas
2. Sem referências específicas ao projeto Afeto em Forma
3. Validado em 5+ ciclos reais
4. Promoção requer gate humano explícito (r-continuidade-cognitiva)

Nunca promover automaticamente.
Nunca promover antes de 5 ciclos reais.
Nunca promover padrão com dependência de domínio específico.
