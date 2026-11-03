---
id: file-organization
title: "Organização de arquivos"
type: standard
status: proposed
consult_when: [write-code, add-module]
applies_to: ["src/**", "tests/**"]
related: [overview, code-style, testing, agentflow-architecture:components, agentflow-architecture:adr-0001]
tags: [diretorios, dependencias, testes]
---

# Organização de arquivos

## Objetivo

Fazer o agente colocar cada arquivo no diretório do componente dono daquilo, com imports que seguem uma matriz fechada. O mapa de diretórios e as responsabilidades estão em `agentflow-architecture:components`. Este documento não cria componente nem diretório de topo.

## Padrões

### STD-FILE-001 Onde o arquivo nasce

Código novo entra no diretório do componente que já é responsável pelo comportamento. O mapa é o de `agentflow-architecture:components`:

```
src/background/    src/content/    src/adapters/<dominio>/
src/storage/       src/ui/popup/   src/ui/dashboard/    src/shared/
```

Não crie outro diretório diretamente sob `src/`. Se nenhum componente servir, pare e peça a alteração em `agentflow-architecture:components` antes de criar o diretório.

Nome de arquivo de módulo: `kebab-case.ts`. Componente React: `PascalCase.tsx`, dentro de `src/ui/`. Teste fica ao lado do módulo: `item-list.test.ts` ou `ItemList.test.tsx`. O teste de ponta a ponta fica em `tests/e2e/<feature>.spec.ts`.

### STD-FILE-002 Quem importa quem

Mensagem `chrome.runtime` não é import. A matriz vale para `import` de TypeScript.

| De \ Para | content | adapters | shared | background | storage | ui |
| --------- | ------- | -------- | ------ | ---------- | ------- | -- |
| content | sim | sim | sim | não | não | não |
| adapters | não | ver abaixo | sim | não | não | não |
| shared | não | não | sim | não | não | não |
| background | não | não | sim | sim | sim | não |
| storage | não | não | sim | não | sim | não |
| ui | não | não | sim | não | sim | sim |

Dentro de `src/adapters/`, um adaptador de loja só importa o próprio diretório, `src/adapters/shared/` e `src/shared`. Não importa outro adaptador de loja. O registro fica em `src/adapters/registry.ts`, que pode importar os adaptadores.

`src/content/**` não importa `src/storage/**`. Essa aresta está proibida também em `agentflow-architecture:components`.

### STD-FILE-003 Forma do import

O único alias é `@/`, apontando para `src/`. Import entre pastas de topo de `src/` usa `@/`. Import dentro da mesma pasta usa caminho relativo que não sobe (`./arquivo`).

É proibido caminho que sobe dois níveis ou mais (`../../`). É proibido um segundo alias.

### STD-FILE-004 Adaptador e fixture

Uma loja, um diretório: `src/adapters/<dominio-da-loja>/`. O adaptador `generic` fica em `src/adapters/generic/`. Fixture HTML fica em `src/adapters/<dominio-da-loja>/fixtures/*.html` e é carregada pelo teste do adaptador. Não coloque fixture em `tests/`.

O contrato `StoreAdapter`, o registro e a regra de não lançar exceção estão em `agentflow-architecture:components`. Não os redefina aqui.

### STD-FILE-005 Barrel

`index.ts` só reexporta. Não contém função, constante de regra nem schema. Pasta sem necessidade de API pública não ganha `index.ts`. `src/shared/messages.ts` e os schemas continuam arquivos nomeados, não um barrel opaco.

### STD-FILE-006 Permissões e manifest

Permissão, `host_permissions` e `optional_host_permissions` são declarados só em `manifest.config.ts`, como manda `agentflow-architecture:adr-0001`. Não espalhe essa lista em outro arquivo. Os valores permitidos estão em `agentflow-trd:nfr` e `agentflow-trd:stores`.

## Exemplos

Viola `STD-FILE-002` e `STD-FILE-003`:

```ts
import { db } from "../../storage/saved-items";
```

escrito de `src/content/capture.ts`. Content não importa storage, e o caminho sobe dois níveis.

Cumpre os dois, escrito de `src/background/handle-captured.ts`:

```ts
import { savedItems } from "@/storage/saved-items";
import type { CapturedProduct } from "@/shared/messages";
```

## Questões em aberto

Nenhuma.

## Regras para agentes

- DEVE colocar arquivo novo no diretório já mapeado em `agentflow-architecture:components` (`STD-FILE-001`).
- NÃO DEVE criar diretório de topo em `src/` sem atualizar aquele catálogo antes.
- DEVE respeitar a matriz de `STD-FILE-002`. NÃO DEVE importar `src/storage` a partir de `src/content`.
- DEVE usar `@/` entre pastas de topo e caminho relativo só dentro da mesma pasta (`STD-FILE-003`).
- DEVE criar um diretório por loja e colocar a fixture HTML dentro dele (`STD-FILE-004`).
- NÃO DEVE declarar permissão fora de `manifest.config.ts`.
