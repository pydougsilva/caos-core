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
| A1.0 | A1.0-indiferenca-agente-auditoria.md | 2026-05-28 | auditoria_arquitetural | concluida | Auditoria da proposta "Indiferença de Agente" — veredicto: incorporação parcial com correções estruturais. Achado crítico: .claude/instructions.md incorreto para Claude Code |
| A1.1 | A1.1-codex-revisao-decisao-final.md | 2026-05-31 | revisao_independente + decisao_final | concluido | Revisão Codex + decisão final: INCORPORAÇÃO PARCIAL. Novo conceito: "Bootstrap Institucional Verificável" + context_receipt. Rastreabilidade: 6 commits verificados, pendência ciclo-hardening-legado-caos-core |
| DNR | dependencia-nominal-residual.md | 2026-05-25 | auditoria | registrado | Auditoria de referências nominais pós-hardening v5.0 — confirma eliminação de dependências nominais em contratos ativos |

---

## VALIDAÇÕES PLANEJADAS (promoção de afetoeforma)

Conforme k-sys-governanca-repositorios, os seguintes artefatos de afetoeforma
são protocolo-específicos e devem ser promovidos ao caos-core:

| ID | Arquivo (em afetoeforma) | Tipo | Prioridade |
|---|---|---|---|
| T0.1 | rag/docs/validacoes/T0.1-retomada-fria.md | protocolo | média |
| T0.1B | rag/docs/validacoes/T0.1B-retomada-pos-ajustes.md | protocolo | média |
| T2.1 | rag/docs/validacoes/T2.1-rejeicao-handoff-invalido.md | protocolo | média |
| T2.2 | rag/docs/validacoes/T2.2-aceitacao-handoff-valido.md | protocolo | média |
| CONV | rag/docs/validacoes/CONVENCOES.md | protocolo | alta |

Ação: ciclo separado de promoção. Não incluídos nesta sprint.

---

## REGRAS DE REGISTRO

1. Toda validação institucional relevante ao protocolo C.A.O.S (não ao produto Afeto em Forma)
   deve ser indexada aqui.
2. Validações de produto (T-MT.x, T4.x, T6.x) permanecem em afetoeforma.
3. Ao adicionar entrada: incluir ID, arquivo, data, tipo, estado e sumário de 1 linha.
4. Ao arquivar: mover o arquivo para rag/arquivo/validacoes/ e remover do índice.
