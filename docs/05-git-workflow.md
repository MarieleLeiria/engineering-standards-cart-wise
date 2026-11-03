---
id: git-workflow
title: "Fluxo de Git"
type: process
status: proposed
consult_when: [commit-and-pr]
applies_to: [".github/**", ".gitignore"]
related: [overview, code-review, quality-gates, agentflow-architecture:docs-resolution, agentflow-architecture:adr-0005]
tags: [git, conventional-commits, pull-request]
---

# Fluxo de Git

## Objetivo

Fazer branch, commit e pull request terem a mesma forma em toda tarefa, para o histórico e o `docs_pins` serem legíveis por outro agente. A fixação da documentação por commit está em `agentflow-architecture:adr-0005` e `agentflow-architecture:docs-resolution`.

## Entradas

- Id da tarefa, quando existir (ex.: `CW-123`).
- Lista curta do que a mudança faz.
- Ids `FR-*`, `NFR-*` ou `STD-*` que a mudança implementa ou verifica.
- `docs_pins` da execução, no formato de `agentflow-architecture:docs-resolution`.

## Passos

### STD-GIT-001 Branch

A branch nasce de `main` atualizada e usa `tipo/id-slug` ou, sem id de tarefa, `tipo/slug`.

| Parte | Valores |
| ----- | ------- |
| `tipo` | `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `build`, `ci` |
| `id` | Id da tarefa, sem espaços |
| `slug` | Minúsculas, ASCII, palavras separadas por hífen, no máximo cinco |

Exemplo: `feat/CW-123-preco-shein`. Uma tarefa, uma branch. Não misture duas tarefas na mesma branch.

### STD-GIT-002 Commit

A mensagem segue Conventional Commits. O tipo é o mesmo conjunto de `STD-GIT-001`. O escopo é o diretório de topo afetado: `content`, `adapters`, `background`, `storage`, `ui`, `shared` ou `docs`.

```
tipo(escopo): assunto

Corpo opcional, explicando o porquê.

Refs: CW-123
```

O assunto é português, imperativo, minúsculo, sem ponto final, com no máximo 72 caracteres. Não use `wip`, `ajustes` nem `fix bug` sem dizer o bug.

Um commit por intenção. Não commite `node_modules/`, `dist/`, `.env` nem artefato de cobertura.

### STD-GIT-003 Pull request

Um pull request por tarefa, aberto contra `main`. O título tem a mesma forma do assunto de `STD-GIT-002`, com o tipo e o escopo. A descrição contém, nesta ordem:

1. O que mudou, em até dez linhas.
2. Os ids `FR-*`, `NFR-*` e `STD-*` cobertos.
3. O bloco `docs_pins`.
4. O resultado dos gates de `quality-gates` (comando e código de saída).

O diff fica limitado à tarefa. Não aproveite o pull request para reformatar arquivo que o comportamento não tocou.

### STD-GIT-004 Squash

O merge é squash. A mensagem do squash é o título do pull request mais o corpo que lista os ids de requisito. Não faça merge commit nem rebase-merge.

### STD-GIT-005 Lock da documentação

`agentflow.docs.lock.yaml` só muda num pull request cujo único assunto é atualizar esse arquivo. O corpo lista os documentos alterados entre o SHA antigo e o novo, como manda `agentflow-architecture:docs-resolution`.

Não altere o lock como efeito colateral de uma feature.

## Saídas

- Branch no padrão `STD-GIT-001`.
- Histórico que, depois do squash, tem um commit Conventional no `main`.
- Descrição do pull request com ids de requisito, `docs_pins` e gates.
- Lock file intocado, salvo no pull request dedicado de `STD-GIT-005`.

## Falhas

| Falha | Comportamento |
| ----- | ------------- |
| Branch fora do padrão | Renomeie antes de abrir o pull request. |
| Título fora de Conventional Commits | Corrija o título. Não faça merge. |
| `docs_pins` ausente | Não faça merge. |
| Lock file alterado junto com código de feature | Separe em outro pull request. |
| Gate com código diferente de 0 | Não faça merge. Ver `quality-gates`. |

## Questões em aberto

Nenhuma.

## Regras para agentes

- DEVE criar a branch no formato `tipo/id-slug` (`STD-GIT-001`).
- DEVE escrever commit e título de pull request em Conventional Commits, com assunto em português (`STD-GIT-002`, `STD-GIT-003`).
- DEVE incluir na descrição os ids de requisito, o `docs_pins` e o resultado dos gates.
- DEVE fazer squash no merge (`STD-GIT-004`).
- NÃO DEVE alterar `agentflow.docs.lock.yaml` fora de um pull request só para isso (`STD-GIT-005`).
- NÃO DEVE commitar `node_modules/`, `dist/`, `.env` nem artefato de cobertura.
