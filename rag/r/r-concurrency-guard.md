# r-concurrency-guard
versao: 1.0

## OBJETIVO

Definir protocolo simples para prevenir colisão entre ciclos ativos
no mesmo domínio operacional.

Responde à pergunta:
"existe outro ciclo ativo neste domínio antes de eu começar?"

---

## PRINCÍPIO CENTRAL

Um domínio com ciclo ativo é um domínio com estado em transição.
Iniciar um segundo ciclo no mesmo domínio sem resolver o primeiro
cria inconsistência de estado e risco de snapshots conflitantes.

---

## MECANISMO — LOCK FILE

O mecanismo de concurrency guard usa um arquivo de lock simples
no diretório `rag/locks/`.

### Estrutura do lock file

```
Localização: rag/locks/[domínio-canônico].lock.yml
Exemplo:     rag/locks/public.audit_logs.lock.yml
```

```yaml
# Arquivo de lock — NÃO commitar este arquivo (está em .gitignore)
dominio: public.audit_logs
ciclo_id: f47ac10b-58cc-4372-a567-0e02b2c3d479
estado: EXECUTANDO
agente_orquestrador: Claude
timestamp_abertura: 2026-05-12T10:00:00-03:00
sessao_id: claude-session-20260512-a1b2
estimativa_conclusao: 2026-05-12T11:00:00-03:00   # opcional
```

---

## CICLO DE VIDA DO LOCK

### Criação do lock (etapa 0a — antes de iniciar análise)

Quando Claude identifica o domínio e está prestes a iniciar ciclo:

```
1. Verificar: existe rag/locks/[domínio].lock.yml ?
   → NÃO: criar lock e prosseguir
   → SIM: ler o lock e aplicar regras de conflito (ver abaixo)
```

### Atualização do lock (a cada transição de estado)

O lock é atualizado quando o estado do ciclo muda:

```yaml
estado: PROPOSTO        # após Claude gerar instrução
estado: VALIDADO        # após aprovação humana
estado: EXECUTANDO      # após handoff para Codex
estado: CONCLUÍDO       # antes de deletar o lock
```

### Remoção do lock (após CONCLUÍDO ou REJEITADO)

Lock é deletado quando o ciclo termina.
Se ciclo é interrompido abruptamente, o lock persiste — ver seção LOCK STALE.

---

## REGRAS DE CONFLITO

Quando um lock existe para o domínio alvo:

### Caso 1 — Lock com estado EXECUTANDO

```
Ação: não iniciar novo ciclo.
Sinalizar ao usuário: "Domínio [X] tem ciclo ativo (ID: [ciclo_id], estado: EXECUTANDO).
Aguardar conclusão ou verificar se o ciclo foi interrompido."
```

### Caso 2 — Lock com estado VALIDADO (handoff emitido, aguardando Codex)

```
Ação: não iniciar novo ciclo.
Sinalizar: "Ciclo [ciclo_id] foi validado mas Codex ainda não retornou.
Verificar estado de execução antes de prosseguir."
```

### Caso 3 — Lock com estado PROPOSTO (aguardando aprovação humana)

```
Ação: pode sinalizar ao usuário para decidir.
Opções: (1) abandonar o ciclo proposto, (2) aguardar resposta, (3) criar ciclo paralelo sob responsabilidade explícita do usuário.
```

### Caso 4 — Lock com estado CONCLUÍDO (lock não foi deletado)

```
Ação: lock stale. Deletar o lock e prosseguir.
Este é o único caso onde o lock é deletado automaticamente por um novo agente.
```

---

## LOCK STALE

Um lock é considerado stale quando:

```
timestamp_abertura < data_atual - 24 horas
E estado ≠ CONCLUÍDO
```

Ação quando lock stale detectado:

```
1. Não deletar automaticamente — sinalizar ao usuário
2. Apresentar: "Lock de [domínio] aberto há [N] horas. Ciclo [ID] pode ter sido interrompido."
3. Aguardar decisão humana: retomar o ciclo interrompido ou declarar abandono e deletar lock
```

---

## LOCALIZAÇÃO DOS LOCKS

```
rag/locks/
  public.audit_logs.lock.yml
  public.[dominio_a].lock.yml
  ...
```

**Este diretório deve estar no .gitignore.**
Locks são estado de sessão — não estado institucional.
Ao final de cada ciclo bem-sucedido, o lock é deletado.

Adicionar ao .gitignore:
```
rag/locks/
```

---

## DOMÍNIOS INDEPENDENTES — SEM CONFLITO

Dois ciclos em domínios diferentes podem ocorrer simultaneamente.

```
Exemplo válido:
  Ciclo A: public.audit_logs (estado: EXECUTANDO)
  Ciclo B: public.[dominio_b] (estado: PROPOSTO)
  → Sem conflito. Domínios independentes.

Exemplo de conflito potencial:
  Ciclo A: public.[dominio_a] (estado: EXECUTANDO)
  Ciclo B: public.[entidade_relacionada] (estado: iniciando)
  → Atenção: [entidade_relacionada] tem FK para [dominio_a].
    Claude deve avaliar se a mudança em [dominio_a] afeta [entidade_relacionada].
```

Quando domínios têm FK entre si, Claude avalia manualmente se há risco de conflito.
Não há lock automático para dependências FK — é julgamento arquitetural.

---

## SIMPLICIDADE OPERACIONAL

Este mecanismo é:
- um arquivo YAML por domínio
- criado e deletado manualmente (por Claude ou Codex)
- legível por qualquer agente ou humano
- sem banco de dados, sem servidor, sem daemon

Se o sistema de locks crescer em complexidade além de arquivos YAML simples,
ele deve ser rejeitado e revisado.

---

## PROIBIÇÕES

Nunca:
- commitar arquivos `.lock.yml` no Git
- criar locks para domínios que não estão no registry
- manter lock indefinidamente após CONCLUÍDO
- deletar lock de outro agente sem verificar o estado do ciclo
