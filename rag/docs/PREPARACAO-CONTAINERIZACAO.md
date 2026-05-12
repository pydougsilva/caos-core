# C.A.O.S — Preparação para Containerização Futura

versao: 1.0
status: preparação — não implementado
data: 2026-05-12

---

## PRINCÍPIO

O C.A.O.S deve ser portátil sem depender de Docker.
Container é uma opção futura — não uma dependência atual.

O sistema funciona como diretório de arquivos.
Funciona em qualquer máquina com Git e um editor.

---

## O QUE CONTAINERIZAÇÃO RESOLVERIA

| Problema | Situação atual | Com container |
|---|---|---|
| "Funciona na minha máquina" | Node/Python versions divergem | Ambiente reproduzível |
| Onboarding de novo dev | npm install + .env manual | docker compose up |
| CI/CD | configuração manual | container rodando testes |
| Múltiplos projetos isolados | diretórios separados | containers separados |

---

## CONTRATOS PARA COMPATIBILIDADE FUTURA

Para que o C.A.O.S funcione em container futuro sem reescrita:

**1. Variáveis de ambiente via .env (já implementado)**
O .env não é commitado. Container injeta variáveis.
Nenhum segredo hardcoded no código.

**2. Caminhos relativos no RAG**
Módulos RAG usam caminhos relativos (`rag/r/`, `rag/k/`).
Não usar caminhos absolutos que mudariam em container.

**3. Git disponível no ambiente**
O C.A.O.S usa Git como ledger.
Container deve ter Git instalado.

**4. Locks em diretório isolado**
`rag/locks/` está em .gitignore.
Container pode montar este diretório como volume efêmero.

**5. Snapshots e telemetria como volumes**
`rag/docs/ciclos/` deve ser volume persistente em container.
Sem isso, memória institucional é perdida quando container é recriado.

---

## ESTRUTURA DE DOCKER FUTURO (CONTRATO, NÃO IMPLEMENTAÇÃO)

```yaml
# docker-compose.yml (FUTURO — não criar agora)
services:
  caos:
    build: .
    volumes:
      - ./rag:/app/rag          # memória institucional persistente
      - ./src:/app/src          # código do produto
    env_file:
      - .env                    # segredos locais
    # rag/locks/ NÃO é volume — é efêmero por container
```

---

## O QUE NÃO IMPLEMENTAR AGORA

- Dockerfile
- docker-compose.yml
- scripts de entrypoint
- health checks
- orquestração de containers

Estes artefatos serão criados quando o projeto tiver necessidade real
de ambiente reproduzível (CI/CD, time distribuído, deploy automatizado).

---

## CRITÉRIO PARA IMPLEMENTAR

Implementar containerização quando qualquer uma das condições for verdadeira:

- Mais de 2 desenvolvedores no projeto com ambientes diferentes
- CI/CD automatizado é necessário
- Deploy em múltiplos ambientes (staging/production)
- O time gasta > 1h/semana resolvendo problemas de ambiente

Enquanto nenhuma condição for verdadeira: manter como diretório de arquivos.
