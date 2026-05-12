# r-recuperacao-contextual
versao: 1.1

## OBJETIVO

Definir como o C.A.O.S recupera contexto histórico antes de novas execuções,
utilizando ciclos passados como memória operacional ativa.

---

## PRINCÍPIO CENTRAL

Antes de executar, perguntar:
este domínio já foi operado antes?

Se sim:
recuperar snapshot relevante antes de carregar módulos novos.

---

## O QUE É UM SNAPSHOT

Um snapshot é o registro compacto de um ciclo operacional homologado.

Contém:

| Campo         | Descrição                                               |
|---|---|
| id            | identificador sequencial por domínio (001, 002...)      |
| tarefa        | classificação do ciclo                                  |
| domínio       | tabela, componente ou fluxo afetado                     |
| módulos usados| lista de /r e /k carregados                             |
| decisão       | o que foi proposto e validado                           |
| resultado     | sucesso, falha ou pendente                              |
| data          | quando o ciclo foi executado                            |
| riscos vistos | riscos identificados durante a análise                  |
| riscos ativos | riscos ainda não resolvidos após o ciclo                |

---

## CRITÉRIOS DE RELEVÂNCIA

Recuperar snapshot quando:

1. o domínio alvo já apareceu em ciclo anterior (mesma tabela, mesmo componente)
2. a classificação da tarefa coincide com ciclo passado (ex: Policy RLS → já houve hotfix RLS)
3. há suspeita de regressão ou drift arquitetural

Não recuperar quando:

- domínio completamente novo, sem histórico
- tarefa de orientação geral sem execução
- ciclo anterior marcado como falha sem resolução documentada

---

## ORDEM DE RECUPERAÇÃO

1. verificar se existe snapshot para o domínio alvo
2. aplicar hierarquia de prioridade (ver seção abaixo)
3. carregar snapshot selecionado (contexto mínimo — apenas campos relevantes)
4. carregar /r necessários
5. carregar /k necessários
6. executar com contexto histórico ativo

A recuperação de snapshot precede o carregamento de módulos RAG.

---

## HIERARQUIA DE PRIORIDADE DE RECUPERAÇÃO

Quando múltiplos snapshots existem para o mesmo domínio, aplicar nesta ordem:

| Prioridade | Critério                                              | Motivo                                        |
|---|---|---|
| 1          | último snapshot com resultado = sucesso               | referência de estado válido mais recente      |
| 2          | snapshot mais recente com riscos_ativos preenchidos   | riscos pendentes exigem atenção antes de agir |
| 3          | snapshot com resultado = falha mais recente           | evitar repetir abordagem que não funcionou    |
| 4          | snapshot mais antigo como referência base             | contexto de origem do domínio                 |

Carregar apenas 1 snapshot por execução.
Em caso de empate de prioridade, usar o mais recente.

---

## LIMITES OPERACIONAIS

| Limite                      | Valor                            |
|---|---|
| Snapshots por execução      | máximo 1                         |
| Snapshots por domínio       | ilimitado (histórico acumulativo)|
| Módulos RAG por execução    | máximo 3                         |
| Campos do snapshot          | apenas os relevantes à tarefa    |
| Histórico retroativo        | guiado pela hierarquia acima     |

Nunca carregar múltiplos snapshots em paralelo na mesma execução.

---

## SNAPSHOTS EXISTENTES

### public.audit_logs — snapshot-001
- id: 001
- tarefa: hotfix-rls
- domínio: public.audit_logs
- módulos: r/r-rls-padrao, k/banco/k-db-funcoes
- decisão: substituir subquery email_admin por get_tenant_id() + is_tenant_admin(); adicionar cobertura is_platform_admin()
- resultado: sucesso
- data: 2026-05-09
- riscos vistos: cross-tenant por email, ausência platform_admin, performance subquery, inconsistência de padrão
- riscos ativos: nenhum

---

## REGRA DE APLICAÇÃO

Ao identificar nova tarefa em domínio com snapshot existente:

1. carregar snapshot de maior prioridade (ver hierarquia)
2. verificar se decisão anterior ainda é válida
3. checar riscos_ativos — se houver, incluir na análise atual
4. ajustar proposta com base no histórico
5. registrar novo snapshot ao final do ciclo homologado

---

## ATUALIZAÇÃO DE SNAPSHOTS

Após cada ciclo homologado e executado:

- criar novo snapshot no domínio (id sequencial)
- registrar resultado real (não esperado)
- preencher riscos_ativos com o que permanece pendente
- não editar snapshots anteriores — apenas acrescentar
- atualizar este arquivo e o index.md se necessário

---

## PROIBIÇÕES

Nunca:
- recuperar snapshot de domínio diferente do alvo
- usar snapshot com resultado = falha como referência positiva
- substituir validação humana por contexto histórico
- carregar mais de 1 snapshot por execução
- editar snapshots já registrados (imutabilidade do histórico)
- expandir snapshot além dos campos definidos

---

## RESULTADO ESPERADO

O C.A.O.S deve:
- operar com memória acumulada entre sessões
- reduzir redundância de análise em domínios conhecidos
- preservar decisões arquiteturais homologadas
- detectar regressões em domínios já operados
- escalar o histórico sem aumentar o contexto por execução
