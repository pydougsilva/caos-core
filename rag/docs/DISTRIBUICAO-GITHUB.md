# C.A.O.S — Política de Distribuição via GitHub

versao: 1.0
data: 2026-05-12

---

## PRINCÍPIO

O Git local preserva execução operacional.
O GitHub preserva continuidade institucional distribuída.

A combinação garante que o projeto sobrevive a:
- perda de máquina local
- troca de computador
- colaboração entre desenvolvedores
- acesso a partir de qualquer ambiente

---

## O QUE DEVE SUBIR PARA O REMOTO (obrigatório)

### Artefatos institucionais

```
AGENTS.md                           ← autoridade máxima
rag/index.md                        ← mapa operacional
rag/r/*.md                          ← todos os módulos de regras
rag/k/**/*.md                       ← todos os módulos de conhecimento
rag/docs/**/*.md                    ← documentação institucional
rag/docs/ciclos/telemetria/*.md     ← telemetria de sessões
rag/templates/*.md                  ← templates reutilizáveis
.gitignore                          ← proteção institucional
```

### Código do produto

```
src/                ← código-fonte
public/             ← assets públicos
supabase/           ← migrations e edge functions
package.json        ← dependências
package-lock.json   ← lock de versões
vite.config.js      ← configuração de build
```

---

## O QUE NÃO DEVE SUBIR (nunca)

```
.env                ← credenciais — NUNCA
.env.*              ← qualquer variante de env
node_modules/       ← dependências (recriáveis via npm install)
dist/               ← build (recriável via npm run build)
rag/locks/          ← locks de sessão (estado efêmero)
*.local             ← arquivos locais temporários
```

**Verificação obrigatória antes de qualquer push:**
```bash
git status | grep "\.env"    # não deve retornar nada
```

---

## ESTRATÉGIA DE BRANCHES

```
main                ← estado institucional validado (protegido)
ops/[domínio]-[data] ← execuções operacionais (vida curta)
rag/[módulo]-[data]  ← atualizações de RAG (vida curta)
docs/[marco]-[data]  ← documentação institucional (vida curta)
```

Nenhum commit vai diretamente para main.
Toda mudança entra via branch com merge explicitamente autorizado.

---

## TAGS SEMÂNTICAS

Usar tags para marcar marcos institucionais significativos:

```bash
git tag -a v3.5 -m "C.A.O.S v3.5 — Persistência Operacional Verificável"
git tag -a v3.9 -m "C.A.O.S v3.9 — Hardening Institucional"
git tag -a v4.0 -m "C.A.O.S v4.0 — Replicabilidade Institucional"
```

Tags permitem:
- localizar rapidamente o estado do sistema em qualquer marco
- criar releases com documentação do estado institucional
- referenciar versões específicas em snapshots e telemetria

---

## RECUPERAÇÃO OPERACIONAL

Se a máquina local for perdida:

```bash
# Clonar o repositório
git clone [url-remoto] projeto

# Restaurar dependências
npm install

# Verificar estado operacional
cat AGENTS.md           # autoridade máxima
cat rag/index.md        # mapa de módulos
ls rag/docs/ciclos/telemetria/  # última telemetria

# Criar .env local
# (recriar manualmente — NUNCA está no remoto)
```

---

## CONTINUIDADE ENTRE MÁQUINAS

Para trabalhar em máquinas diferentes no mesmo projeto:

```bash
# Antes de trocar de máquina
git add [arquivos alterados]
git commit -m "[tipo](domínio): [descrição]"
git push

# Ao retomar em outra máquina
git pull
cat rag/docs/ciclos/telemetria/*.md  # última sessão
```

A telemetria de sessão preserva o contexto que não está no código.
Antes de retomar, ler a última telemetria.

---

## ROLLBACK INSTITUCIONAL

Se o projeto precisar voltar a um estado anterior:

```bash
# Ver marcos disponíveis
git log --oneline
git tag

# Criar branch de investigação a partir de um marco
git checkout -b investigacao v3.5

# NÃO fazer git reset --hard em main — isso destrói histórico
# USAR git revert para desfazer commits específicos
```

---

## RELEASES INSTITUCIONAIS

Ao concluir uma fase significativa do C.A.O.S ou do produto:

```bash
# Tag da release
git tag -a [versao] -m "[descrição do marco]"

# Push da tag
git push origin [versao]
```

O release é acompanhado de:
- Documento de fechamento oficial (rag/docs/)
- Snapshot consolidado de todos os domínios ativos
- Telemetria da sessão de fechamento

---

## REPOSITÓRIO PÚBLICO vs PRIVADO

Para projetos com dados de clientes: **sempre privado**.

Se o C.A.O.S for distribuído como template para outros projetos:
criar repositório separado apenas com os artefatos de infraestrutura (sem código do produto).

```
caos-template/
├── AGENTS.md (template)
├── rag/
│   ├── index.md (template)
│   ├── r/ (módulos genéricos)
│   ├── k/sistema/ (módulos de sistema)
│   └── templates/
└── .gitignore
```
