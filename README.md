# C.A.O.S — Cognitive Autonomous Operational System

> Middleware de Continuidade Cognitiva para desenvolvimento assistido por múltiplos agentes de IA.

versao: 4.0
status: estável
origem: Afeto em Forma (2026)

---

## O que é

O C.A.O.S é uma infraestrutura cognitiva local que preserva continuidade operacional
entre sessões de IA, agentes diferentes e ambientes distintos.

Ele resolve um problema específico:

> O conhecimento técnico e operacional fica preso dentro das sessões efêmeras dos modelos de IA.
> Quando ocorre troca de agente, sessão, janela de contexto ou plano pago,
> há perda de causalidade, decisões, arquitetura e estado cognitivo.

O C.A.O.S cria uma camada de memória institucional persistente que sobrevive a essas trocas.

---

## O que NÃO é

- Não é um framework de código
- Não é um plugin
- Não é um sistema autônomo
- Não é uma plataforma cloud
- Não é um sistema de tickets ou gestão de projetos

---

## Como funciona

O C.A.O.S opera através de três camadas:

**1. Memória Modular (RAG)**
Arquivos Markdown e YAML que respondem perguntas específicas sobre como operar.
Carregados por demanda — máximo 3 por ciclo.

**2. Memória Institucional (Snapshots)**
Registros de decisões arquiteturais homologadas, com contexto, riscos e resultado.
Persistem entre sessões e agentes.

**3. Evidência Operacional (Git)**
Trail verificável de execuções — o que foi alterado, quando, por quem.
Conectado aos snapshots por vínculo bidirecional.

---

## Quickstart — 15 minutos

### Projeto novo

```bash
# 1. Clonar o core
git clone https://github.com/[usuario]/caos-core.git meu-projeto
cd meu-projeto

# 2. Reinicializar como novo repositório
rm -rf .git
git init
git branch -m master main

# 3. Preencher identidade do projeto
# Editar: AGENTS.md (nome, stack, fase)
# Criar: rag/k/projeto/k-proj-identidade.md
# Preencher: rag/k/sistema/k-sys-registry-dominios.md com seus domínios

# 4. Primeiro commit
git add .
git commit -m "[snapshot](sistema): baseline inicial [nome-projeto]"

# 5. Executar primeiro ciclo com Claude
# Abrir AGENTS.md → Claude lê → Claude propõe → você valida → ciclo registrado
```

### Projeto legado

Ver `rag/k/projeto/k-bootstrap-caos.md` — guia completo para 3 perfis de adoção.

---

## Arquitetura

```
caos-core/                     ← este repositório
├── AGENTS.md                  ← autoridade máxima do sistema
├── README.md                  ← este arquivo
├── CHANGELOG.md               ← histórico de versões
├── .gitignore                 ← proteções (.env, locks, node_modules)
└── rag/
    ├── index.md               ← mapa de módulos
    ├── r/                     ← regras operacionais (20 módulos)
    ├── k/
    │   ├── sistema/           ← conhecimento do sistema C.A.O.S
    │   └── projeto/           ← bootstrap e metodologia
    ├── docs/                  ← documentação institucional
    ├── templates/             ← templates prontos para copiar
    ├── locks/                 ← estado de sessão (em .gitignore)
    └── arquivo/               ← módulos arquivados
```

Cada projeto que adota o C.A.O.S adiciona suas próprias camadas:

```
meu-projeto/
├── [arquivos caos-core copiados]
└── rag/k/
    ├── banco/                 ← conhecimento específico do banco
    ├── frontend/              ← conhecimento específico do frontend
    └── projeto/               ← identidade, roadmap, decisões
```

---

## Projetos que usam o C.A.O.S

| Projeto | Domínio | Status |
|---|---|---|
| Afeto em Forma | SaaS de micro-pedidos para artesãos | Piloto homologado — v4.0 |

---

## Origem

O C.A.O.S nasceu dentro do desenvolvimento do **Afeto em Forma** —
uma plataforma SaaS para padarias artesanais em São Sebastião/SP.

O problema era simples mas custoso:
cada sessão de IA começava do zero.
Decisões arquiteturais precisavam ser reconstruídas.
Contexto de multi-tenant, RLS, fornadas e pedidos se perdia entre sessões.

A solução emergiu organicamente:
documentar decisões em snapshots, carregar contexto por demanda,
usar Git como evidência de execução, formalizar handoffs entre agentes.

Após validação em produção real (2026), o núcleo foi extraído como infraestrutura reutilizável.

---

## Versões

| Versão | Marco |
|---|---|
| v1.0-v2.2 | Framework operacional + snapshots + auto-recuperação |
| v3.0 | Continuidade operacional entre agentes (registry, handoff, estados) |
| v3.5 | Persistência verificável (Git como ledger institucional) |
| v3.9 | Hardening (staleness, pruning, concurrency, telemetria) |
| v4.0 | Replicabilidade institucional (bootstrap, núcleo mínimo, manual) |

---

## Filosofia

1. **Legibilidade humana acima de tudo** — Markdown, YAML, Git. Qualquer stakeholder pode auditar.
2. **Um módulo, uma pergunta** — escopo isolado, sem duplicação.
3. **Gate humano obrigatório** — nenhuma execução estrutural sem aprovação explícita.
4. **Degradação controlada** — sem módulo avançado → modo mais simples. Nunca falha total.
5. **Produto antes de infraestrutura** — o C.A.O.S serve o projeto. Não o contrário.

---

## Licença

MIT — use, adapte, distribua. Cite a origem se possível.
