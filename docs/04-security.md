---
id: security
title: "Segurança"
type: standard
status: proposed
consult_when: [write-code, change-security]
applies_to: ["src/**", "manifest.config.*", "package.json", "package-lock.json"]
related: [overview, code-style, quality-gates, agentflow-trd:nfr, agentflow-trd:stores, agentflow-architecture:adr-0001, agentflow-architecture:adr-0008]
tags: [mv3, xss, permissoes, dependencias]
---

# Segurança

## Objetivo

Fixar as verificações que impedem a extensão de executar HTML da loja, abrir rede, alargar permissão ou gravar segredo. O que é proibido no produto está em `agentflow-trd:nfr`. Aqui está como o agente prova que a mudança não cruzou essa linha.

## Padrões

### STD-SEC-001 HTML externo não é executado

Dados de página, URL, nome, preço e variação não passam por `eval`, `new Function`, `innerHTML`, `outerHTML`, `document.write` nem `insertAdjacentHTML`.

No React, isso inclui `dangerouslySetInnerHTML` (`STD-CODE-006`). O toast é o Shadow DOM fechado de `agentflow-architecture:adr-0008`; o texto do toast também é nó de texto, não HTML.

### STD-SEC-002 Mensagem só entra validada

Handler de `chrome.runtime.onMessage` valida o payload com o schema Zod de `src/shared` antes de ler campo. Payload inválido recebe o `reason` já catalogado. Não confie no remetente só porque a mensagem veio de outra parte da própria extensão.

O catálogo de mensagens não é redefinido aqui. Ele está em `agentflow-architecture:components`.

### STD-SEC-003 Sem saída de rede

O código da extensão não chama `fetch`, `XMLHttpRequest`, `WebSocket` nem `sendBeacon`. Não há analytics, log remoto nem relatório de erro. A regra de produto é `NFR-PRIV-001` e `NFR-PRIV-002` em `agentflow-trd:nfr`.

O ESLint falha, pela regra `no-restricted-syntax` configurada em `quality-gates` (`STD-QUAL-001`), se `src/**` chamar uma dessas quatro APIs. Imagem exibida pela UI continua sendo URL `https` carregada pelo navegador, não download feito pela extensão.

### STD-SEC-004 Permissões

`manifest.config.ts` é o único lugar que declara permissão (`STD-FILE-006`, `agentflow-architecture:adr-0001`). A lista fechada está em `agentflow-trd:nfr` (`NFR-PRIV-003`) e os hosts em `agentflow-trd:stores`.

Um diff que altera `permissions`, `host_permissions` ou `optional_host_permissions` cita o id do `NFR` ou do documento `stores` no corpo do pull request. Sem essa citação, a mudança está bloqueada. `<all_urls>` em `host_permissions` é proibido.

### STD-SEC-005 Segredos

Não há token, chave ou cookie de sessão no repositório, no código, na fixture nem no workflow de CI. Não adicione `.env` com segredo. A extensão do MVP não tem credencial de serviço: se a tarefa pedir uma, pare e reporte o conflito com `agentflow-trd:nfr`.

`.gitignore` ignora `.env` e `.env.*`. `.env.example`, se existir, só contém nomes de variável vazios, e hoje nenhuma é necessária.

### STD-SEC-006 Dependências

O gerenciador é npm. `package-lock.json` é commitado. Não troque para Yarn ou pnpm. Não use `*` nem `latest` em `package.json`.

O CI roda `npm audit --audit-level=high`. Achado `high` ou `critical` falha o gate. Dependência nova no diff precisa estar no lockfile do mesmo commit. Não adicione pacote para uma função que a biblioteca padrão ou uma dependência já presente cobre.

Script `postinstall` de dependência não é motivo para afrouxar o audit.

### STD-SEC-007 CSP da extensão

Não afrouxe `content_security_policy` do manifest. Não peça `unsafe-eval` nem `unsafe-inline` em `extension_pages`. Manifest V3 já restringe isso; a mudança que reabre está bloqueada.

## Exemplos

Viola `STD-SEC-001` e `STD-SEC-003`:

```ts
host.innerHTML = product.name;
await fetch("https://telemetry.example/event", { method: "POST", body: product.productUrl });
```

Cumpre os dois:

```ts
host.textContent = product.name;
console.error("[content] price_not_found");
```

O nome vem do schema. O erro fica no console da extensão.

## Questões em aberto

- Decidir se achado `moderate` do `npm audit` também bloqueia, antes de mudar este documento para `accepted`. Hoje só `high` e `critical` bloqueiam (`STD-SEC-006`).

## Regras para agentes

- NÃO DEVE inserir dado de página como HTML, nem usar `eval` ou `new Function` (`STD-SEC-001`).
- DEVE validar com Zod todo payload de mensagem antes de usá-lo (`STD-SEC-002`).
- NÃO DEVE adicionar `fetch`, `XMLHttpRequest`, `WebSocket` ou `sendBeacon` em `src/` (`STD-SEC-003`).
- NÃO DEVE alterar permissão sem citar `agentflow-trd:nfr` ou `agentflow-trd:stores` no pull request (`STD-SEC-004`).
- NÃO DEVE gravar segredo no repositório (`STD-SEC-005`).
- DEVE manter `package-lock.json` e fazer `npm audit --audit-level=high` passar (`STD-SEC-006`).
- NÃO DEVE afrouxar a CSP da extensão (`STD-SEC-007`).
