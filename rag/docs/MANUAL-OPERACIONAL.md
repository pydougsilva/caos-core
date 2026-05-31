# C.A.O.S — Manual Operacional

versao: 1.0
data: 2026-05-12
audiencia: operadores humanos, novos agentes, desenvolvedores que adotam o sistema

---

## O QUE É O C.A.O.S

O C.A.O.S (Cognitive Autonomous Operational System) é um middleware de continuidade cognitiva.

Em termos práticos: é um conjunto de arquivos Markdown, YAML e Git que preserva
o contexto operacional de um projeto de software mesmo quando o agente de IA muda de sessão,
o desenvolvedor muda de ambiente, ou o executor muda de ferramenta.

**O C.A.O.S não é:**
- Um framework de código
- Um plugin
- Uma plataforma
- Um sistema autônomo

**O C.A.O.S é:**
- Um protocolo de trabalho com IA que preserva continuidade
- Um conjunto de regras para estruturar decisões e execuções
- Um sistema de memória institucional baseado em arquivos legíveis

---

## QUANDO USAR

O C.A.O.S agrega valor quando:

- O projeto tem mais de uma sessão de trabalho com IA por semana
- Há troca de ferramentas de IA (agente_orquestrador → agente_executor → agente_orquestrador)
- O projeto tem múltiplos domínios interdependentes (banco, frontend, auth)
- Há risco de perder decisões arquiteturais entre sessões
- Mais de uma pessoa usa IA no projeto

**Quando NÃO vale a pena:**
- Projetos de duração < 2 semanas
- Protótipos descartáveis
- Experimentação exploratória sem intenção de manutenção
- Projetos onde o overhead de documentação supera o benefício

---

## CUSTO OPERACIONAL ESPERADO

Com o núcleo mínimo ativo:

| Operação | Overhead estimado |
|---|---|
| Leitura inicial de contexto (novo agente) | 10-20 min |
| Registro de telemetria por sessão | 5-10 min |
| Criação de snapshot após ciclo | 5-15 min |
| Ciclo completo com handoff formal | +15-20 min vs. ciclo informal |

O C.A.O.S compensa quando ciclos informais (sem memória) custam mais em reconstrução
do que os ciclos formais custam em overhead.

---

## FLUXO OPERACIONAL PADRÃO

```
1. Ler AGENTS.md             → entender o projeto
2. Ler rag/index.md          → mapear capacidades disponíveis
3. Verificar rag/locks/      → há ciclo ativo?
4. Identificar domínio       → qual é o alvo?
5. Recuperar snapshot        → há histórico?
6. Carregar módulos (/r, /k) → máximo 3 por ciclo
7. Propor instrução          → agente_orquestrador analisa e propõe
8. Validar com usuário       → gate obrigatório
9. Executar                  → agente_executor ou operador humano
10. Registrar snapshot       → preservar decisão
11. Registrar telemetria     → preservar contexto da sessão
```

---

## ONBOARDING HUMANO

Para desenvolvedores que ainda não usam o C.A.O.S:

**30 minutos de onboarding:**
1. Ler AGENTS.md do projeto (5 min)
2. Ler rag/index.md (5 min)
3. Ler um snapshot existente (5 min)
4. Executar um ciclo simples com agente_orquestrador usando o protocolo (15 min)

Após o primeiro ciclo real, o sistema começa a fazer sentido.
Documentação adicional é lida por demanda.

**Erro comum:** tentar entender todo o C.A.O.S antes de executar o primeiro ciclo.
Não fazer isso. Executar primeiro, entender depois.

---

## ONBOARDING DE NOVO AGENTE

Ver k-sys-handoff-institucional para o protocolo completo.

Resumo:
1. Ler AGENTS.md
2. Ler rag/index.md
3. Verificar locks ativos
4. Verificar histórico Git
5. Aplicar r-staleness-detection
6. Registrar telemetria de entrada com nível de confiança

---

## TROUBLESHOOTING INSTITUCIONAL

### "Não sei em que fase o projeto está"

→ Ler k-proj-identidade.md
→ Se estiver stale: verificar telemetria mais recente e snapshots de domínio

ATENÇÃO: NÃO usar App.jsx ou UI como fonte de verdade sobre estado do produto.
A UI representa contexto VISUAL, não contexto operacional.
Ver seção "OS QUATRO CONTEXTOS" abaixo.

---

## OS QUATRO CONTEXTOS — PRINCÍPIO DE ISOLAMENTO COGNITIVO

Descoberta empírica (T-MT.1b/2b, 2026-05-15):
Admin do tenant piloto acessou /teste-padaria com painel admin aberto.
Slug, branding e tenant visual estavam corretos. Autorização estava incorreta.
A RLS protegeu os dados — mas o guard de frontend falhou.

Esta distinção é fundamental para qualquer agente operando neste sistema:

CONTEXTO VISUAL — O que o usuário vê no navegador.
  slug na URL, nome do tenant, cores e branding.
  → NÃO garante autorização. NÃO garante isolamento de dados.

CONTEXTO FRONTEND — O que o React resolve como activeTenant.
  getRouteTenantSlug() → slug → resolveTenant() → activeTenant.id
  → Garante: qual tenant está sendo exibido.
  → NÃO garante: qual tenant o usuário pertence.

CONTEXTO AUTH — O que o JWT contém.
  get_tenant_id() lê app_metadata.tenant_id, profile.tenant_id lê da tabela profiles.
  → Garante: qual tenant o usuário PERTENCE.
  → NÃO pode ser inferido do slug ou do branding.

CONTEXTO INSTITUCIONAL — O que a RLS enforça no banco.
  get_tenant_id() nas policies → isolamento real.
  WITH CHECK nas writes → rejeição no nível correto.
  → É a única camada que garante segurança real.

REGRA: Nenhum agente deve inferir isolamento operacional apenas por contexto visual.
O contexto visual pode estar correto enquanto o contexto auth é completamente diferente.
Sempre verificar: JWT tenant_id, profile.tenant_id e activeTenant.id — os três devem coincidir
para que admin access seja legítimo.

---

### "Não sei qual módulo carregar"

→ Consultar tabela TAREFA → MÓDULOS em rag/index.md
→ Se a tarefa não estiver mapeada: carregar r-orquestracao-caos + módulo mais próximo

### "Snapshot parece desatualizado"

→ Aplicar r-staleness-detection
→ Sinalizar ao usuário antes de usar como base de análise
→ Não bloquear ciclo por suspeita de staleness — documentar a suspeita

### "Há dois módulos que parecem cobrir o mesmo problema"

→ Aplicar r-module-pruning
→ Consolidar antes de criar novo ciclo nesse domínio

### "index.md referencia módulo que não existe"

→ Remover a referência do index.md
→ Commit `[rag](simplificação): remover referência fantasma [nome]`
→ Não bloquear operação — registrar no telemetria

### "Ciclo foi interrompido no meio"

→ Verificar snapshot do domínio: qual é o estado_atual?
→ Verificar rag/locks/ — há lock ativo?
→ Usar r-estados-ciclo para retomar do estado correto

---

## LIMITES CONHECIDOS

1. **Não é autônomo:** gate humano é obrigatório em toda execução estrutural.
2. **LLMs não são determinísticos:** replay produz reconstrução, não reprodução exata.
3. **Evidência ≠ corretude:** o Git trail prova que algo aconteceu, não que foi correto.
4. **Escala não testada:** o sistema foi validado em 1 projeto solo. Equipes maiores precisam adaptar.
5. **Identidade do executor é declarativa:** "agente_executor executou" é uma afirmação, não uma prova técnica (até v4.5).

---

## PRINCÍPIOS DE OPERAÇÃO

1. **Legibilidade humana acima de eficiência técnica:** qualquer stakeholder deve conseguir ler os artefatos.
2. **Um módulo, uma pergunta:** módulos que respondem múltiplas perguntas devem ser divididos.
3. **Gate humano obrigatório:** nenhuma execução estrutural sem aprovação explícita.
4. **Degradação controlada:** sem módulo v3.5 → modo v3.0. Sem módulo v3.0 → modo v2.2.
5. **Produto antes de infraestrutura:** o C.A.O.S serve o produto. O produto não serve o C.A.O.S.

---

## CONTATO E SUPORTE

Problemas com o C.A.O.S devem ser tratados como ciclos operacionais:
proposta de melhoria → análise → validação → implementação.

Nenhuma mudança arquitetural no C.A.O.S deve ser feita sem snapshot correspondente.
