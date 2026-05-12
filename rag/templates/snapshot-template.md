# Snapshot — [domínio] — [id]

```yaml
# SNAPSHOT BASE (preencher este bloco)
id: [domínio-abreviado]-001
tipo: base
dominio: public.[tabela]
familia: [banco/segurança | banco/operacional | frontend | etc.]
data: YYYY-MM-DD

tarefa: [hotfix-rls | migration | auditoria | hotfix-frontend | etc.]
modulos_usados:
  - r/r-rls-padrao
  - k/banco/k-db-funcoes

decisao: |
  [Descrever a decisão arquitetural tomada.
   O que foi proposto, por que, e qual foi a abordagem escolhida.
   Ser específico — este texto deve ser compreensível sem contexto adicional.]

resultado: sucesso | falha | pendente
estado_atual: CONCLUÍDO | VALIDADO | EXECUTANDO

historico_estados:
  - estado: DETECTADO    timestamp: YYYY-MM-DDTHH:MM:SS
  - estado: ANALISADO    timestamp: YYYY-MM-DDTHH:MM:SS
  - estado: PROPOSTO     timestamp: YYYY-MM-DDTHH:MM:SS
  - estado: VALIDADO     timestamp: YYYY-MM-DDTHH:MM:SS
  - estado: EXECUTANDO   timestamp: YYYY-MM-DDTHH:MM:SS
  - estado: CONCLUÍDO    timestamp: YYYY-MM-DDTHH:MM:SS

riscos_vistos:
  - [risco 1 identificado durante a análise]
  - [risco 2]

riscos_ativos:
  - [riscos que NÃO foram resolvidos — ou "nenhum"]

artefatos_alterados:
  - [arquivo ou objeto alterado 1]
  - [arquivo ou objeto alterado 2]

agente_orquestrador: Claude
agente_executor: [Codex | operador humano | Claude]

# v3.5 — preencher quando aplicável
commit_hash: null
branch: null
```
