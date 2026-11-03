# agentflow-engineering-standards

Fonte de conhecimento de **padrões de engenharia** do CartWise para os agentes do AgentFlow: código, organização de arquivos, testes, segurança, Git, revisão e qualidade.

O CartWise é uma extensão de navegador que, quando o usuário adiciona um item ao carrinho de uma loja, captura nome, fotos, site, preço e variação do produto e reúne os itens salvos em uma única lista.

Este repositório faz parte de um conjunto de três (ver `agentflow-architecture:adr-0007`):

| Repositório | Quando o agente consulta |
| ----------- | ------------------------ |
| `agentflow-architecture` | Antes de definir ou alterar a estrutura do sistema. |
| `agentflow-trd` | Antes de implementar uma feature. |
| `agentflow-engineering-standards` (este) | Durante a implementação e a validação de qualquer feature. |

Em conflito, este repositório prevalece em estilo de código, testes, segurança, Git e revisão. A tabela completa e a regra de parar estão em `agentflow-architecture:docs-resolution`.

## Estrutura

```
manifest.yaml          índice legível por máquina (ponto de entrada do AgentFlow)
docs/                  padrões transversais
templates/             modelo para um padrão novo
```

| Id | Documento | Conteúdo |
| -- | --------- | -------- |
| `overview` | [docs/00-overview.md](docs/00-overview.md) | Precedência, glossário e ids `STD`. |
| `code-style` | [docs/01-code-style.md](docs/01-code-style.md) | TypeScript, React, Zod, ESLint e Prettier. |
| `file-organization` | [docs/02-file-organization.md](docs/02-file-organization.md) | Onde colocar arquivo e quem pode importar quem. |
| `testing` | [docs/03-testing.md](docs/03-testing.md) | Vitest, fixtures, Playwright e cobertura. |
| `security` | [docs/04-security.md](docs/04-security.md) | XSS, permissões, rede, segredos e dependências. |
| `git-workflow` | [docs/05-git-workflow.md](docs/05-git-workflow.md) | Branch, Conventional Commits, PR e squash. |
| `code-review` | [docs/06-code-review.md](docs/06-code-review.md) | Checklist e o que bloqueia o merge. |
| `quality-gates` | [docs/07-quality-gates.md](docs/07-quality-gates.md) | Comandos de CI e definição de pronto. |

## Protocolo de consumo pelo AgentFlow

1. Ler `agentflow.docs.lock.yaml` no repositório do produto e obter o **commit SHA completo** deste repositório.
2. Ler `manifest.yaml` nesse commit.
3. Selecionar os documentos cujo `consult_when` contém `any-task` ou intersecta os tipos da tarefa, ou cujo `applies_to` casa com os caminhos que a tarefa toca.
4. Ignorar documentos `superseded`, `deprecated` e `rejected`. Tratar `accepted` como obrigatório e `proposed` como orientação sinalizada.
5. Entregar ao agente o conteúdo dos documentos, destacando `## Regras para agentes`.
6. Registrar em `docs_pins` o commit e os ids usados.

Uma tarefa de implementação declara os `consult_when` deste repositório junto com os de arquitetura e de TRD. Exemplos:

| Tarefa | `consult_when` |
| ------ | -------------- |
| Implementar ou alterar código | `write-code`, `write-tests`, `validate-work` |
| Criar arquivo, diretório ou módulo | `write-code`, `add-module` |
| Permissões, HTML externo, segredo ou dependência | `write-code`, `change-security` |
| Branch, commit ou pull request | `commit-and-pr` |
| Revisar antes do merge | `review-code`, `validate-work` |

`write-code`, `add-module`, `write-tests`, `change-security`, `commit-and-pr`, `review-code` e `validate-work` existem só neste repositório. Não os invente nos outros dois.

## Contrato dos documentos

O mesmo de `agentflow-architecture:adr-0006`.

- `type`, `status` e `consult_when` vêm do `vocabulary` deste `manifest.yaml`.
- A ordem das seções H2 está em `section_order`.
- Todo documento termina a parte normativa com `## Regras para agentes`.
- Padrões usam id `STD-<SIGLA>-<NNN>` (`CODE`, `FILE`, `TEST`, `SEC`, `GIT`, `REV`, `QUAL`). O id não muda quando o texto muda.
- Referências cruzadas usam `<repositorio>:<id>`, por exemplo `agentflow-trd:nfr`.
- Este repositório não guarda ADR. Decisão de estrutura vai para `agentflow-architecture`.

## Como contribuir

- **Novo padrão:** copie [templates/standard.md](templates/standard.md), use a próxima sigla livre e registre o documento em `manifest.yaml` antes de usar um `consult_when` novo.
- **Novo `consult_when`:** entre primeiro no `vocabulary` do manifesto. Não invente o valor no frontmatter.
- **Toda mudança:** mantenha frontmatter e `manifest.yaml` idênticos nos campos `id`, `type`, `status`, `consult_when`, `applies_to` e `tags`.

Os produtos só passam a usar a mudança quando o SHA for atualizado no `agentflow.docs.lock.yaml` deles.

## Estado atual

Todos os documentos estão como `proposed`. Valores assumidos, ainda não aceitos:

- `noUncheckedIndexedAccess` desligado (`code-style`).
- Cobertura mínima de 80% em linhas, funções, ramos e statements (`testing`, `STD-TEST-005`).
- `npm audit` bloqueia em `high` e `critical`; `moderate` não bloqueia (`security`, `STD-SEC-006`).
- Playwright roda em todo pull request (`quality-gates`, `STD-QUAL-005`).
