# Northline

Northline is an Angular 22 application managed with Nx and pnpm.

## Prerequisites

- Node.js 24; the repository version is recorded in `.node-version`.
- Corepack enabled so the `packageManager` field selects pnpm 10.

Install the locked dependency graph:

```bash
corepack enable
pnpm install --frozen-lockfile
```

Use the repository scripts below instead of globally installed Angular, Nx, TypeScript, ESLint, or Prettier commands.

## Development commands

| Command             | Purpose                                                                                           |
| ------------------- | ------------------------------------------------------------------------------------------------- |
| `pnpm serve`        | Run the Angular development server with live reload.                                              |
| `pnpm build`        | Create the optimized production bundle in `dist/apps/web`.                                        |
| `pnpm lint`         | Run the repository ESLint configuration against `web`.                                            |
| `pnpm typecheck`    | Check application TypeScript, Angular templates, and unit-test TypeScript without emitting files. |
| `pnpm test`         | Run the Vitest unit suite once.                                                                   |
| `pnpm test:watch`   | Keep Vitest running during development.                                                           |
| `pnpm format`       | Format supported repository files with Prettier.                                                  |
| `pnpm format:check` | Check formatting without changing files.                                                          |
| `pnpm check`        | Run formatting, lint, type-check, tests, and the production build.                                |

Nx owns the project tasks. To inspect or invoke them directly:

```bash
pnpm exec nx show project web
pnpm exec nx graph
pnpm exec nx run web:typecheck
```

## WebStorm setup

Keep WebStorm settings local; `.idea` is intentionally ignored.

The engineering foundation was verified with WebStorm `2026.2.1` (`WS-262.9437.27`). Later
versions should also work, but repository commands—not IDE state—remain the source of truth.

1. Select a Node.js 24 interpreter that matches `.node-version`.
2. Select pnpm as the package manager and let Corepack use the version declared in `package.json`.
3. Configure TypeScript to use `node_modules/typescript`, not WebStorm's bundled TypeScript.
4. Keep the Angular plugin enabled so templates use the installed `@angular/language-service` package.
5. Run and debug the scripts from `package.json`. For example, run `serve` from the package scripts panel rather than creating a configuration that calls a global `ng` or `nx` binary.
6. Point automatic ESLint and Prettier integration at the packages in this repository. Treat `pnpm check` as the final source of truth before review.

WebStorm may create personal run configurations and on-save actions, but the repository scripts must remain sufficient for another developer and CI to reproduce every check.

## Architecture decisions

- [ADR 0001: Use Nx as the workspace orchestrator](docs/decisions/0001-use-nx-workspace.md)
- [ADR 0002: Defer repository-scoped MCP configuration](docs/decisions/0002-defer-repository-scoped-mcp.md)

## Developer guides

- [AI engineering working agreement](AGENTS.md): repository context, engineering constraints,
  mentor/delegation behavior, review expectations, and security rules for Codex-assisted work.
- [Engineering foundation](docs/guides/engineering-foundation.md): how Angular, Nx, pnpm,
  TypeScript, ESLint, Prettier, testing, builds, and architectural boundaries fit together.
- [Quality strategy](docs/guides/quality-strategy.md): which checks block delivery now and what
  evidence will justify future gates.
- [Mentor ticket workflow](docs/guides/mentor-ticket-workflow.md): how a learner and Codex work
  through a ticket one step at a time.
- [Delivery workflow](docs/guides/delivery-workflow.md): how a Notion ticket stays traceable through
  its Codex task, Git branch, commits, pull request, review, and completion.
