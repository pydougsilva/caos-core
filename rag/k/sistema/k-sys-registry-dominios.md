# k-sys-registry-dominios
versao: 1.0

## OBJETIVO

Catálogo canônico de domínios operacionais deste projeto.

Responde à pergunta:
"quais domínios o sistema conhece e como identifica cada um?"

---

## INSTRUÇÃO PARA ADOTANTE

Este arquivo é um TEMPLATE.
Preencha com os domínios do SEU projeto antes de operar.

Para cada domínio principal do seu produto (tabelas de banco, componentes frontend, etc.),
crie uma entrada seguindo o formato abaixo.

Mínimo recomendado: 3-5 domínios críticos no primeiro setup.

---

## ESTRUTURA DE UM DOMÍNIO

```
id: public.[tabela]                    ← identificador canônico único
canônico: public.[tabela]
aliases:
  - [nome_tabela]    (peso: 1.0)       ← nome exato
  - [variação]       (peso: 0.9)       ← singular/plural
  - [tradução]       (peso: 0.8)       ← termo em português
família: banco/[categoria]             ← banco/operacional | banco/segurança | frontend | etc.
módulos_padrão:
  - r/r-rls-padrao
  - k/banco/[módulo-relevante]
snapshots: []                          ← preenchido conforme ciclos são executados
estado_atual: desconhecido             ← estável | degradado | em_manutenção | desconhecido
última_operação: —
embedding_path:                        ← reservado para v4.5 (SBERT)
```

---

## REGISTRY DE DOMÍNIOS

<!-- Substitua este bloco pelos domínios do seu projeto -->

### [DOMÍNIO 1 — exemplo]

```
id: public.usuarios
canônico: public.usuarios
aliases:
  - usuarios        (peso: 1.0)
  - usuario         (peso: 0.9)
  - users           (peso: 0.8)
  - perfil          (peso: 0.7)
família: banco/auth
módulos_padrão:
  - r/r-rls-padrao
snapshots: []
estado_atual: desconhecido
última_operação: —
embedding_path:
```

---

## FAMÍLIAS SUGERIDAS

| Família | Quando usar |
|---|---|
| banco/auth | tabelas de autenticação e perfis |
| banco/operacional | tabelas do negócio (pedidos, produtos, etc.) |
| banco/segurança | audit logs, permissões, policies |
| banco/financeiro | pagamentos, assinaturas, faturamento |
| banco/plataforma | métricas, configurações de plataforma |
| frontend | componentes principais da UI |
| integração | serviços externos, APIs, webhooks |

---

## LIMITES

- Peso mínimo para alias: 0.5 (abaixo disso, falsos positivos)
- Aliases por domínio: 2-8
- Domínios por arquivo: sem limite, mas > 20 → considerar separar por família
