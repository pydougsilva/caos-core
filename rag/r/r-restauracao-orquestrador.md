# r-restauracao-orquestrador
versao: 1.0

## OBJETIVO

Definir como o C.A.O.S restaura o papel de `agente_orquestrador`
quando a ferramenta, sessao ou agente que exercia esse papel fica indisponivel.

Responde a pergunta:
"como um novo agente assume a orquestracao institucional sem depender da identidade anterior?"

---

## PRINCIPIO CENTRAL

O C.A.O.S depende de papeis.
Nunca depende de identidades.

Ferramentas sao implementacoes possiveis.
Papeis sao estrutura institucional.

`agente_orquestrador` e um papel substituivel. A identidade concreta do agente
deve ser registrada apenas como evidencia operacional, nunca como requisito
estrutural para continuidade.

---

## QUANDO ATIVAR

Ativar este modulo quando:

- o agente que exercia `agente_orquestrador` esta indisponivel;
- ha troca de ferramenta de orquestracao;
- uma sessao longa foi perdida e precisa ser reconstruida por artefatos;
- houve reorganizacao manual fora da trilha institucional;
- ha suspeita de dependencia nominal em contratos, handoffs ou Git governance.

---

## ARTEFATOS MINIMOS

O novo `agente_orquestrador` deve carregar, nesta ordem:

1. `AGENTS.md`
2. `rag/index.md`
3. `k/sistema/k-sys-registry-dominios.md`
4. `r/r-continuidade-cognitiva.md`
5. `r/r-estados-ciclo.md`
6. `r/r-handoff-executor.md`
7. `r/r-recuperacao-contextual.md`
8. ultima telemetria em `rag/docs/ciclos/telemetria/`
9. historico Git institucional
10. locks em `rag/locks/`

Se qualquer artefato minimo estiver ausente, declarar continuidade parcial antes
de propor execucao estrutural.

---

## PROTOCOLO DE RESTAURACAO

### Etapa 1 - Declarar restauracao

Registrar no inicio da analise:

```yaml
modo_restauracao_orquestrador: ativo
agente_orquestrador_anterior: [identidade se conhecida | desconhecido]
agente_orquestrador_atual: [identidade concreta]
motivo: [indisponibilidade | troca de ferramenta | sessao perdida | reorganizacao manual]
```

### Etapa 2 - Verificar estado institucional

Verificar:

- branch atual e ultimo commit;
- existencia de branches `ops/`, `rag/` ou `docs/` abertas;
- conteudo de `rag/locks/`;
- ultima telemetria;
- snapshots com `estado_atual` intermediario;
- divergencias entre arquitetura planejada e execucao real.

### Etapa 3 - Reconstruir ciclo ativo

Se houver ciclo ativo:

- recuperar `ciclo_id`;
- identificar estado formal;
- localizar handoff, snapshot ou commit associado;
- retomar pelo estado definido em `r-estados-ciclo`.

Se nao houver ciclo ativo:

- declarar `ciclo_ativo: nenhum`;
- reconstruir apenas objetivo institucional pendente.

### Etapa 4 - Validar papel, nao identidade

O novo agente pode assumir `agente_orquestrador` se conseguir:

- classificar tarefa;
- recuperar contexto;
- consultar registry;
- selecionar modulos;
- avaliar riscos;
- propor instrucao deterministica;
- emitir handoff valido;
- validar retorno e diff;
- registrar snapshot e telemetria.

Se alguma capacidade faltar, declarar `restauracao: parcial`.

### Etapa 5 - Retorno formal

Ao encerrar a restauracao, registrar telemetria com:

```yaml
restauracao_orquestrador:
  resultado: concluida | parcial | falhou
  agente_orquestrador_anterior: [identidade | desconhecido]
  agente_orquestrador_atual: [identidade]
  artefatos_consultados: [...]
  lacunas: [...]
  continuidade_resultante: plena | parcial | falha
```

---

## MODO DEGRADADO

Modo degradado ocorre quando o mesmo agente exerce simultaneamente:

- `agente_orquestrador`
- `agente_executor`

Declaracao obrigatoria:

```yaml
modo_operacao: degradado
agente_orquestrador: [identidade concreta]
agente_executor: [mesma identidade concreta]
motivo: [razao]
restricoes:
  - gate humano reforcado
  - diff revisado antes de merge
  - telemetria obrigatoria
  - maximo 5 ciclos consecutivos antes de auditoria
```

Modo degradado nao elimina separacao de responsabilidades. Ele apenas acumula
papeis em uma mesma identidade concreta, com rastreabilidade reforcada.

---

## PROIBICOES

Nunca:

- tratar nome de ferramenta como requisito institucional;
- bloquear restauracao porque a identidade anterior esta ausente;
- criar handoff sem estado `VALIDADO`;
- executar mudanca estrutural sem gate humano;
- apagar evidencia historica para tornar a documentacao "limpa";
- confundir evidencia historica com contrato ativo.

---

## RESULTADO ESPERADO

A restauracao e bem-sucedida quando um novo agente consegue assumir
`agente_orquestrador`, reconstruir o estado institucional por artefatos,
retomar ou encerrar ciclos pendentes e registrar telemetria sem depender da
sessao, memoria ou ferramenta anterior.
