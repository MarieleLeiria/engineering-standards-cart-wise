---
id: nome-curto
title: "<Título do padrão>"
type: standard          # overview | standard | process | checklist
status: proposed        # proposed | accepted | superseded | deprecated | rejected
consult_when: []        # valores de manifest.yaml > vocabulary.consult_when
applies_to: []          # globs de código no repositório do produto
related: []             # ids deste repositório, ou <repositorio>:<id>
tags: []
---

# <Título do padrão>

## Objetivo

<O que o padrão torna previsível para o agente. Uma frase.>

## Padrões

### STD-XXX-001 <título curto>

<Regra testável. Um padrão, um comportamento. O id não muda quando o texto muda.>

## Exemplos

<Trecho curto do que cumprir e do que viola. O exemplo não cria regra nova.>

## Questões em aberto

- <Itens que impedem `status: accepted`. Escreva "Nenhuma." se não houver.>

## Regras para agentes

- DEVE manter o id `STD-...` estável; revisão, testes e o resultado da tarefa citam esse id.
- DEVE atualizar a entrada em `manifest.yaml` ao criar este documento ou mudar `status`, `consult_when`, `applies_to` ou `tags`.
- NÃO DEVE inventar `consult_when`. O valor entra primeiro em `vocabulary`.
- NÃO DEVE copiar para cá regras de estrutura ou de comportamento. Aponte `agentflow-architecture:<id>` ou `agentflow-trd:<id>`.
