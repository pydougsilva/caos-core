# k-bootstrap-caos
versao: 1.0

## OBJETIVO

Guia de inicialização do C.A.O.S em projetos novos, projetos legados
e projetos sem governança organizada.

Responde à pergunta:
"como começar a usar o C.A.O.S num projeto real?"

---

## PRINCÍPIO DE ADOÇÃO

O C.A.O.S é adotado incrementalmente — não instalado de uma vez.

Começar com o mínimo absoluto e adicionar capacidades conforme
a necessidade operacional real se manifesta.

**Nunca inicializar tudo de uma vez.**
O overhead de setup deve ser proporcional ao problema que resolve.

---

## TRÊS PERFIS DE ADOÇÃO

### Perfil A — Projeto Novo (greenfield)

Projeto começando do zero. Nenhuma história técnica existe.
**Overhead de bootstrap: ~2 horas.**

### Perfil B — Projeto Legado com Git organizado

Projeto existente com histórico Git razoável e documentação parcial.
**Overhead de bootstrap: ~4 horas.**

### Perfil C — Projeto Legado sem governança

Projeto existente sem Git organizado, sem documentação, código monolítico.
**Overhead de bootstrap: 1-2 dias distribuídos.**

---

## CHECKLIST — PERFIL A: PROJETO NOVO

```
□ 1. Inicializar repositório Git:
      git init
      git branch -m master main

□ 2. Criar .gitignore com proteções mínimas:
      node_modules/, dist/, .env, rag/locks/

□ 3. Criar estrutura RAG mínima:
      mkdir -p rag/r rag/k/projeto rag/k/sistema rag/k/banco rag/k/frontend
      mkdir -p rag/docs/ciclos/telemetria rag/locks rag/arquivo/r

□ 4. Copiar núcleo mínimo do C.A.O.S (ver k-sys-nucleo-minimo)

□ 5. Preencher AGENTS.md com identidade do projeto

□ 6. Preencher k-proj-identidade.md com:
      - nome do projeto
      - tipo e conceito central
      - stack atual
      - fase atual
      - próximas entregas

□ 7. Preencher k-sys-registry-dominios.md com domínios iniciais conhecidos

□ 8. Commit de baseline:
      git add .
      git commit -m "[snapshot](sistema): baseline inicial [nome-projeto]"

□ 9. Registrar telemetria de abertura (r-telemetria-cognitiva)
```

**Duração estimada por etapa:** 5-15 minutos cada.

---

## CHECKLIST — PERFIL B: PROJETO LEGADO COM GIT

```
□ 1. Verificar branch principal e garantir que é "main"
      git branch -m master main (se necessário)

□ 2. Criar .gitignore se ausente ou incompleto
      Verificar: .env não está rastreado

□ 3. Adicionar estrutura RAG ao projeto existente:
      mkdir -p rag/r rag/k/projeto rag/k/sistema rag/docs/ciclos/telemetria rag/locks

□ 4. Copiar núcleo mínimo (ver k-sys-nucleo-minimo)

□ 5. Preencher AGENTS.md com estado atual do projeto
      Importante: documentar tecnologias, fase e padrões EXISTENTES

□ 6. Criar k-proj-identidade.md refletindo o estado atual (não o ideal)

□ 7. Mapear domínios existentes em k-sys-registry-dominios.md
      Para cada tabela/componente principal: criar entrada no registry

□ 8. Criar snapshot de auditoria inicial:
      Executar r-staleness-detection sobre artefatos existentes
      Documentar o que foi encontrado

□ 9. Identificar o domínio mais crítico e executar primeiro ciclo real
      Objetivo: validar que o protocolo funciona ANTES de documentar tudo

□ 10. Registrar telemetria de entrada

□ 11. Commit de baseline com estado atual documentado
```

**Armadilha comum:** tentar documentar TODO o histórico antes de executar o primeiro ciclo.
Resistir. Executar um ciclo real primeiro — ele revelará o que precisa ser documentado.

---

## CHECKLIST — PERFIL C: PROJETO LEGADO SEM GOVERNANÇA

Esta é a situação mais comum e a que mais beneficia do C.A.O.S.
Também é a mais arriscada para começar com tudo de uma vez.

**Abordagem recomendada: bootstrapping incremental em 3 sessões.**

### Sessão 1 — Salvar o que existe (2-4 horas)

```
□ Inicializar Git se não existir
□ Criar .gitignore com proteções de segurança (.env obrigatório)
□ Primeiro commit: captura o estado atual
□ Criar AGENTS.md mínimo (pode ser incompleto — será atualizado)
□ Registrar telemetria desta sessão
```

### Sessão 2 — Mapear o que é crítico (2-4 horas)

```
□ Identificar os 3 domínios mais críticos do projeto
□ Criar k-proj-identidade.md
□ Criar k-sys-registry-dominios.md com os 3 domínios identificados
□ Para o domínio mais crítico: criar primeiro snapshot (mesmo que incompleto)
□ Identificar o maior risco arquitetural e criar snapshot de auditoria
```

### Sessão 3 — Primeiro ciclo real (1-3 horas)

```
□ Executar um ciclo operacional real no domínio mais crítico
□ Usar o protocolo completo: proposta → validação → handoff → execução → snapshot
□ Validar que o sistema funciona antes de expandir
□ Registrar telemetria e ajustar AGENTS.md com aprendizados
```

**Após as 3 sessões:** o sistema está funcionando minimamente. Adicionar capacidades conforme necessário.

---

## O QUE NÃO FAZER NO BOOTSTRAP

**Não fazer:**
- Documentar toda a arquitetura antes de executar um ciclo
- Criar todos os módulos RAG possíveis antes de saber o que é necessário
- Migrar 100% do histórico para snapshots de uma vez
- Parar o desenvolvimento do produto para "instalar o C.A.O.S"

**Fazer:**
- Executar um ciclo real nas primeiras 2 horas
- Documentar por demanda — quando o ciclo pedir
- Começar com o domínio de maior risco, não com o mais simples
- Aceitar que o C.A.O.S vai evoluir junto com o projeto

---

## CRITÉRIO DE MATURIDADE

O bootstrap está completo quando:

```
□ Um novo agente consegue entender o projeto em < 20 minutos
□ Existe pelo menos 1 snapshot operacional real
□ index.md não tem referências fantasma
□ AGENTS.md descreve o projeto atual (não o ideal)
□ Git tem pelo menos 1 commit institucional com o formato correto
```

Não existe bootstrap "perfeito". O sistema amadurece com uso.

---

## TEMPLATES DISPONÍVEIS

Ver diretório `rag/templates/` para:
- AGENTS.md template
- k-proj-identidade.md template
- snapshot-exemplo.md
- telemetria-exemplo.md
- handoff-exemplo.md
