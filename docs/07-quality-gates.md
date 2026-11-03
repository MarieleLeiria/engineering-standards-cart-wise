---
id: quality-gates
title: "Gates de qualidade"
type: process
status: proposed
consult_when: [validate-work, commit-and-pr]
applies_to: ["src/**", "tests/**", "package.json", ".github/**"]
related: [overview, code-style, testing, security, git-workflow, code-review, agentflow-architecture:adr-0001]
tags: [ci, qualidade, definicao-de-pronto]
---

# Gates de qualidade

## Objetivo

Definir a sequência fechada de comandos que uma mudança precisa passar antes do merge, e o que conta como tarefa pronta. A stack de teste e build está em `agentflow-architecture:adr-0001`. Os limiares de cobertura e de audit estão em `testing` e `security`.

## Entradas

- A árvore do produto no commit que será revisado.
- `package-lock.json` presente e consistente com `package.json`.
- Os scripts de `STD-QUAL-001` declarados no `package.json`.

## Passos

Os gates rodam nesta ordem. O próximo só começa se o anterior saiu com código 0. O CI usa a mesma ordem. Não há gate opcional.

### STD-QUAL-001 Scripts

O `package.json` do produto declara estes scripts, com estes comandos:

| Script | Comando |
| ------ | ------- |
| `lint` | `eslint .` |
| `format:check` | `prettier --check .` |
| `typecheck` | `tsc --noEmit` |
| `test` | `vitest run --coverage` |
| `test:e2e` | `playwright test` |
| `build` | `vite build` |
| `audit` | `npm audit --audit-level=high` |

ESLint usa `typescript-eslint` em `recommendedTypeChecked`, `eslint-plugin-react-hooks`, `eslint-plugin-react` e `eslint-config-prettier`. As regras que implementam `code-style` ficam em `error`: `no-explicit-any`, `no-floating-promises`, `consistent-type-imports` e `no-console` permitindo só `warn` e `error`. `no-restricted-syntax` fica em `error` para chamadas a `fetch`, `XMLHttpRequest`, `WebSocket` e `sendBeacon` em `src/**` (`STD-SEC-003`). Prettier não é duplicado dentro do ESLint.

### STD-QUAL-002 Ordem no CI

1. `npm ci`
2. `npm run lint`
3. `npm run format:check`
4. `npm run typecheck`
5. `npm run test`
6. `npm run test:e2e`
7. `npm run build`
8. `npm run audit`

`npm ci` falha se o lockfile estiver ausente ou divergente. Não use `npm install` no CI.

### STD-QUAL-003 Cobertura e audit

`npm run test` aplica os mínimos de `STD-TEST-005`. `npm run audit` aplica `STD-SEC-006`. Falha nesses dois é falha de gate, igual à falha de compilação.

### STD-QUAL-004 Definição de pronto

A tarefa só está pronta quando todos os itens abaixo são verdadeiros no mesmo commit:

- Os oito comandos de `STD-QUAL-002` saíram com código 0.
- Cada critério de aceitação tocado tem teste com o id no nome (`STD-TEST-001`).
- A revisão `STD-REV-001` a `STD-REV-008` está `ok`.
- A descrição do pull request contém `docs_pins` (`STD-GIT-003`).
- Documento `proposed` usado na tarefa foi citado no resultado.

Faltar um item significa que a tarefa não está pronta. Não abra o pull request como rascunho para contornar a lista, salvo se a tarefa for explicitamente um rascunho e o resultado disser isso.

### STD-QUAL-005 Playwright em todo pull request

`npm run test:e2e` roda em todo pull request, não só quando o diff toca UI. O conjunto é o de `tests/e2e/`. `retries` permanece 0 (`STD-TEST-004`).

## Saídas

- CI verde no commit do pull request.
- Resultado da tarefa listando cada script e o código de saída.
- Merge permitido só com a definição de pronto de `STD-QUAL-004` completa.

## Falhas

| Falha | Comportamento |
| ----- | ------------- |
| Qualquer gate com código diferente de 0 | Pare. Não faça merge. Corrija a causa, não o limiar. |
| Script de `STD-QUAL-001` ausente ou com outro comando | O gate não existe. Trate como falha. |
| Cobertura abaixo de `STD-TEST-005` | Falha. Não exclua o arquivo da medição. |
| `npm audit` com achado `high` ou `critical` | Falha. Não use `--force` nem abaixe o nível. |
| E2E pulado por filtro de caminho | Falha enquanto `STD-QUAL-005` estiver vigente. |

Não desative uma regra de ESLint no arquivo para passar o lint. Se a regra estiver errada, proponha a mudança neste repositório.

## Questões em aberto

- Confirmar se `STD-QUAL-005` permanece em todo pull request antes de mudar este documento para `accepted`. A alternativa é rodar Playwright só quando o diff toca `src/ui/**`, `src/content/**` ou `src/adapters/**`.

## Regras para agentes

- DEVE rodar os comandos de `STD-QUAL-002` na ordem, no commit que será entregue.
- DEVE registrar no resultado o código de saída de cada um.
- NÃO DEVE fazer merge com gate vermelho, script ausente ou cobertura contornada.
- NÃO DEVE trocar o comando dos scripts de `STD-QUAL-001` sem atualizar este documento e o `manifest.yaml` no mesmo commit.
- DEVE tratar a definição de pronto de `STD-QUAL-004` como a lista fechada. Não acrescente gate informal nem dispense item.
- NÃO DEVE pular `npm run test:e2e` (`STD-QUAL-005`).
