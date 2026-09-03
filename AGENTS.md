# Northline repository instructions

## Product and sources of truth

- Northline is a B2B portfolio-operations and risk-monitoring workspace. The MVP is read-only,
  uses synthetic data, and must not execute trades, move money, give financial advice, or process
  real customer personal data.
- Read `docs/product/overview.md` for product scope, roles, business rules, and vocabulary. Prefer
  **exception case** and **audit event** unless a later approved decision gives another term a
  distinct meaning.
- Notion owns ticket requirements, priority, learning progress, and completion status. GitHub pull
  requests own automated-check and review evidence. Follow `docs/guides/delivery-workflow.md`.
- `README.md` and `package.json` define the supported developer commands. Repository configuration
  is authoritative if prose and executable behavior disagree.
- Record durable product decisions in `docs/product/`, architecture decisions in `docs/decisions/`,
  developer practices in `docs/guides/`, and reproducibility evidence in `docs/verification/`.

## Commands

- Use Node 24 and the pnpm 10 version selected through Corepack from `package.json`.
- Install with `pnpm install --frozen-lockfile`.
- Use repository scripts rather than global Angular, Nx, TypeScript, ESLint, Prettier, or Vitest
  commands:
  - `pnpm serve` — start the development server.
  - `pnpm build` — create the production bundle.
  - `pnpm lint` — run ESLint.
  - `pnpm typecheck` — check application, template, and test TypeScript.
  - `pnpm test` or `pnpm test:watch` — run Vitest once or in watch mode.
  - `pnpm format` or `pnpm format:check` — apply or verify formatting.
  - `pnpm check` — run the complete review gate.

## Architecture and code boundaries

- Keep `apps/web` as the only Nx project until a concrete product or engineering seam justifies
  another application or library. Do not create speculative `shared`, `core`, or `data-access`
  projects.
- Use Angular standalone APIs and strict TypeScript. Preserve the additional checks in
  `tsconfig.base.json`; do not use `any` or disable a rule to hide a design problem.
- Keep application-wide providers in `apps/web/src/app/app.config.ts` and routes in
  `apps/web/src/app/app.routes.ts`.
- Put deliberate document-wide styles in `apps/web/src/styles.scss` and component-owned styles in
  the component stylesheet.
- Do not add a backend, SSR, global state library, component library, microfrontend, real-data
  integration, or another deployable without an approved requirement and, where architectural, an
  ADR. Follow `docs/decisions/0001-use-nx-workspace.md`.
- Add tests for observable behavior and failure states introduced by the ticket. Do not test only
  implementation details.
- Add nested `AGENTS.md` files only when a directory has genuinely different commands or rules;
  avoid restating root guidance.

## Mentor, delegation, and review modes

- In mentor mode, work one ticket step at a time. Explain why the current step matters, what the
  learner should do, useful commands, and verification before implementation.
- Apply user-attempt-first: do not edit files for the current step until the learner shares an
  attempt or explicitly delegates it. Offer only the requested hint level and do not reveal later
  steps prematurely.
- Use this progressive hint ladder:
  - Level 0 — restate the outcome and acceptance criteria only.
  - Level 1 — ask a guiding question or point to a relevant source.
  - Level 2 — explain the governing concept and identify where to investigate.
  - Level 3 — give a targeted hint, relevant command, or API shape without the solution.
  - Level 4 — show pseudocode or a small isolated example without editing repository files.
  - Level 5 — implement the current step after explicit delegation.
- `Delegate implementation for Step N` authorizes implementation of that step only.
  `Delegate the entire ticket` authorizes implementation of all remaining steps. Equivalent language
  must be unambiguous; delegation never bypasses acceptance criteria, verification, or review.
- For an AI-off exercise, Codex may restate the goal before the attempt and review submitted work
  afterward, but must not provide hints, pseudocode, solution commands, code, or file edits during
  the attempt. Resume assistance only after the learner explicitly ends AI-off mode.
- In review or diagnosis mode, inspect evidence and report findings without editing files. Classify
  review findings as **must fix**, **should fix**, or **optional**. Implement fixes only when the
  learner asks for them.
- Follow `docs/guides/mentor-ticket-workflow.md` for ticket state transitions and progress reports.

## Quality and definition of done

- Map the change and its tests to the ticket's acceptance criteria before requesting review.
- Run focused checks while working and `pnpm check` before review. Treat a failing or skipped stage
  as not ready; do not raise lint, test, or bundle thresholds merely to make the gate pass.
- Review the complete diff for unrelated changes, generated files, secrets, personal paths,
  accessibility regressions, privacy/security impact, and speculative abstractions.
- Treat AI-generated output as untrusted until a human can explain the design, inspect the diff, and
  verify it with relevant tests. Record assumptions and unresolved trade-offs rather than presenting
  them as facts.
- Update documentation when behavior, commands, product vocabulary, architecture, or workflow
  changes. Use an ADR for decisions with meaningful alternatives or revisit triggers.
- Keep one ticket per branch and task. Use the ticket ID in branch, commit, and pull-request names;
  do not merge or mark the ticket `Done` before review and explicit approval.

## Dependencies, security, and prohibited shortcuts

- Add or upgrade dependencies only for a current requirement. Use pnpm to update the lockfile and
  explain the need, maintenance cost, and security implications.
- Do not approve new dependency lifecycle scripts without understanding the package and why code
  execution is required.
- Never commit credentials, tokens, real customer data, personal absolute paths, local IDE state,
  generated output, or transient chat content. Use environment-variable names or documented
  external secret storage, never secret values, in configuration.
- Give integrations the minimum tools and permissions needed for a named workflow. Repository-scoped
  MCP remains deferred by `docs/decisions/0002-defer-repository-scoped-mcp.md` until its acceptance
  conditions are met.
- Do not bypass safeguards with `any`, ignored promises, disabled rules, skipped tests, weakened
  compiler options, inflated budgets, force pushes, or direct material changes to protected `main`.
