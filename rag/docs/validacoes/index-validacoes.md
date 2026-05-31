# index-validacoes — C.A.O.S Core
versao: 1.0
data: 2026-05-28

## OBJETIVO

Índice de todas as validações institucionais registradas no caos-core.

Máximo recomendado: 20 entradas.
Ao atingir 20: arquivar as mais antigas em rag/arquivo/ antes de adicionar novas.

---

## CATEGORIAS

| Prefixo | Categoria | Descrição |
|---|---|---|
| T0.x | Protocolo — bootstrap | Validações do ciclo de inicialização e retomada |
| T2.x | Protocolo — handoff | Validações do protocolo de handoff agente→agente |
| A1.x | Arquitetural — auditoria | Auditorias de propostas arquiteturais externas ou internas |

---

## VALIDAÇÕES REGISTRADAS

| ID | Arquivo | Data | Tipo | Estado | Sumário |
|---|---|---|---|---|---|
| T0.1 | T0.1-retomada-fria.md | 2026-05-12 | protocolo | homologado | Retomada fria com 3 artefatos: T_ret=12min, 4 gaps estruturais confirmados, r-telemetria-cognitiva v1.0→v1.1 |
| T0.1B | T0.1B-retomada-pos-ajustes.md | 2026-05-13 | protocolo | homologado | 4/4 gaps convertidos INFERIDA→DIRETA, T_ret=8min (−33%). Padrão crítico: DIRETA+Alta Confiança ≠ Correto |
| T2.1 | T2.1-rejeicao-handoff-invalido.md | 2026-05-13 | protocolo | homologado | 3/3 falhas detectadas com precisão — protocolo de rejeição validado empiricamente |
| T2.2 | T2.2-aceitacao-handoff-valido.md | 2026-05-13 | protocolo | homologado | 10/10 itens passam — protocolo de aceitação validado. T2.1+T2.2 = par inseparável |
| CONV | CONVENCOES.md | — | referencia | ativo | Convenções do repositório de validações: formato, estados, nomenclatura, limites, Modo de Execução |
| A1.0 | A1.0-indiferenca-agente-auditoria.md | 2026-05-28 | auditoria_arquitetural | concluida | Auditoria da proposta "Indiferença de Agente" — veredicto: incorporação parcial com correções estruturais. Achado crítico: .claude/instructions.md incorreto para Claude Code |
| A1.1 | A1.1-codex-revisao-decisao-final.md | 2026-05-31 | revisao_independente + decisao_final | concluido | Revisão Codex + decisão final: INCORPORAÇÃO PARCIAL. Novo conceito: "Bootstrap Institucional Verificável" + context_receipt. Rastreabilidade: 6 commits verificados |
| DNR | dependencia-nominal-residual.md | 2026-05-25 | auditoria | registrado | Auditoria de referências nominais pós-hardening v5.0 — confirma eliminação de dependências nominais em contratos ativos |

---

## VALIDAÇÕES PLANEJADAS

Todos os artefatos T0.x, T2.x e CONVENCOES foram promovidos na Sprint v5.1 (2026-05-31).
Nenhuma promoção pendente no momento.

---

## REGRAS DE REGISTRO

1. Toda validação institucional relevante ao protocolo C.A.O.S (não ao produto Afeto em Forma)
   deve ser indexada aqui.
2. Validações de produto (T-MT.x, T4.x, T6.x) permanecem em afetoeforma.
3. Ao adicionar entrada: incluir ID, arquivo, data, tipo, estado e sumário de 1 linha.
4. Ao arquivar: mover o arquivo para rag/arquivo/validacoes/ e remover do índice.
