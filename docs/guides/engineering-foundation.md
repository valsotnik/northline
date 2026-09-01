# Northline engineering foundation

This guide explains the infrastructure established by SFE-2 and the conventions every Northline
developer should understand. The repository configuration remains the source of truth when this
guide and executable behavior disagree.

## Mental model

The toolchain has four layers with different responsibilities:

| Layer         | Responsibility                                                                           |
| ------------- | ---------------------------------------------------------------------------------------- |
| Angular       | Application runtime, standalone components, templates, routing, and dependency injection |
| Nx            | Project and task graphs, task execution, dependency ordering, and computation caching    |
| pnpm          | Dependency installation and lockfile management                                          |
| Quality tools | TypeScript, Angular compiler, ESLint, Prettier, and Vitest validation                    |

The root `package.json` scripts are the stable developer and future CI interface. Prefer those
scripts over globally installed `ng`, `nx`, TypeScript, ESLint, or Prettier commands.

## Toolchain contract

The project supports Node.js `>=24.15.0 <25`; `.node-version` records the verified version
`24.18.0`. The `packageManager` field selects pnpm `10.33.0` through Corepack, while the pnpm engine
range allows compatible pnpm 10 installations.

Important framework and tool versions are:

- Angular 22
- Nx `23.1.2`
- TypeScript 6
- Vitest 4

After cloning the repository, install the committed dependency graph:

```bash
corepack enable
pnpm install --frozen-lockfile
```

`--frozen-lockfile` rejects disagreement between `package.json` and `pnpm-lock.yaml` instead of
silently changing dependency resolution. Do not edit `pnpm-lock.yaml` manually; use pnpm when adding,
removing, or upgrading dependencies.

The pnpm workspace allows dependency installation scripts only for `esbuild` and `nx`. Do not approve
another package's lifecycle scripts without understanding why execution is required and whether the
package is trusted.

## Repository shape

The workspace intentionally contains one Nx project:

```text
Northline
├── apps/
│   └── web/              Angular application
├── docs/
│   ├── decisions/        Architecture decision records
│   ├── guides/           Durable developer guidance
│   ├── product/          Product documentation
│   └── verification/     Reproducibility evidence
├── package.json          Developer commands and dependency declarations
├── nx.json               Workspace-level Nx behavior
├── tsconfig.base.json    Shared TypeScript rules
└── pnpm-lock.yaml        Exact dependency graph
```

Inspect the registered projects with:

```bash
pnpm exec nx show projects
```

The expected result is currently `web`. Do not create speculative `shared`, `core`, or `data-access`
libraries simply to categorize folders.

## Daily command interface

| Command             | Purpose                                                                             |
| ------------------- | ----------------------------------------------------------------------------------- |
| `pnpm serve`        | Start the Angular development server with live reload                               |
| `pnpm build`        | Produce an optimized production build under `dist/apps/web`                         |
| `pnpm lint`         | Run ESLint against application TypeScript and Angular templates                     |
| `pnpm typecheck`    | Check application TypeScript, templates, and test TypeScript without emitting files |
| `pnpm test`         | Run the Vitest suite once                                                           |
| `pnpm test:watch`   | Run Vitest continuously during development                                          |
| `pnpm format`       | Apply Prettier formatting                                                           |
| `pnpm format:check` | Check formatting without changing files                                             |
| `pnpm check`        | Run formatting, linting, type-checking, tests, and the production build in sequence |

Run `pnpm check` before requesting review. A failure at any stage means the change is not ready.

When diagnosing cache-sensitive behavior, reset Nx and run the gate uncached:

```bash
pnpm exec nx reset
NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm check
```

## Nx responsibilities

`nx.json` owns workspace defaults and `apps/web/project.json` owns the application's targets. Nx
provides:

- a project graph describing project dependencies;
- a task graph describing execution order;
- a common interface for serving, building, testing, linting, and type-checking; and
- computation caching when relevant inputs have not changed.

Production builds and unit tests are cacheable. The build target uses `dependsOn: ["^build"]`, so
future library dependencies would build before their consumer. Cache use improves speed but should be
disabled when explicitly proving an uncached verification result.

## Angular application

Northline uses standalone Angular bootstrapping. `apps/web/src/main.ts` calls
`bootstrapApplication(App, appConfig)`; there is no root `AppModule`.

Application-wide providers live in `apps/web/src/app/app.config.ts`. The current configuration adds
browser error listeners and the Angular router. Routes live in `apps/web/src/app/app.routes.ts`; the
route array remains empty until product features introduce navigation.

This foundation is client-side rendered. SFE-2 deliberately does not add SSR, prerendering, a backend,
authentication, state-management libraries, end-to-end tests, CI/CD, or feature libraries. Later
tickets should introduce those only with explicit requirements.

## SCSS

SCSS is the default style format for future Angular generators.

- `apps/web/src/styles.scss` is for deliberate global concerns such as resets, typography, theme
  tokens, and document-level behavior.
- Component `.scss` files are for styles owned by one component.

The global stylesheet is intentionally empty. Do not add a speculative design system or broad global
rules before the UI requirements exist.

## TypeScript safety

`tsconfig.base.json` enables strict TypeScript and additional correctness checks:

- `strict` enables the main strict family of checks.
- `exactOptionalPropertyTypes` distinguishes an absent property from explicitly assigning
  `undefined`.
- `noUncheckedIndexedAccess` makes an indexed result potentially `undefined`.
- `noImplicitReturns` requires all relevant code paths to return.
- `noImplicitOverride` requires an explicit `override` modifier.
- `noFallthroughCasesInSwitch` prevents accidental switch fallthrough.
- `noPropertyAccessFromIndexSignature` makes dictionary access explicit.
- `noUncheckedSideEffectImports` detects missing side-effect imports.
- `verbatimModuleSyntax` preserves explicit type/value import intent.
- `forceConsistentCasingInFileNames` prevents cross-platform filename casing failures.

For example, accessing `names[0]` produces `string | undefined`, so the missing case must be handled.
This is intentional: possible runtime failures should become development-time feedback.

`pnpm typecheck` runs two checks. Angular's `ngc` validates application TypeScript and templates;
TypeScript separately validates test files with Vitest types. Plain `tsc` cannot provide complete
Angular template validation.

## ESLint and Prettier

ESLint owns correctness and suspicious-code rules. Important enforced behavior includes:

- no explicit `any`;
- no unused variables;
- explicit type-only imports;
- no ignored, misused, or incorrectly awaited promises;
- strict equality, braces, and `const` where applicable;
- `app-` component and `app` directive selector conventions;
- modern Angular signals and output APIs where supported;
- no `any` or duplicate attributes in templates; and
- explicit button types in templates.

Nx module-boundary enforcement is already enabled. It becomes more significant when justified
libraries or applications are introduced.

Prettier owns presentation: single quotes, two-space indentation, semicolons, trailing commas,
100-character print width, LF endings, and Angular-aware HTML formatting. `.editorconfig` gives
editors compatible basic behavior, but Prettier remains the final formatting authority.

Generated state—including `node_modules`, `dist`, `.nx`, `.angular`, coverage, and local IDE files—is
ignored by Git.

## Testing and production build

Unit tests use Vitest through Angular's unit-test builder. The initial test proves that the root
component can be created; feature work should add behavioral tests for real requirements rather than
testing internal implementation details alone.

The production build enforces bundle budgets:

- initial bundle warning at 500 kB and error at 1 MB;
- individual component stylesheet warning at 4 kB and error at 8 kB.

A budget failure is a signal to investigate application growth, not a threshold to increase without
evidence. SFE-2 verified the production build at 187.54 kB of initial JavaScript and an estimated
51.13 kB transfer size.

## Generating Angular code

Use repository-local Nx/Angular generators so generated files follow workspace conventions:

```bash
pnpm exec nx generate @nx/angular:component feature-name --project=web
```

Inspect unfamiliar generator options first:

```bash
pnpm exec nx generate @nx/angular:component --help
```

After generation, run:

```bash
pnpm format
pnpm check
```

A generator supplies a consistent starting point; the developer must still verify location,
interface design, accessibility, behavior, and tests.

## Architectural boundary rule

Create another Nx project only when there is evidence of a real boundary. Useful evidence includes:

- at least two genuine consumers and a stable interface;
- independent ownership, publishing, release, or verification;
- interchangeable implementations behind one meaningful seam; or
- measured project-graph or cache improvements.

A future API should normally remain outside this workspace when it has a different runtime,
deployment, release cadence, ownership model, or access policy. Add it as an Nx application only when
sharing this repository and task graph provides concrete coordination value. Record that decision in
a new ADR before creating the API project.

See [ADR 0001](../decisions/0001-use-nx-workspace.md) for the complete decision and revisit triggers.

## What every developer should be able to explain

1. Angular builds and runs the application; Nx orchestrates projects and tasks; pnpm manages the
   dependency graph.
2. `pnpm-lock.yaml` and `pnpm install --frozen-lockfile` make installation reproducible.
3. `pnpm check` is the complete pre-review quality gate.
4. Angular's `ngc` is necessary because templates need Angular-aware type-checking.
5. ESLint checks correctness; Prettier checks presentation.
6. Strict TypeScript deliberately requires handling states that may be missing or invalid.
7. Nx caching improves speed, while uncached runs provide stronger verification evidence.
8. The repository has one application because project boundaries should follow real seams.
9. Generated files and personal IDE state do not belong in Git.
10. Infrastructure should grow from concrete product and engineering needs, not anticipated folder
    categories.

## Related documentation

- [Repository quick start](../../README.md)
- [ADR 0001: Use Nx as the workspace orchestrator](../decisions/0001-use-nx-workspace.md)
- [SFE-2 reproducibility evidence](../verification/SFE-2-step-7-reproducibility-2026-09-01.md)
