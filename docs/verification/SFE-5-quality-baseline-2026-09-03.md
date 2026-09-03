# SFE-5 quality baseline verification

- **Ticket:** SFE-5
- **Branch:** `codex/SFE-5-quality-baseline`
- **Date:** 2026-09-03
- **Local environment:** Node `v24.18.0`, Corepack `0.35.0`, pnpm `10.33.0`

## Controlled failure evidence

Each probe introduced one deliberate failure, ran the matching repository script, and then reverted
the deliberate change before moving on.

| Gate       | Probe                                            | Command                                                | Observed rejection                                                                     |
| ---------- | ------------------------------------------------ | ------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| Format     | Misformatted one Markdown list item.             | `pnpm format:check`                                    | Prettier named `docs/guides/quality-strategy.md` and exited `1`.                       |
| Lint       | Added an unused `unusedLintProbe` symbol.        | `NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm lint`      | ESLint reported `@typescript-eslint/no-unused-vars`; Nx named failed task `web:lint`.  |
| Type-check | Assigned `'Northline'` to a `number` constant.   | `NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm typecheck` | TypeScript reported `TS2322`; Nx named failed task `web:typecheck`.                    |
| Unit test  | Changed the app-name expectation to `Wrongline`. | `NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm test`      | Vitest named `apps/web/src/app/app.spec.ts`; Nx named failed task `web:test`.          |
| Build      | Lowered the initial bundle budget to `1kb`.      | `pnpm exec nx run web:build --skip-nx-cache --verbose` | Angular reported the bundle-budget error; Nx named failed task `web:build:production`. |

The build probe was run outside the sandbox because the sandbox masks uncached Angular build output.
That limitation matches the SFE-2 verification record.

## Passing evidence after reverting probes

- `pnpm check` passed after all controlled failures were reverted.
- `pnpm exec nx reset` cleared local cache metadata created by the deliberate failure probes.
- `env NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm check` passed outside the sandbox with every Nx
  target reporting `Cache: Skipped (--skip-nx-cache)`.
- `git diff --check` passed.
- The current workflow is formatted by Prettier.

## Branch protection evidence

The stable required check names for the GitHub main-branch ruleset are:

1. `format`
2. `lint`
3. `typecheck`
4. `test`
5. `build`

The repository currently has no local `gh` CLI, and this Codex task has no safe non-interactive
GitHub ruleset administration path. Configure these checks in the existing `Protect main` ruleset
after the `Quality` workflow has run at least once on the pull request, then link the GitHub ruleset
or PR evidence from the ticket before marking SFE-5 complete.
