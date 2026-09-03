# SFE-5 quality baseline verification

- **Ticket:** SFE-5
- **Branch:** `codex/SFE-5-quality-baseline`
- **Date:** 2026-09-03
- **Local environment:** Node `v24.18.0`, Corepack `0.35.0`, pnpm `10.33.0`

## Controlled failure evidence

Each probe introduced one deliberate failure, ran the matching repository script, and then reverted
the deliberate change before moving on.

| Gate       | Probe                                            | Command                                                | Observed rejection                                                                    |
| ---------- | ------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------------------------------- |
| Format     | Misformatted one Markdown list item.             | `pnpm format:check`                                    | Prettier named `docs/guides/quality-strategy.md` and exited `1`.                      |
| Lint       | Added an unused `unusedLintProbe` symbol.        | `NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm lint`      | ESLint reported `@typescript-eslint/no-unused-vars`; Nx named failed task `web:lint`. |
| Type-check | Assigned `'Northline'` to a `number` constant.   | `NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm typecheck` | TypeScript reported `TS2322`; Nx named failed task `web:typecheck`.                   |
| Unit test  | Changed the app-name expectation to `Wrongline`. | `NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm test`      | Vitest named `apps/web/src/app/app.spec.ts`; Nx named failed task `web:test`.         |
| Build      | Lowered the initial bundle budget to `1kb`.      | `NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm build`     | Nx named failed task `web:build:production` and reported `Cache: Skipped`.            |

A direct verbose diagnostic run of the same build target also confirmed the underlying Angular
bundle-budget error. That diagnostic was run outside the sandbox because the sandbox masks uncached
Angular build details, matching the SFE-2 verification record.

## Passing evidence after reverting probes

- `pnpm check` passed after all controlled failures were reverted.
- `pnpm exec nx reset` cleared local cache metadata created by the deliberate failure probes.
- `env NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm check` passed outside the sandbox with every Nx
  target reporting `Cache: Skipped (--skip-nx-cache)`.
- `git diff --check` passed.
- The current workflow is formatted by Prettier.

## Branch protection evidence

GitHub Actions run `33768111815` completed these `Quality` workflow checks successfully on PR #3
commit `5661d2e1c37d10089537d01abf1e0bff403e33e7`:

1. `format`
2. `lint`
3. `typecheck`
4. `test`
5. `build`

The active `Protect main` repository ruleset, updated on 2026-09-03 at 16:31:07 Europe/Warsaw time,
requires those same five status checks from the GitHub Actions integration. The ruleset also keeps
deletion protection, non-fast-forward protection, pull requests before merge, and required review
thread resolution active without bypass actors.
