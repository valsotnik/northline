# SFE-6 Step 2 API project proof

- **Ticket:** SFE-6
- **Branch:** `codex/SFE-6-portfolio-api-boundary`
- **Date:** 2026-09-04
- **Scope:** Create only the smallest API project needed to prove Nx build, lint, type-check, test, and project-graph integration.

## Change summary

Added a minimal NestJS API application at `apps/api` using the Nx-matched Nest plugin:

- `@nx/nest@23.1.2`
- NestJS `11.x` runtime packages
- Jest-based API unit-test target
- Webpack-based API build target
- no e2e project
- no frontend proxy
- no portfolio endpoint, OpenAPI contract, authentication, database, or deployment behavior

The generated `.vscode/launch.json` file was removed because repository instructions exclude local
IDE state from commits.

## Repository command alignment

The root package scripts now run every project that owns the selected target:

```bash
pnpm build
pnpm lint
pnpm typecheck
pnpm test
pnpm check
```

The API project also has an explicit `typecheck` target:

```bash
tsc -p apps/api/tsconfig.app.json --noEmit
tsc -p apps/api/tsconfig.spec.json --noEmit
```

This keeps the SFE-5 quality gate meaningful after adding a second Nx project.

## Integration friction found and resolved

The generated Nest application initially failed this repository's stricter TypeScript and ESLint
settings:

- `api:build` failed because the API inherited `verbatimModuleSyntax` from the Angular-oriented
  root TypeScript config while the generated Nest build used CommonJS output.
- `api:test` failed because the generated Jest tsconfig did not emit Nest decorator metadata.
- TypeScript 6 rejected the generated `node10` module-resolution path.
- `api:lint` rejected generated imports that were only used as types.
- `main.ts` used `process.env.PORT`, which violates the repository's
  `noPropertyAccessFromIndexSignature` setting.

The API project now localizes its Node/Nest TypeScript settings with `module` and `moduleResolution`
set to `node16`, keeps decorator metadata enabled for app and test compilation, overrides the
generated Jest compiler environment to avoid `node10`, and uses bracket access for `process.env`.

## Nx graph evidence

`pnpm exec nx show projects` returned:

```json
["api", "web"]
```

`pnpm exec nx graph --file=/tmp/sfe-6-step-2-graph.json` produced a JSON graph with these project
nodes:

```json
["api", "web"]
```

The graph has no project dependencies yet:

```json
{
  "api": [],
  "web": []
}
```

That is intentional for this step. The API project is present in the Nx graph, but the Angular app is
not coupled to it before the OpenAPI transport-contract proof.

`pnpm exec nx show project api` confirmed these relevant targets:

- `api:build`
- `api:serve`
- `api:lint`
- `api:typecheck`
- `api:test`

## Verification

Focused API checks passed:

```bash
pnpm exec nx run api:lint
pnpm exec nx run api:typecheck
pnpm exec nx run api:test
pnpm exec nx run api:build
```

Root repository checks passed:

```bash
pnpm typecheck
pnpm check
```

The first sandboxed `pnpm check` passed formatting, linting, type-checking, both project test
suites, and `api:build`, then failed during `web:build:production` without Angular diagnostics. This
matches the sandbox limitation already recorded in prior verification notes. The rerun outside the
sandbox passed completely; Nx reported `web:build:production` as flaky because it observed the
preceding sandbox-induced failure.

## Step 3 recommendation

Proceed to the contract proof without adding product behavior yet:

- generate or hand-author one minimal OpenAPI document for a proof endpoint;
- generate frontend-owned transport types from that OpenAPI document;
- keep Angular code from importing NestJS DTOs or backend implementation types; and
- record any generator peer-dependency friction before selecting the Step 4 adapter shape.

## Sources

- Nx Nest plugin guide: https://nx.dev/docs/technologies/node/nest/introduction
- Nx Nest application generator reference: https://nx.dev/docs/technologies/node/nest/generators
