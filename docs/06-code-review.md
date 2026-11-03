---
id: code-review
title: "Revisão de código"
type: checklist
status: proposed
consult_when: [review-code, validate-work]
applies_to: ["src/**", "tests/**"]
related: [overview, code-style, file-organization, testing, security, git-workflow, quality-gates]
tags: [revisao, bloqueio]
---

# Revisão de código

## Objetivo

Dar ao agente revisor uma lista fechada do que impede o merge. O revisor marca cada item como `ok` ou `bloqueio` e cita o id `STD-REV`. Quem abriu o pull request não marca a própria revisão como aprovada.

## Checklist

### STD-REV-001 Requisito rastreável

Todo comportamento novo ou alterado cita o id `FR-*`, `NFR-*` ou `STD-*` na descrição. Critério de aceitação tocado tem teste com esse id no nome (`STD-TEST-001`).

### STD-REV-002 Teste da mudança

Há teste novo ou atualizado para o comportamento. Correção de defeito inclui o teste que falhava antes (`STD-TEST-006`). Teste sem afirmação (`expect`) não conta.

### STD-REV-003 Tipos e fronteiras

O diff não introduz `any`, `@ts-ignore`, `@ts-nocheck`, `export default` indevido nem schema Zod fora de `src/shared` (`STD-CODE-001`, `STD-CODE-002`, `STD-CODE-003`).

### STD-REV-004 Dependências entre pastas

Nenhum import novo viola a matriz de `STD-FILE-002`. Em especial, `src/content/**` não passa a importar `src/storage/**`. Arquivo novo está no diretório do componente dono (`STD-FILE-001`).

### STD-REV-005 Segurança

O diff não adiciona HTML dinâmico, `eval`, chamada de rede, segredo, afrouxamento de CSP nem permissão sem citação de `agentflow-trd:nfr` ou `agentflow-trd:stores` (`security`).

### STD-REV-006 Fora de escopo

A mudança não implementa item de "Fora de escopo" de `agentflow-trd:overview` nem de `agentflow-architecture:overview`. Se a tarefa pedir isso, o revisor bloqueia e devolve o conflito. Não reescreve o escopo no código.

### STD-REV-007 Gates e entrega

Os gates de `quality-gates` terminaram com código 0 no commit revisado. A descrição tem `docs_pins` (`STD-GIT-003`). `agentflow.docs.lock.yaml` não mudou junto com código de feature (`STD-GIT-005`).

### STD-REV-008 Diff mínimo

O diff não reformata arquivo alheio à tarefa, não renomeia símbolo sem necessidade e não troca dependência que o comportamento não exigiu. Formatação de arquivo tocado, feita pelo Prettier, é aceita.

## Severidade

| Severidade | Itens | Efeito |
| ---------- | ----- | ------ |
| Bloqueio | `STD-REV-001` a `STD-REV-008` quando a resposta é não | Impede o merge. O autor corrige e pede revisão de novo. |
| Observação | Sugestão que não viola um `STD` | Não impede o merge. Não vira mudança obrigatória dentro do mesmo pull request. |

Não existe severidade intermediária. Ou viola um `STD-REV`, ou é observação.

## Questões em aberto

Nenhuma.

## Regras para agentes

- DEVE percorrer `STD-REV-001` a `STD-REV-008` e registrar `ok` ou `bloqueio` para cada um.
- DEVE impedir o merge enquanto existir um bloqueio.
- NÃO DEVE aprovar o próprio pull request.
- NÃO DEVE transformar observação em mudança obrigatória no mesmo pull request.
- DEVE citar o id `STD-REV-*` e o arquivo em cada bloqueio.
- NÃO DEVE dispensar gate vermelho porque a mudança parece pequena (`STD-REV-007`).
