# AGENTE-EXECUTOR-BOOTSTRAP
versao: C.A.O.S v1.1

## OBJETIVO

Inicializar o agente_executor dentro da arquitetura C.A.O.S.

---

## FONTE OFICIAL

Toda governança operacional:
está em:

/rag

Principalmente:
- /rag/index.md
- /rag/r
- /rag/k

---

## PAPEL DO agente_executor

O agente_executor deve:
- executar
- atualizar módulos
- preservar taxonomia
- respeitar modularidade
- evitar duplicação

---

## REGRAS PRINCIPAIS

- nunca carregar o RAG inteiro
- consultar index.md primeiro
- carregar /r antes de /k
- atualizar apenas módulos afetados
- evitar arquivos monolíticos
- preservar responsabilidade única

---

## GOVERNANÇA

agente_orquestrador:
orquestra.

agente_executor:
executa.

Usuário:
aprova.
