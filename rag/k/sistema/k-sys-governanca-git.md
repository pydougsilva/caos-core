# k-sys-governanca-git
versao: 1.1

## OBJETIVO

Definir as convenções operacionais de branch, commit e governança
Git dentro do C.A.O.S.

Responde à pergunta:
"quais são as regras de uso do Git como camada institucional?"

Para as regras operacionais de quando e como usar cada convenção,
ver r-git-operacional, r-commit-governance, r-rollback-contextual.

---

## ESTRATÉGIA DE BRANCHES

O C.A.O.S opera com quatro tipos de branch. Cada tipo tem
propósito, ciclo de vida e política de merge distintos.

### main

Branch institucional de produção.

```
Propósito:   estado validado e verificável do sistema
Commits:     NUNCA direto — sempre via merge de branch
Proteção:    obrigatória (branch protection rule)
Ciclo:       permanente
```

Cada commit em `main` representa um ciclo institucional completo:
decisão homologada + execução verificada + merge autorizado pelo usuário.

### ops/[domínio-abreviado]-[YYYYMMDD]

Branches operacionais para execuções de ciclos.

```
Propósito:   isolamento da execução de um ciclo específico
Criado por:  agente_executor, antes de iniciar a execução
Nomeação:    ops/[domínio-abreviado]-[YYYYMMDD]
Ciclo:       vida curta — criado no início, deletado após merge
```

Exemplos:
```
ops/audit-logs-20260509
ops/fornadas-20260612
ops/profiles-rls-20260701
```

Regra: uma branch ops/ por ciclo operacional.
Não reutilizar. Não mesclar ciclos distintos em uma mesma branch.

### rag/[módulo-abreviado]-[YYYYMMDD]

Branches para atualizações de módulos RAG.

```
Propósito:   versionamento de atualizações na memória operacional
Criado por:  agente_executor (ou agente_orquestrador via instrução de atualização RAG)
Nomeação:    rag/[módulo-abreviado]-[YYYYMMDD]
Ciclo:       vida curta — criado, commitado, mergeado, deletado
```

Exemplos:
```
rag/r-rls-padrao-20260515
rag/k-registry-20260601
rag/index-v80-20260701
```

### docs/[marco]-[YYYYMMDD]

Branches para documentação institucional.

```
Propósito:   registros de marcos, fechamentos, relatórios
Criado por:  agente_executor (a partir de instrução de agente_orquestrador)
Nomeação:    docs/[marco-abreviado]-[YYYYMMDD]
Ciclo:       vida curta
```

Exemplos:
```
docs/v35-fechamento-20260601
docs/caos-v20-doc-20260510
```

---

## NOMENCLATURA INSTITUCIONAL

### Domínio abreviado em branches

Usar o nome da tabela ou componente sem o schema prefix.

| Domínio canônico | Abreviação em branch |
|---|---|
| rag/r/r-rls-padrao | r-rls-padrao |
| sistema C.A.O.S | sistema |
| bootstrap | bootstrap |

Exemplos genéricos — adaptar ao projeto.

### Data

Formato obrigatório: `YYYYMMDD`
Exemplos: `20260509`, `20261231`

---

## FORMATO OBRIGATÓRIO DO COMMIT

Todo commit institucional deve seguir exatamente este formato:

```
[tipo](domínio): descrição em imperativo, presente, sem ponto final

snapshot: ID-do-snapshot
agent-executor: [agente_executor_id]
agent-orchestrator: [agente_orquestrador_id]
risks-addressed: N

Co-Authored-By: agente_executor <noreply@agente>
Orchestrated-By: agente_orquestrador <noreply@agente>
```

### Linha de título

```
[tipo](domínio): descrição
```

- `[tipo]` — prefixo em colchetes (ver tabela abaixo)
- `(domínio)` — domínio canônico abreviado entre parênteses
- `: ` — separador obrigatório
- `descrição` — imperativo, presente, sem ponto final
- Comprimento máximo da linha de título: 72 caracteres

Exemplos válidos:
```
[rls](public.audit_logs): substituir policy email_admin por get_tenant_id()
[migration](public.fornadas): adicionar coluna capacidade_bolo
[rag](r-rls-padrao): atualizar regra de alias para fornadas
[docs](sistema): formalizar persistência operacional verificável
```

### Corpo do commit

Separado do título por uma linha em branco.
Todos os campos são obrigatórios:

| Campo | Obrigatório | Formato |
|---|---|---|
| `snapshot:` | sim | ID do snapshot que originou o ciclo |
| `agent-executor:` | sim | agente_executor |
| `agent-orchestrator:` | sim | agente_orquestrador |
| `risks-addressed:` | sim | número inteiro ≥ 0 |

### Trailers Git

Separados do corpo por uma linha em branco:

```
Co-Authored-By: agente_executor <noreply@agente>
Orchestrated-By: agente_orquestrador <noreply@agente>
```

---

## TIPOS VÁLIDOS DE COMMIT

| Tipo | Quando usar | Exemplo |
|---|---|---|
| `[hotfix]` | correções em código de aplicação | `[hotfix](src/checkout): corrigir cálculo de capacidade` |
| `[migration]` | alterações de schema SQL | `[migration](public.fornadas): adicionar coluna capacidade_bolo` |
| `[rls]` | alterações em Row Level Security | `[rls](public.audit_logs): alinhar policy ao padrão get_tenant_id()` |
| `[rag]` | atualizações de módulos RAG | `[rag](r-rls-padrao): adicionar regra para fornadas` |
| `[docs]` | documentação institucional | `[docs](sistema): formalizar v3.5 persistência verificável` |
| `[snapshot]` | quando snapshots são versionados via Git | `[snapshot](audit-logs): registrar ciclo hotfix-rls` |
| `[handoff]` | registros de transferência entre agentes | `[handoff](sistema): registrar handoff ciclo-id abc123` |
| `[revert]` | rollback técnico de execução anterior | `[revert](public.fornadas): reverter migration capacidade_bolo` |

---

## COMMITS PROIBIDOS

| Commit proibido | Motivo |
|---|---|
| Sem campo `snapshot:` | execução sem decisão institucional |
| Sem `agent-executor:` | autoria indeterminada — não rastreável |
| Diretamente em `main` | bypassa governança — sem gate humano |
| Múltiplos domínios sem aprovação | viola escopo único — dificulta auditoria |
| Contendo `.env`, credentials, tokens | violação de segurança |
| Mensagem sem `[tipo](domínio):` | não é commit institucional reconhecível |
| `git commit --amend` após merge em `main` | altera histórico verificável — evidência comprometida |
| Commit vazio (`--allow-empty`) sem justificativa | ruído no histórico institucional |

---

## POLÍTICA DE MERGE

### Branch ops/ → main

```
1. agente_executor cria commit na branch ops/
2. agente_orquestrador valida diff ↔ instrução autorizada (etapa 7b)
3. Usuário autoriza merge [GATE]
4. Merge executado (preferencialmente fast-forward ou merge commit explícito)
5. Branch ops/ deletada
```

O histórico do merge permanece em `main`.
A branch deletada não perde evidência — o commit está em `main`.

### Merge sem gate humano

Proibido. Toda branch ops/ requer autorização explícita do usuário antes de merge.
Não há exceções — nem para ciclos considerados "simples" ou "de baixo risco".

### Fast-forward vs merge commit

Preferência: fast-forward (`git merge --ff-only`) para manter histórico linear.
Se fast-forward não for possível: merge commit explícito com mensagem institucional.
Nunca: `git merge --squash` — apaga autoria e metadados do commit original.

---

## PROTEÇÃO DA BRANCH MAIN

A branch `main` deve ser configurada com proteção explícita.

Configuração recomendada:
```
Regras de proteção de branch (configurar no repositório):
- Proibir push direto em main
- Exigir pull request antes de merge
- Exigir revisão antes de merge (1 aprovação mínima)
```

Enquanto o repositório operar apenas localmente:
a proteção é disciplinar — garantida pelo processo, não por ferramenta.
agente_executor não deve criar commits em `main` diretamente.

---

## ESCOPO ÚNICO POR COMMIT

Cada commit deve representar exatamente um ciclo operacional.

Isso implica:

- Um commit por handoff executado
- Um domínio por commit (salvo aprovação explícita multi-domínio)
- Os artefatos alterados no commit devem corresponder a `artefatos_alterados` no handoff
- Nenhum arquivo "de passagem" — todo arquivo no commit foi alterado intencionalmente

### Por que escopo único importa

Quando um commit altera múltiplos domínios sem aprovação:
- O rollback se torna destrutivo (reverter um domínio reverte outro)
- A auditoria se torna ambígua (qual snapshot autorizou qual alteração?)
- O replay se torna impossível de isolar

Escopo único é o que torna o rollback cirúrgico e o replay preciso.

---

## RELAÇÃO BRANCH ↔ DOMÍNIO ↔ CICLO

A branch ops/ herda sua identidade do ciclo que a originou:

```
handoff.dominio         →  ops/[domínio-abreviado]-[data]
handoff.ciclo_id        →  pode ser incluído no nome da branch
handoff.snapshot_ref    →  campo snapshot: no commit
```

Exemplo de rastreabilidade completa:

```
Handoff:
  ciclo_id: f47ac10b-58cc-4372-a567-0e02b2c3d479
  dominio: sistema
  snapshot_ref: sistema-001

Branch criada por agente_executor:
  ops/sistema-20260509

Commit na branch:
  [rag](sistema): atualizar módulo r-rls-padrao
  snapshot: sistema-001
  agent-executor: [agente_executor_id]
  agent-orchestrator: [agente_orquestrador_id]
```

Da branch, chega-se ao ciclo.
Do ciclo, chega-se à decisão.
Da decisão, chega-se ao contexto completo.

---

## LIMITES

- Este arquivo define convenções — não impõe tecnicamente
- Proteção de `main` deve ser configurada manualmente no repositório
- O formato de commit é verificado por r-commit-governance, não por hook automático
- A relação branch ↔ ciclo é rastreável apenas se as convenções de nomenclatura forem seguidas
