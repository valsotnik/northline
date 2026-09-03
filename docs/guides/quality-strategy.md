# Quality strategy

This guide records Northline's quality gates and the evidence needed before a new gate becomes
blocking. SFE-5 starts with the smallest stable suite that protects the current Angular workspace
without adding checks that do not yet have useful product signal.

## Failure map

| Failure type        | Earliest useful check          | Blocks SFE-5 now? | Why                                                                                       | Entry signal if deferred                                                                |
| ------------------- | ------------------------------ | ----------------- | ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Formatting          | Local command and pull request | Yes               | Formatting is deterministic, cheap, and keeps review focused on behavior.                 | Already present through `pnpm format:check`.                                            |
| Linting             | Local command and pull request | Yes               | ESLint catches agreed TypeScript and Angular mistakes before review.                      | Already present through `pnpm lint`.                                                    |
| Type checking       | Local command and pull request | Yes               | Strict TypeScript and Angular template checks protect the application contract.           | Already present through `pnpm typecheck`.                                               |
| Unit behavior       | Local command and pull request | Yes               | Unit tests are the fastest executable signal for introduced application behavior.         | Already present through `pnpm test`; broaden ownership as product workflows appear.     |
| Production bundling | Local command and pull request | Yes               | The build proves the compiled browser artifact can be produced with production settings.  | Already present through `pnpm build`.                                                   |
| Accessibility       | Review now; future automation  | No                | There is not enough delivered workflow surface for a stable automated accessibility gate. | Add automated checks when keyboard-operable user journeys and shared UI patterns exist. |
| End-to-end behavior | Future pull request check      | No                | An E2E suite would be mostly placeholder work before a stable walking skeleton exists.    | Add Playwright or equivalent once a core portfolio journey has stable routes and data.  |
| Security scanning   | Review now; future CI          | No                | The MVP has no backend, secrets, production data, or deployable integration surface yet.  | Add dependency and code scanning when integrations, auth, or deployable services enter. |
| Performance         | Manual observation now         | No                | Performance budgets need real UI and usage targets to avoid arbitrary thresholds.         | Add budgets when a representative workflow and measurement baseline exist.              |

## Current blocking suite

The SFE-5 blocking suite is:

1. `pnpm format:check`
2. `pnpm lint`
3. `pnpm typecheck`
4. `pnpm test`
5. `pnpm build`

Local development and CI should call the same repository scripts so failures name the same check in
both places. Deferred gates should enter only when they have an owner, an actionable failure mode, and
a feedback cost appropriate for the daily development loop.

Initial test ownership belongs to the frontend feature owner for the behavior being changed. Until a
separate platform or quality owner exists, the repository maintainer owns triage for format, lint,
type-check, and build gate failures, while the author of each feature owns adding or updating the
unit tests that protect that feature's behavior.

## Local command contract

The root `package.json` scripts are the local quality interface. Use those scripts instead of global
Angular, Nx, TypeScript, ESLint, Prettier, or Vitest commands.

| Script              | Underlying check    | Failure identity                                                          |
| ------------------- | ------------------- | ------------------------------------------------------------------------- |
| `pnpm format:check` | Prettier repository | The Prettier output names unformatted files.                              |
| `pnpm lint`         | `web:lint`          | Nx output names the `web` project lint target.                            |
| `pnpm typecheck`    | `web:typecheck`     | Nx output names the `web` project type-check target.                      |
| `pnpm test`         | `web:test`          | Nx and Vitest output name the `web` project test target and failing spec. |
| `pnpm build`        | `web:build`         | Nx output names the `web` project production build target.                |
| `pnpm check`        | Full local suite    | The first failed script identifies the failed quality stage.              |

Dependency installation remains deterministic through `pnpm install --frozen-lockfile`, the
committed `pnpm-lock.yaml`, and the `packageManager` value in `package.json`.

## CI command contract

The `Quality` GitHub Actions workflow runs on pull requests and pushes to `main`. It uses these
stable blocking job names:

1. `format`
2. `lint`
3. `typecheck`
4. `test`
5. `build`

Each job checks out the repository with read-only content permissions, installs the Node.js version
from `.node-version`, enables Corepack, restores the pnpm store from a lockfile-keyed cache, installs
dependencies with `pnpm install --frozen-lockfile`, and runs the matching root script.

CI sets `NX_DAEMON=false` and `NX_SKIP_NX_CACHE=true` so the GitHub check proves the command can run
from source on the current commit. The dependency cache is limited to pnpm's content-addressed store;
`node_modules` and Nx build outputs are not cached.

## Local and CI alignment

The local and CI gates share one command contract:

| Stable check | Local command       | CI job      |
| ------------ | ------------------- | ----------- |
| Format       | `pnpm format:check` | `format`    |
| Lint         | `pnpm lint`         | `lint`      |
| Type-check   | `pnpm typecheck`    | `typecheck` |
| Unit tests   | `pnpm test`         | `test`      |
| Build        | `pnpm build`        | `build`     |

Keep CI focused on invoking these scripts. If a future project is added, update the root scripts
first and let CI continue to call the repository interface rather than adding CI-only commands.

## Branch protection checks

The main-branch ruleset should require the `Quality` workflow's stable job names:

1. `format`
2. `lint`
3. `typecheck`
4. `test`
5. `build`

Only these checks are stable enough to block `main` during SFE-5. Do not require end-to-end,
accessibility, security, performance, deployment, signed-commit, merge-queue, or code-scanning gates
until their entry signals are met.

## Future gate roadmap

| Future gate             | Owner                  | Entry signal                                                                     | Expected feedback cost           |
| ----------------------- | ---------------------- | -------------------------------------------------------------------------------- | -------------------------------- |
| Automated accessibility | Frontend feature owner | A keyboard-operable user journey and reusable UI patterns exist.                 | Fast PR check after unit tests.  |
| End-to-end journey      | Walking-skeleton owner | A stable portfolio route, deterministic data, and observable user outcome exist. | Medium PR check.                 |
| API contract            | API boundary owner     | A client/server or generated-contract boundary exists.                           | Fast PR check near type-check.   |
| Dependency scanning     | Repository maintainer  | External dependency changes become routine enough to need automated review.      | Fast scheduled or PR check.      |
| Code security scanning  | Security/privacy owner | Authentication, secrets, backend code, or real integrations enter scope.         | Medium PR or scheduled check.    |
| Performance budgets     | Performance owner      | A representative workflow and measured baseline exist.                           | Medium PR check or release gate. |

Each new gate must name the failure it catches, the person or role that owns triage, and the reason
its runtime belongs in the daily development loop.
