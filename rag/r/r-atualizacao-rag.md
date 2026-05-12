# r-atualizacao-rag
versao: C.A.O.S v1.1

## OBJETIVO

Definir como a memória operacional do sistema deve evoluir.

---

## PRINCÍPIO

O RAG:
não é documentação estática.

O RAG:
é memória operacional viva.

---

# REGRA PRINCIPAL

Toda alteração estrutural relevante:
deve atualizar o RAG.

---

# QUANDO ATUALIZAR

Atualizar módulos quando houver:

- nova feature
- alteração arquitetural
- mudança de fluxo
- novo padrão operacional
- alteração de schema
- nova policy
- nova integração
- mudança relevante de frontend

---

# QUANDO NÃO ATUALIZAR

Não atualizar:
para:
- pequenos textos
- refactors irrelevantes
- ajustes cosméticos
- logs temporários

---

# PRINCÍPIO DE ESCOLHA

Atualizar:
apenas módulos afetados.

Nunca:
reescrever o RAG inteiro.

---

# DETECÇÃO DE IMPACTO

Antes de atualizar:
identificar:

1. quais módulos foram afetados
2. se o index.md precisa ajuste
3. se houve sobreposição de escopo
4. se novos módulos são necessários

---

# CRIAÇÃO DE NOVOS MÓDULOS

Criar novo módulo apenas quando:

- o escopo atual ficou grande
- existem múltiplas responsabilidades
- há perda de precisão semântica
- há aumento excessivo de tokens

---

# PROIBIÇÕES

Nunca:
- duplicar conhecimento
- misturar /k com /r
- criar módulos genéricos demais
- transformar módulos em documentação monolítica

---

# VERSIONAMENTO

Todo módulo deve possuir:

versao: X.Y

Atualizar versão quando:
- comportamento mudar
- arquitetura mudar
- fluxo mudar

---

# STALE MODULES

Detectar módulos stale quando:

- App.jsx mudou significativamente
- schema mudou
- fluxo operacional mudou
- função SQL mudou
- roadmap mudou

---

# PRINCÍPIO DE COERÊNCIA

O RAG:
deve refletir:
o estado real do sistema.

Nunca:
estado imaginado.

---

# INDEX

Atualizar index.md quando:

- novos módulos forem criados
- taxonomia mudar
- novas skills surgirem
- novos domínios forem adicionados

---

# ECONOMIA COGNITIVA

Objetivo:
máxima precisão
com mínimo contexto.

---

# FLUXO OPERACIONAL

1. alteração ocorre
2. impacto é identificado
3. módulos afetados são localizados
4. atualização mínima é aplicada
5. index.md é revisado
6. versão é incrementada

---

# RESULTADO ESPERADO

O sistema deve:

- preservar coerência
- evitar degradação do RAG
- manter contexto confiável
- escalar sem caos documental
- permitir múltiplos agentes cooperando