---
id: overview
title: "Padrões de engenharia do CartWise"
type: overview
status: proposed
consult_when: [any-task]
applies_to: ["**"]
related: [code-style, file-organization, testing, security, git-workflow, code-review, quality-gates, agentflow-architecture:adr-0007, agentflow-architecture:docs-resolution]
tags: [escopo, precedencia, vocabulario]
---

# Padrões de engenharia do CartWise

## Objetivo

Fixar como o código do CartWise é escrito, testado, revisado e entregue. A estrutura do sistema está em `agentflow-architecture:components`. O comportamento das features está em `agentflow-trd`. Este documento não repete nenhum dos dois.

Os padrões usam id `STD-<SIGLA>-<NNN>`. A sigla identifica o documento: `CODE`, `FILE`, `TEST`, `SEC`, `GIT`, `REV`, `QUAL`. O id não muda quando o texto muda.

## Precedência

A tabela abaixo é a mesma de `agentflow-architecture:docs-resolution`. A fonte da regra é aquele documento. Não edite a precedência aqui.

| Assunto | Repositório que prevalece |
| ------- | ------------------------- |
| Estrutura, componentes, dependências e integrações | `agentflow-architecture` |
| Comportamento, regras de negócio, contratos e critérios de aceitação | `agentflow-trd` |
| Estilo de código, testes, segurança, Git e revisão | `agentflow-engineering-standards` |

Se o conflito não cair numa linha da tabela, pare e reporte os ids e os commits, como manda `docs-resolution`.

## Glossário

| Termo | Significado |
| ----- | ----------- |
| DEVE | Obrigatório. A ausência é defeito. |
| NÃO DEVE | Proibido. A presença é defeito. |
| PODE | Permitido. Não é obrigatório e não cria exceção a um DEVE. |
| `STD-<SIGLA>-<NNN>` | Id estável de um padrão deste repositório. |
| `proposed` | Orientação. O agente segue e sinaliza no resultado que a tarefa dependeu do documento. |
| `accepted` | Obrigatório, sem essa ressalva. |
| Gate | Comando de `quality-gates` que precisa terminar com código 0. |
| Bloqueio | Item de `code-review` que impede o merge. |
| Fronteira | Ponto em que dados entram de outro contexto: mensagem `chrome.runtime`, DOM da loja, IndexedDB ou `chrome.storage`. |

## Questões em aberto

Nenhuma neste documento. Os limiares numéricos estão nas questões em aberto de `code-style`, `testing`, `security` e `quality-gates`.

## Regras para agentes

- DEVE consultar este repositório em toda implementação, teste, revisão e abertura de pull request.
- DEVE citar o id `STD-*` no resultado da tarefa quando a mudança depender dele.
- DEVE tratar documento `proposed` como orientação e dizer isso no resultado. Documento `accepted` é obrigatório.
- NÃO DEVE copiar para cá regras de estrutura ou de comportamento. Aponte `agentflow-architecture:<id>` ou `agentflow-trd:<id>`.
- NÃO DEVE inventar `consult_when`, sigla `STD` ou seção H2 fora de `manifest.yaml`.
