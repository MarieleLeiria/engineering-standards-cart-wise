---
id: testing
title: "Testes"
type: standard
status: proposed
consult_when: [write-tests, validate-work]
applies_to: ["src/**", "tests/**"]
related: [overview, file-organization, quality-gates, code-review, agentflow-trd:nfr, agentflow-architecture:adr-0001]
tags: [vitest, playwright, cobertura, nfr]
---

# Testes

## Objetivo

Fazer cada critério de aceitação e cada prazo de `agentflow-trd:nfr` ter um teste que cita o id do requisito e falha quando o comportamento volta atrás. A ferramenta está escolhida em `agentflow-architecture:adr-0001`: Vitest e Playwright.

## Padrões

### STD-TEST-001 Um teste por critério

Cada critério **Dado / Quando / Então** de `agentflow-trd` tem ao menos um teste. O nome do teste contém o id do requisito, no formato `FR-CAP-001 extrai o nome quando o preço já está no DOM`. O teste chama o comportamento real do módulo, não uma cópia da regra escrita dentro do próprio teste.

Critério que o documento marca como fora de escopo não ganha teste que exija o comportamento.

### STD-TEST-002 Pirâmide

| Camada | Ferramenta | Cobre | Arquivo |
| ------ | ---------- | ----- | ------- |
| Unidade | Vitest | Função pura, schema Zod, dedupe, formatação | `<módulo>.test.ts` ao lado do módulo |
| Adaptador | Vitest + fixture HTML | `matches`, `isAddToCartTarget`, `extract` | `src/adapters/<loja>/<adaptador>.test.ts` |
| Ponta a ponta | Playwright com a extensão carregada | Um fluxo por feature do MVP | `tests/e2e/<feature>.spec.ts` |

Não use Playwright para regra que um teste de unidade já cobre. Não use Vitest para afirmar que o botão da loja real, na internet, continua com o mesmo HTML.

### STD-TEST-003 Sem rede e sem relógio de parede

Teste de Vitest não abre socket. `fetch`, `XMLHttpRequest`, `WebSocket` e `sendBeacon` ficam indisponíveis no setup do Vitest. Fixture é arquivo local. Não baixe HTML de loja durante o teste.

Prazo de `NFR-PERF-001`, `NFR-PERF-002` e `NFR-PERF-003` é verificado com relógio falso (`vi.useFakeTimers`). O teste avança o tempo até o limite documentado em `agentflow-trd:nfr` e falha se o código precisar de mais. Não use `Date.now()` real, `setTimeout` real nem a duração do runner como evidência. O nome do teste contém o id `NFR-*`.

Não copie o número do prazo para uma segunda fonte. O teste importa a constante usada em produção, e essa constante tem o valor do `NFR`.

### STD-TEST-004 Determinismo

Não há `Math.random`, ordem de objeto instável nem dependência de fuso para o que o teste afirma. `chrome.*` é substituído por um fake no teste de unidade. O teste não lê o perfil do navegador de quem está rodando.

Corrija o teste que falha de forma intermitente. Não use `retry` para esconder a falha. Playwright pode ter `retries: 0` no CI.

### STD-TEST-005 Cobertura

O gate de cobertura do Vitest, medido em `src/**` e ignorando `**/*.test.ts`, `**/*.test.tsx`, `**/fixtures/**` e `src/**/*.d.ts`, é:

| Métrica | Mínimo |
| ------- | ------ |
| lines | 80 |
| statements | 80 |
| functions | 80 |
| branches | 80 |

Arquivo novo em `src/` entra na conta. Não exclua um arquivo da cobertura para fazer o gate passar. Teste de ponta a ponta não entra nesse percentual.

### STD-TEST-006 Correção de defeito

Correção de defeito inclui um teste que falha no código anterior e passa no código corrigido. O nome do teste diz o defeito em uma linha. Sem esse teste, a correção não está pronta.

## Exemplos

Viola `STD-TEST-001` e `STD-TEST-003`:

```ts
it("funciona", async () => {
  await new Promise((resolve) => setTimeout(resolve, 500));
  expect(true).toBe(true);
});
```

Cumpre os dois:

```ts
it("NFR-PERF-001 conclui a extração dentro do prazo quando o preço já está no DOM", async () => {
  vi.useFakeTimers();
  const pending = extract(fixture, url);
  await vi.advanceTimersByTimeAsync(MAX_EXTRACT_MS);
  await expect(pending).resolves.toMatchObject({ status: "ok" });
});
```

`MAX_EXTRACT_MS` é a constante de produção. O valor dela é o de `NFR-PERF-001` em `agentflow-trd:nfr`.

## Questões em aberto

- Confirmar o mínimo de 80% de `STD-TEST-005` antes de mudar este documento para `accepted`.

## Regras para agentes

- DEVE criar ou atualizar o teste do critério de aceitação na mesma mudança que implementa o critério (`STD-TEST-001`).
- DEVE colocar o id `FR-*` ou `NFR-*` no nome do teste.
- NÃO DEVE acessar a rede, o relógio de parede ou o Chrome real em teste de Vitest (`STD-TEST-003`, `STD-TEST-004`).
- DEVE manter a cobertura de `STD-TEST-005`. NÃO DEVE excluir arquivo de `src/` da medição para passar.
- DEVE acompanhar correção de defeito com um teste que falhava antes (`STD-TEST-006`).
- NÃO DEVE marcar `retries` maior que zero no Playwright do CI.
