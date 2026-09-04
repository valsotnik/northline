# SFE-6 final spike decision

- **Ticket:** SFE-6
- **Branch:** `codex/SFE-6-portfolio-api-boundary`
- **Date:** 2026-09-04
- **Scope:** Compare the completed proof with ADR 0001 and record the final API placement and
  contract recommendation.

## Step 4 comparison

ADR 0001 kept `apps/web` as the only Nx project until a concrete seam justified another project.
SFE-6 provides that evidence for the API boundary:

| ADR 0001 rule or trigger                                                                                          | SFE-6 evidence                                                                                                                                         | Decision                                                                                       |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| Add another application only when there is a concrete seam.                                                       | The first portfolio workflow needs an HTTP API boundary, and the proof adds an independently addressable Nest application at `apps/api`.               | The API project is justified for the initial portfolio API boundary.                           |
| Keep a future API outside the workspace by default unless sharing the repo and task graph has coordination value. | `api` and `web` share one root lockfile, root scripts, Nx graph, and green quality gate for the walking-skeleton stage.                                | Keep the initial API in this workspace.                                                        |
| Add a project only when it can follow the root toolchain lifecycle.                                               | `api:lint`, `api:typecheck`, `api:test`, `api:build`, `pnpm typecheck`, and `pnpm check` pass with pinned repository dependencies.                     | The API can follow the current lifecycle.                                                      |
| Prefer deep modules and avoid shallow speculative libraries.                                                      | The OpenAPI proof has one frontend consumer and one API proof; no second consumer or stable library interface exists.                                  | Do not create a shared contract library yet.                                                   |
| Add adapters at real external or interchangeable boundaries.                                                      | The OpenAPI document is the transport boundary; generated frontend types stay out of Nest source and implementation classes stay out of Angular code.  | Use OpenAPI-generated frontend transport types, then map them into frontend-owned state later. |
| Record the API choice in a new ADR before treating it as the implementation path.                                 | ADR 0003 records placement, contract strategy, alternatives, consequences, extraction triggers, NestJS upgrade triggers, and follow-up recommendation. | ADR 0003 is the accepted SFE-6 decision.                                                       |

## Acceptance criteria mapping

| Acceptance criterion                                                                                     | Evidence                                                                                                                                                                                                           |
| -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| A documented proof confirms the proposed API can build and test through repository commands.             | `docs/verification/SFE-6-step-2-api-project-proof-2026-09-04.md`; focused API commands and root `pnpm check` passed.                                                                                               |
| The Nx project graph shows the intended API project and relevant task relationships.                     | `pnpm exec nx show projects` returns `api` and `web`; `pnpm exec nx show project api` shows build, serve, lint, type-check, and test targets.                                                                      |
| The frontend contract proof does not import NestJS DTO classes or backend implementation types.          | `docs/verification/SFE-6-step-3-openapi-type-generation-2026-09-04.md`; boundary search across generated frontend API files and Angular app source returned no matches for Nest/API implementation imports.        |
| Framework and Nx plugin versions are compatible and pinned through the repository dependency mechanism.  | `docs/verification/SFE-6-step-1-version-compatibility-2026-09-04.md`; `@nx/nest@23.1.2` and `@hey-api/openapi-ts@0.99.0` are pinned in `package.json` and `pnpm-lock.yaml`, with NestJS 11 selected for the proof. |
| An ADR explains why the API belongs in the current workspace and when that decision should be revisited. | `docs/decisions/0003-use-in-repository-nest-api-with-openapi-contract.md`, sections "Decision", "Consequences", and "Extraction triggers".                                                                         |
| The ADR records why NestJS 11 is used initially and what evidence would justify NestJS 12.               | `docs/decisions/0003-use-in-repository-nest-api-with-openapi-contract.md`, sections "Decision", "Alternatives considered", and "NestJS upgrade trigger".                                                           |
| The spike concludes with a decision, evidence, and follow-up implementation recommendation.              | `docs/decisions/0003-use-in-repository-nest-api-with-openapi-contract.md`, section "Follow-up recommendation"; this final evidence file summarizes the decision and proof.                                         |

## Final decision

Proceed with an in-repository NestJS 11 API application at `apps/api` for the first portfolio API
implementation, using OpenAPI as the transport-contract source and generated frontend transport
types under `apps/web/src/app/api/generated`.

Do not share NestJS DTO classes with Angular, do not introduce a contract library yet, and do not
adopt NestJS 12 until the recorded upgrade trigger is met.

## Verification commands

```bash
pnpm contract:generate
pnpm exec nx show projects
pnpm exec nx show project api
pnpm typecheck
pnpm check
git diff --check
```

The latest `pnpm check` run passed for formatting, linting, type-checking, tests, and builds across
`api` and `web`. Earlier SFE-6 verification observed a sandbox-only Angular production build failure
with no diagnostics; subsequent full-gate runs passed, and Nx flagged `web:build:production` as
flaky because it observed both outcomes.

## Follow-up implementation recommendation

The next portfolio implementation ticket should use this boundary, not expand it:

- implement the smallest synthetic `GET /api/v1/portfolio-summary` endpoint;
- update the OpenAPI contract and regenerate frontend transport types;
- map generated transport types into frontend-owned view state through a narrow adapter;
- add user-visible tests for introduced loading, success, stale, empty, and failure states; and
- keep authentication, persistence, deployment topology, and shared contract packaging out of scope
  until explicitly approved.
