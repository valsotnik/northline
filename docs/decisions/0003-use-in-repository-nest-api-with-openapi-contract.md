# ADR 0003: Use an in-repository Nest API with an OpenAPI contract

- **Status:** Accepted
- **Date:** 2026-09-04
- **Scope:** SFE-6 portfolio API and contract boundary spike

## Context

Northline needs an HTTP boundary for the first portfolio workflow, but ADR 0001 intentionally kept
`apps/web` as the only Nx project until a concrete product or engineering seam justified another
project. SFE-6 tested whether a minimal API can fit that rule without expanding into a generic
platform or leaking backend implementation types into Angular.

The spike had to answer four questions before portfolio-summary implementation:

1. Can the current Node, Angular, TypeScript, Nx, and Nest toolchain versions coexist?
2. Can a minimal API project participate in repository-owned Nx tasks?
3. Can the frontend receive transport types from an OpenAPI contract without importing NestJS DTOs
   or backend source?
4. What evidence should trigger extraction from this workspace or a NestJS major-version upgrade?

The answer is intentionally limited to architecture and contract placement. It does not approve a
production portfolio endpoint, authentication, database work, deployment topology, GraphQL, realtime
transport, or a broad shared-library split.

## Decision

Use a small NestJS 11 API application in the current Nx workspace for the initial portfolio API
implementation, and keep the Angular/API contract boundary at OpenAPI-generated frontend transport
types.

Concretely:

- Keep the API project at `apps/api` while it shares the root Node, pnpm, TypeScript, Nx, lint,
  test, build, and quality-gate lifecycle.
- Keep `@nx/nest` pinned to the same Nx line as the workspace.
- Use NestJS 11 for the initial API line because the Nx Nest plugin supports Nest `>=10 <12` and
  the local proof is green on Nest 11.
- Defer NestJS 12 until the Nx plugin line supports it or a separate compatibility proof justifies
  an override.
- Treat `docs/contracts/*.openapi.json` as the contract source for frontend transport types.
- Generate Angular-side transport types into `apps/web/src/app/api/generated` through
  `pnpm contract:generate`.
- Keep Angular from importing NestJS decorators, DTO classes, controllers, services, or other API
  implementation types.
- Keep frontend view/domain state separate from generated transport types when the portfolio-summary
  UI is implemented.
- Do not introduce a shared library for the contract yet; one frontend consumer and one proof API do
  not create enough evidence for an additional project.

This decision revisits ADR 0001 for the API boundary only. ADR 0001's restraint still applies to
other applications, libraries, and speculative modules.

## Alternatives considered

### Keep the API outside the workspace

This preserves the original default from ADR 0001 and would be appropriate if the API had a separate
runtime lifecycle, deployment ownership, access policy, or release cadence. It was rejected for the
initial portfolio slice because the proof API currently benefits from the same root dependency
management, project graph, and repository quality gate as the Angular app.

### Continue with frontend-only mocks

This would minimize near-term setup and keep SFE-7 style UI work moving quickly. It was rejected
because the first portfolio outcome needs a real HTTP boundary and generated contract proof before
feature code depends on accidental mock shapes.

### Share NestJS DTO classes with Angular

Sharing DTO classes would appear to remove duplication, but it couples the Angular app to NestJS
decorators, backend module format assumptions, and server implementation ownership. It was rejected
because OpenAPI gives the client a transport contract without importing backend code.

### Create a shared contract library now

A library could later be useful when multiple consumers need the same generated contract artifact.
It was rejected for now because there is only one frontend consumer and no stable multi-consumer
interface. Keeping generated transport types inside `apps/web` is simpler and matches ADR 0001's
preference for avoiding shallow speculative projects.

### Adopt NestJS 12 now

NestJS 12 would put the API on the newest major line, but `@nx/nest@23.1.2` excludes Nest 12 through
its peer range. It was rejected until the Nx plugin line supports Nest 12 or a separate spike proves
the override, cost, and maintenance risk.

## Consequences

### Positive

- The API and web app are visible in one Nx project graph.
- Root scripts can build, lint, type-check, test, and bundle every project that owns those targets.
- The first portfolio API can be implemented without inventing a second repository or deployment
  process during the walking-skeleton stage.
- The frontend can compile against generated transport types while remaining independent of NestJS
  implementation classes.
- Contract changes have a repeatable generation command instead of hand-maintained TypeScript
  copies.

### Negative and accepted trade-offs

- The repository now has two deployable applications sharing one dependency lifecycle.
- API-specific TypeScript settings must remain localized so Angular-oriented compiler assumptions do
  not leak into Nest builds.
- Generated frontend types must be refreshed when the OpenAPI file changes.
- The proof uses a hand-authored OpenAPI document; production implementation may later decide
  whether the contract is authored first or emitted from Nest metadata.
- A future extraction may require moving the API and rebuilding its CI/deployment boundary after real
  operational evidence exists.

## Evidence

SFE-6 recorded evidence in these files:

- `docs/verification/SFE-6-step-1-version-compatibility-2026-09-04.md`
- `docs/verification/SFE-6-step-2-api-project-proof-2026-09-04.md`
- `docs/verification/SFE-6-step-3-openapi-type-generation-2026-09-04.md`
- `docs/verification/SFE-6-final-spike-decision-2026-09-04.md`

The proof established:

- Node `24.18.0`, pnpm `10.33.0`, Angular `22.0.x`, TypeScript `6.0.3`, Nx `23.1.2`, `@nx/nest`
  `23.1.2`, and NestJS `11.x` are compatible for this workspace.
- `pnpm exec nx show projects` returns `api` and `web`.
- The API project owns build, lint, type-check, test, and serve targets.
- `pnpm contract:generate` produces Angular-side transport types from
  `docs/contracts/portfolio-summary-proof.openapi.json`.
- A source search found no Angular imports of NestJS packages, API source files, generated Nest
  controllers, or generated Nest services.
- `pnpm check` passed outside the filesystem sandbox after a sandbox-only Angular production build
  failure with no diagnostics.

## Extraction triggers

Revisit keeping `apps/api` in this workspace when one or more of the following is observed:

- The API needs independent deployment, release cadence, ownership, access policy, or incident
  response.
- The API requires a materially different Node, TypeScript, framework, build, lint, or test lifecycle
  from `apps/web`.
- API build or test time measurably harms frontend delivery despite Nx affected-task and cache
  behavior.
- A backend team or operational workflow needs repository permissions that should not apply to the
  frontend.
- Multiple clients consume the same API contract and need a separately versioned contract artifact.
- Security, compliance, or deployment requirements make the shared repository boundary too broad.

An extraction trigger starts a new ADR. It does not automatically require moving the API.

## NestJS upgrade trigger

Revisit NestJS 12 when at least one of these conditions is true:

- The Nx Nest plugin line used by the repository supports NestJS 12 in its peer range.
- A current feature requires a NestJS 12 capability that cannot be reasonably achieved on NestJS 11.
- Security or support policy requires leaving NestJS 11.

Before upgrading, record a compatibility proof covering Nx generation/build behavior, TypeScript
settings, Jest, linting, OpenAPI generation or contract generation, and `pnpm check`.

## Follow-up recommendation

Implement the first portfolio-summary slice against this boundary:

- Start with a minimal `GET /api/v1/portfolio-summary` endpoint using synthetic deterministic data.
- Keep the OpenAPI contract as the transport source and regenerate frontend transport types.
- Add a narrow frontend adapter that maps generated transport types into frontend-owned view state.
- Cover loading, success, stale, empty, and failure behavior in user-visible tests when those states
  are introduced.
- Do not add authentication, persistence, trading actions, real customer data, or a shared contract
  library until a later approved ticket requires them.

## References

- [ADR 0001: Use Nx as the workspace orchestrator](0001-use-nx-workspace.md)
- [SFE-6 Step 1 version compatibility](../verification/SFE-6-step-1-version-compatibility-2026-09-04.md)
- [SFE-6 Step 2 API project proof](../verification/SFE-6-step-2-api-project-proof-2026-09-04.md)
- [SFE-6 Step 3 OpenAPI type generation](../verification/SFE-6-step-3-openapi-type-generation-2026-09-04.md)
