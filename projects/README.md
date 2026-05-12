# Projetos — C.A.O.S Core

Este diretório documenta os projetos que adotam o C.A.O.S como infraestrutura cognitiva.

---

## Como adicionar um projeto

Quando um novo projeto adota o C.A.O.S:

1. Crie uma entrada neste arquivo
2. Documente o repositório do projeto
3. Registre a data de adoção e versão do core utilizada

---

## Projetos registrados

### Afeto em Forma

| Campo | Valor |
|---|---|
| Repositório | c:\afetoeforma (local) |
| Domínio | Plataforma SaaS de micro-pedidos para artesãos |
| Data de adoção | 2026-05-09 (retroativo — projeto existia antes do C.A.O.S) |
| Versão do core | 4.0 |
| Status | Piloto operacional homologado |
| Nota | Projeto de origem do C.A.O.S — onde o sistema foi desenvolvido e validado |

---

## Arquitetura multi-projeto

Cada projeto que adota o C.A.O.S mantém seu próprio repositório.
O caos-core é o repositório de referência — não um submodule obrigatório.

Fluxo recomendado:
1. Clonar caos-core como ponto de partida
2. Adicionar módulos de domínio específico (k/banco/, k/frontend/)
3. Preencher k-sys-registry-dominios.md com os domínios do projeto
4. Executar primeiro ciclo real
5. Registrar aqui como projeto ativo

---

## Sincronização com core

Quando o caos-core evolui (nova versão de módulo), projetos existentes podem:
1. Copiar manualmente os módulos atualizados
2. Comparar com suas versões locais (diff)
3. Decidir se a atualização é relevante para o contexto do projeto

Não há sincronização automática — decisão humana sempre.
