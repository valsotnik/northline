# SFE-6 Step 3 OpenAPI type generation

- **Ticket:** SFE-6
- **Branch:** `codex/SFE-6-portfolio-api-boundary`
- **Date:** 2026-09-04
- **Scope:** Generate a minimal OpenAPI document and prove frontend transport-type generation without importing backend implementation types.

## Change summary

Added a minimal OpenAPI 3.1 proof contract:

- `docs/contracts/portfolio-summary-proof.openapi.json`
- versioned server URL: `/api/v1`
- proof path: `GET /portfolio-summary-proof`
- response schema: `PortfolioSummaryProofResponse`

The proof contract includes the portfolio-summary transport shape needed by the later success-path
story: portfolio identity, valuation currency, total value, signed daily movement, largest exposure,
observation timestamp, and freshness. It deliberately does not add a production Nest endpoint or
Angular UI behavior in this step.

Added a pinned OpenAPI generator:

- `@hey-api/openapi-ts@0.99.0`

Added a repeatable generation command:

```bash
pnpm contract:generate
```

That script runs:

```bash
openapi-ts -i docs/contracts/portfolio-summary-proof.openapi.json -o apps/web/src/app/api/generated -p @hey-api/typescript
prettier --write apps/web/src/app/api/generated
```

The explicit TypeScript plugin keeps the proof type-only, and the Prettier pass keeps generated
source compatible with the repository formatting gate.

## Generated frontend transport types

`pnpm contract:generate` produced:

- `apps/web/src/app/api/generated/index.ts`
- `apps/web/src/app/api/generated/types.gen.ts`

The generated output includes these transport types:

- `PortfolioSummaryProofResponse`
- `SignedMovement`
- `LargestExposure`
- `GetPortfolioSummaryProofData`
- `GetPortfolioSummaryProofResponse`
- `GetPortfolioSummaryProofResponses`

## Boundary evidence

The generated frontend files are produced from OpenAPI, not from NestJS DTOs or backend source.

This search returned no matches:

```bash
rg -n "@nestjs|apps/api|app\\.controller|app\\.service|from '.*/api/|from \".*/api/" apps/web/src/app/api/generated apps/web/src/app
```

The result confirms that the Angular application and generated transport files do not import:

- NestJS packages;
- `apps/api` files;
- generated Nest controller or service classes.

## Verification

Focused checks passed:

```bash
pnpm contract:generate
pnpm typecheck
pnpm exec tsc -p apps/web/tsconfig.app.json --noEmit
```

The first formatting check failed because the raw generated files did not match the repository's
Prettier style. The generation script now formats the generated folder after codegen.

## Step 4 recommendation

Proceed to mapping only after treating the generated transport types as an external boundary:

- import generated transport types only into a narrow frontend adapter;
- define frontend-owned view state separately from the generated transport shape;
- keep Nest controller/service classes out of Angular imports; and
- preserve the generated folder as codegen output that is refreshed through `pnpm contract:generate`.

## Sources

- Hey API OpenAPI TypeScript README: https://github.com/hey-api/openapi-ts
- Hey API OpenAPI TypeScript introduction: https://hey-api-openapi-ts.mintlify.app/introduction
