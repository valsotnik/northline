# SFE-4 AI working agreement verification

- **Date:** 2026-09-03
- **Branch:** `codex/SFE-4-ai-working-agreement`
- **Scope:** Root Codex instructions, learning contract, scoped-instruction decision, and
  repository-scoped MCP decision

## Durable-context inventory

| Category                                                      | Existing source                                        | Instruction treatment                                                                                   |
| ------------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| Product purpose and safety boundary                           | `docs/product/overview.md`                             | State the read-only synthetic-data boundary directly; link the detailed charter and vocabulary          |
| Supported commands and toolchain                              | `README.md`, `package.json`                            | State the stable pnpm command interface; keep executable configuration authoritative                    |
| Delivery sources of truth                                     | `docs/guides/delivery-workflow.md`                     | State the Notion/GitHub ownership split and link the complete workflow                                  |
| Architecture boundaries                                       | `docs/guides/engineering-foundation.md`, ADR 0001      | State the constraints most likely to prevent scope drift; link the detailed reasoning                   |
| Learning behavior                                             | `docs/guides/mentor-ticket-workflow.md`                | State user-attempt-first, hint, delegation, AI-off, review, and diagnosis rules directly                |
| Quality and review                                            | `package.json`, delivery and mentor guides             | State the complete gate and human-review expectations directly                                          |
| Dependencies and security                                     | `pnpm-workspace.yaml`, `.gitignore`, engineering guide | State dependency restraint, lifecycle-script review, secret exclusion, and prohibited bypasses directly |
| Ticket detail, personal state, generated output, chat history | Not durable                                            | Exclude from repository instructions                                                                    |

The inventory produced one root `AGENTS.md`. No nested instruction file is justified because the
only Nx project, `apps/web`, uses the same command, review, and learning workflow as the repository.

## Fresh-task smoke test

A fresh ephemeral Codex CLI run started from the repository root with its own read-only sandbox. The
prompt named the categories to report but supplied no Northline facts and prohibited commands,
additional file reads, and edits:

```bash
codex exec --ephemeral --sandbox read-only --cd . \
  "Perform a read-only fresh-task smoke test. Without running commands, reading additional files, or editing anything, report only what the repository instructions already loaded tell you about: (1) product purpose and safety boundary, (2) supported commands, (3) source-of-truth locations, (4) architecture constraints, (5) mentor, delegation, review, and AI-off behavior, and (6) verification before review. Say MISSING for any category not established by the loaded instructions."
```

| Discovery category                  | Result                                                                                                                                                        |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Product purpose and safety boundary | Passed: identified the B2B portfolio-operations purpose and read-only, synthetic-data, no-trading, no-advice, no-real-PII boundary                            |
| Supported commands                  | Passed: identified Node/pnpm expectations and all repository scripts                                                                                          |
| Sources of truth                    | Passed: identified product docs, Notion, GitHub pull requests, command configuration, and durable documentation locations                                     |
| Architecture constraints            | Passed: identified the one-project boundary, standalone Angular, strict TypeScript, provider/route/style locations, and prohibited speculative infrastructure |
| Learning modes                      | Passed: distinguished mentor, per-step and whole-ticket delegation, review/diagnosis, hint levels, and AI-off behavior                                        |
| Review verification                 | Passed: identified acceptance-criteria mapping, focused checks, `pnpm check`, diff review, and documentation updates                                          |

No category was reported as missing. The initial host-sandbox attempt could not initialize Codex's
user-level state database. The successful rerun allowed normal host state access while preserving
the fresh child task's read-only repository sandbox.

## MCP decision verification

- No `.codex/config.toml` or other repository-scoped MCP configuration was added.
- ADR 0002 records the current Notion and GitHub workflows, owners, authentication location, smoke
  tests, least-privilege acceptance conditions, failure behavior, and revisit triggers.
- Credential and personal-path patterns were scanned across the changed documents with no matches.

## Repository verification

The complete uncached quality gate passed:

```bash
NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm check
```

- Prettier: passed.
- ESLint: passed.
- Angular/template and test TypeScript checks: passed.
- Vitest: 1 file and 1 test passed.
- Production build: passed; 187.54 kB raw initial JavaScript and 51.13 kB estimated transfer size.

Angular's production build first failed without diagnostics inside the filesystem/process sandbox,
matching the limitation recorded by SFE-2. The same uncached command passed outside that restriction;
Nx labelled the successful build flaky because it observed the preceding sandbox-induced failure.

Additional checks confirmed that `web` remains the only Nx project, `git diff --check` passes, the
root `AGENTS.md` is the only repository instruction file, and project MCP configuration is absent as
decided.
