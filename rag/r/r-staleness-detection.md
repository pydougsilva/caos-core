# r-staleness-detection
versao: 1.0

## OBJETIVO

Definir como detectar quando snapshots, módulos RAG e artefatos
de produto podem estar desatualizados em relação ao estado real do sistema.

Responde à pergunta:
"este artefato ainda reflete a realidade do sistema?"

---

## PRINCÍPIO CENTRAL

Artefatos desatualizados são mais perigosos do que artefatos ausentes.
Um módulo ausente falha explicitamente.
Um módulo stale produz confiança incorreta.

---

## TRÊS TIPOS DE STALENESS

### Tipo 1 — Staleness de Snapshot

Indicadores de que um snapshot pode não refletir o estado atual do domínio:

```
SUSPEITO quando:
  snapshot.data < data atual por mais de 30 dias
  E houve commits no domínio após snapshot.data (detectável via r-git-operacional)

SUSPEITO quando:
  snapshot.estado_atual = CONCLUÍDO
  E snapshot.commit_hash = null (pré-v3.5)
  E arquivos do domínio foram alterados fora do C.A.O.S

DEFINITIVAMENTE STALE quando:
  policy RLS, tabela ou função referenciada na decisão foi alterada
  sem ciclo correspondente no snapshot
```

**Consequência:** carregar snapshot mas sinalizar ao usuário a suspeita de staleness antes de usar como base de análise.

---

### Tipo 2 — Staleness de Módulo RAG

Indicadores de que um módulo /r ou /k pode estar desatualizado:

```
SUSPEITO quando:
  módulo.versao.data < data de mudança arquitetural relevante
  (ex: k-proj-identidade v5.3 mas produto está em fase diferente)

SUSPEITO quando:
  módulo referencia arquivo que mudou significativamente
  (ex: k-db-funcoes menciona fn_provision_tenant como "planejada"
   mas a função já existe em supabase/functions/)

SUSPEITO quando:
  módulo /k descreve schema diferente do que o banco tem atualmente
  (verificável via r-git-operacional ou auditoria MCP)

DEFINITIVAMENTE STALE quando:
  módulo referencia função, tabela ou padrão marcado como LEGADO
  em outro módulo mais recente
```

**Consequência:** usar o módulo mas documentar suspeita no snapshot da sessão.

---

### Tipo 3 — Staleness de Produto

Indicadores de que o knowledge de produto pode não refletir o App.jsx atual:

```
SUSPEITO quando:
  versão no cabeçalho do App.jsx ≠ versão em k-proj-identidade
  (ex: "// AfetoEmForma v5.3" vs módulo v5.3 → coincide, mas precisa verificar se fase mudou)

SUSPEITO quando:
  k-proj-identidade lista "próximas entregas" que parecem já implementadas
  (verificável lendo App.jsx ou migrações SQL)

SUSPEITO quando:
  módulo de frontend descreve comportamento que não corresponde
  ao que Claude observa no código atual
```

**Consequência:** atualizar o módulo antes de usá-lo como referência para qualquer ciclo.

---

## VERIFICAÇÃO RÁPIDA — CHECKLIST DE ENTRADA

Antes de iniciar qualquer ciclo num domínio com histórico:

```
□ 1. snapshot.data: há quanto tempo foi criado?
      → > 30 dias: sinalizar suspeita

□ 2. git log -- [arquivos_do_domínio] após snapshot.data:
      → commits existem: snapshot pode estar stale

□ 3. módulos carregados: versão vs última mudança conhecida?
      → divergência: documentar antes de usar

□ 4. arquivos referenciados no snapshot ainda existem com mesmo nome?
      → renomeados ou deletados: snapshot definitivamente stale

□ 5. decisão do snapshot usa funções/patterns que o RAG marca como LEGADO?
      → usar com cautela, sinalizar ao usuário
```

Não bloquear o ciclo. Registrar suspeitas no snapshot da sessão.

---

## QUANDO FORÇAR REAVALIAÇÃO

Reavaliação obrigatória (novo ciclo de análise antes de executar):

- Policy RLS do domínio foi alterada fora do C.A.O.S
- Migração SQL foi executada sem snapshot correspondente
- Módulo /k principal do domínio está marcado como stale
- Versão do App.jsx no cabeçalho diverge do k-proj-identidade

---

## LIMITES

Este módulo NÃO:
- substitui análise humana de staleness
- garante que detectou TODA desatualização
- é executado automaticamente sem gate humano
- opera como monitor contínuo

Staleness detection é um ponto de verificação na entrada do ciclo,
não um sistema de monitoramento.

---

## RESULTADO ESPERADO

Ao final da etapa 0b, Claude deve ser capaz de dizer:
"Este snapshot parece atual" ou "Este snapshot tem indicadores de staleness: [lista]"
Nunca: silêncio sobre a questão.
