---
id: code-style
title: "Estilo de código"
type: standard
status: proposed
consult_when: [write-code]
applies_to: ["src/**"]
related: [overview, file-organization, security, quality-gates, agentflow-architecture:adr-0001]
tags: [typescript, react, eslint, prettier]
---

# Estilo de código

## Objetivo

Fazer com que código escrito por agentes saia no mesmo formato, com os mesmos tipos e com os mesmos pontos de validação. A escolha da stack está em `agentflow-architecture:adr-0001`. Este documento só fixa como usá-la.

## Padrões

### STD-CODE-001 TypeScript strict

Código de produção é TypeScript com `strict: true` no `tsconfig`. Não há arquivo `.js` nem `.jsx` em `src/`. São proibidos `any`, `as any`, `@ts-ignore` e `@ts-nocheck`.

`as` só é permitido para estreitar um tipo que o compilador não acompanha, na linha imediatamente anterior a um comentário `// STD-CODE-001: <motivo>`. Não use `as` para calar um erro.

### STD-CODE-002 Zod em toda fronteira

Todo valor que cruza uma fronteira passa pelo schema Zod definido em `src/shared` antes de ser usado. Fronteira aqui é mensagem `chrome.runtime`, resultado de adaptador, registro lido do IndexedDB e valor lido de `chrome.storage`.

Não crie schema Zod fora de `src/shared`. Não duplique um schema que já existe. O catálogo de mensagens e de `reason` continua em `agentflow-architecture:components` e `agentflow-trd:contracts`.

### STD-CODE-003 Módulos e funções

Use named export. `export default` só no entry que o bundler da extensão exigir, com o comentário `// STD-CODE-003: entry do bundler` na mesma linha.

Toda função exportada declara o tipo de retorno. Uma função não mistura leitura de DOM de loja, regra de negócio e escrita em storage. Cada uma dessas três fica em função própria.

### STD-CODE-004 Nomes

| Elemento | Forma | Exemplo |
| -------- | ----- | ------- |
| Tipo, interface e componente React | PascalCase | `SavedItem`, `ItemList` |
| Função, variável e constante de módulo | camelCase | `upsertFromCapture`, `maxWaitMs` |
| Booleano | `is`, `has` ou `should` | `isAddToCartTarget` |
| Tipo de mensagem e `reason` | o literal já definido no contrato | `"ITEM_CAPTURED"`, `"price_not_found"` |

Não use `SCREAMING_SNAKE_CASE`. Não abrevie (`prd`, `btn`, `tmp`), salvo o nome já usado no contrato (`id`, `ok`).

### STD-CODE-005 Imports

Tipo é importado com `import type`. A ordem, sem linha em branco no meio de um grupo e com uma linha em branco entre grupos, é:

1. Módulos de pacote (`react`, `zod`).
2. Alias `@/` (veja `file-organization`).
3. Imports relativos.

Não deixe import sem uso. Não crie ciclo entre arquivos.

### STD-CODE-006 React

Componente é função. Não use classe nem `React.FC`. As props ficam em `type Props`, no mesmo arquivo do componente.

Não use `dangerouslySetInnerHTML`. Texto vindo da captura é filho de texto do React, não HTML. Lista de item salvo usa o `id` do item como `key`. Índice do array não é `key`.

### STD-CODE-007 Erros e log

`catch` vazio é proibido. `catch` que descarta o erro também. O tratamento ou relança, ou converte para o `reason` do catálogo, ou registra com `console.error`.

`console.log` e `console.debug` não entram em código entregue. `console.error` e `console.warn` incluem o módulo, no formato `[content]`, `[background]`, `[adapters]`, `[storage]` ou `[ui]`.

Não envie log para fora da extensão. Isso é `agentflow-trd:nfr` (`NFR-PRIV-002`).

### STD-CODE-008 Formatação

Prettier é a única formatação. Não alinhe código à mão e não desative o Prettier num arquivo.

| Opção | Valor |
| ----- | ----- |
| `semi` | `true` |
| `singleQuote` | `true` |
| `trailingComma` | `"all"` |
| `printWidth` | `100` |
| `arrowParens` | `"always"` |

O arquivo é `prettier.config.mjs` na raiz do produto.

## Exemplos

Viola `STD-CODE-001`, `STD-CODE-002` e `STD-CODE-006`:

```ts
export default function save(data: any) {
  box.innerHTML = data.name;
}
```

Cumpre os três:

```ts
export function ItemName({ name }: Props): JSX.Element {
  return <p>{name}</p>;
}
```

`name` já saiu de um schema Zod na fronteira. O componente não valida de novo e não interpreta HTML.

## Questões em aberto

- Ligar `noUncheckedIndexedAccess` no `tsconfig` antes de mudar este documento para `accepted`. Hoje permanece desligado.

## Regras para agentes

- DEVE escrever código novo em TypeScript com `strict: true` (`STD-CODE-001`).
- NÃO DEVE adicionar `.js`, `.jsx`, `any`, `@ts-ignore` ou `@ts-nocheck` em `src/`.
- DEVE validar com Zod, em `src/shared`, todo valor que cruza uma fronteira (`STD-CODE-002`).
- NÃO DEVE usar `export default`, `React.FC`, `dangerouslySetInnerHTML`, `console.log` ou `catch` vazio.
- DEVE formatar com Prettier e a tabela de `STD-CODE-008`, sem exceção por arquivo.
- NÃO DEVE citar um `reason` ou um `type` de mensagem que não exista em `agentflow-trd:contracts` e `agentflow-architecture:components`.
