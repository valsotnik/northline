# ADR 0001: Use Nx as the workspace orchestrator

- **Status:** Accepted
- **Date:** 2026-09-01
- **Scope:** SFE-2 engineering foundation

## Context

Northline currently has one deployable Angular 22 application, `web`. The repository needs one
reproducible toolchain for development and future CI, while leaving room for additional applications
or libraries only when the product creates a real reason for them.

Nx 23 provides a project graph, task graph, task inference, and computation caching. Nx documents
these as independently adoptable capabilities: its historical distinction between “integrated” and
“package-based” repositories is deprecated as of Nx 20. This decision therefore uses
**integrated-style** only as a description of the repository shape required by the ticket, not as a
current Nx repository category.

The chosen shape has one root package manifest, one pnpm lockfile, shared root configuration, and an
Angular application under `apps/web`. Angular supports a workspace containing one or more projects
with shared workspace-level configuration; using that capability does not require creating multiple
projects in advance.

An Nx project is not automatically a well-designed module. A module earns a separate project when it
can hide meaningful implementation behind a small interface at a stable seam. Premature libraries
would add interfaces and configuration without enough implementation to create depth, leverage for
callers, or locality for maintainers.

## Decision

Use Nx 23 as the root workspace and task orchestrator for the Angular 22 application, with pnpm as a
single root dependency and lockfile manager.

Concretely:

- Keep root-owned toolchain configuration in `package.json`, `pnpm-lock.yaml`, `nx.json`,
  `tsconfig.base.json`, and the root ESLint and Prettier files.
- Keep `apps/web` as the only Nx project until evidence justifies another project.
- Treat the root `pnpm` scripts as the developer and CI interface for serving, building, linting,
  type-checking, testing, formatting, and running the complete check.
- Use Nx's project and task graphs, task orchestration, and caching instead of maintaining a second
  task system beside Nx.
- Allow project-local configuration where behavior is genuinely specific to `web`; use shared root
  configuration where a rule applies across the repository.
- Introduce a library or application only at a concrete seam, such as a second deployable
  application, a separately consumed module, or implementation that needs independent ownership or
  verification.
- Keep a future API outside this workspace by default when it is independently deployed or requires
  a different runtime, release cadence, ownership model, or access policy. Add it as an Nx
  application only when sharing this repository and task graph provides measurable coordination
  value and the API can follow the root toolchain lifecycle. Record that choice in a new ADR before
  creating the API project.
- Prefer a deep module with a small interface over several shallow pass-through libraries. Add an
  adapter at a seam when an actual external integration or interchangeable implementation exists,
  rather than to anticipate one.

This is the repository shape historically described as an integrated Nx workspace: applications and
libraries can share a root toolchain and are coordinated through Nx. The decision is about those
selected features, not about preserving deprecated Nx terminology.

## Alternatives considered

### Angular CLI workspace without Nx

This would minimize initial orchestration configuration for a single application. It was rejected
because the project intentionally wants one extensible task interface, graph-aware orchestration,
caching, and enforceable project relationships as it grows. Keeping Nx now also avoids a later
toolchain migration when additional justified projects appear.

### Package-per-project workspace

Each application or library could own a `package.json`, dependency set, and package-oriented
interface. This is useful when projects are independently published, released, or require materially
different toolchains. Northline has none of that evidence today, so the extra manifests and dependency
coordination would reduce locality without providing useful independence.

### Multiple Nx libraries from the start

The repository could begin with `shared`, `core`, `data-access`, or feature libraries. This was
rejected because no stable interfaces, separate consumers, ownership split, or interchangeable
adapters exist yet. Such projects would create speculative seams and shallow modules. The current
application keeps related change and verification local while the domain model is still forming.

## Consequences

### Positive

- Developers and future CI use one root command and dependency interface.
- Nx can analyze projects and tasks, order work, and reuse cached results.
- Angular, TypeScript, ESLint, Prettier, and test versions remain aligned through one lockfile.
- The repository can add an application or library later without replacing its task orchestrator.
- Keeping one project today preserves locality and avoids pretending that unproven seams already
  exist.

### Negative and accepted trade-offs

- Nx adds configuration and concepts beyond a plain single-application Angular CLI workspace.
- A shared root toolchain couples projects to coordinated dependency upgrades if more projects are
  added.
- The single `web` project does not yet provide project-graph-enforced internal architecture; code
  organization and Angular/TypeScript interfaces must carry that responsibility until a project seam
  is justified.
- Extracting a module later may require moving code and tightening its interface after real usage is
  known. That cost is accepted in exchange for avoiding speculative structure now.

## Evidence in the repository

The decision is currently represented by:

- `pnpm exec nx show projects` returning only `web`;
- `apps/web/project.json` defining the Angular application and its explicit project tasks;
- `nx.json` providing shared task defaults and ESLint task inference;
- root `package.json` scripts forming the stable command interface;
- one root `pnpm-lock.yaml` and `pnpm-workspace.yaml` controlling dependency installation; and
- `pnpm check` exercising formatting, linting, Angular and TypeScript checks, unit tests, and the
  production build through that interface.

These are observable checks, not a promise that one project will remain correct forever.

## Revisit triggers

Revisit this decision when concrete evidence shows one or more of the following:

- A second independently deployable application is approved.
- A module has at least two real consumers and a stable interface whose extraction improves leverage
  and locality.
- Two concrete adapters must satisfy the same interface, making the seam real rather than
  hypothetical.
- A project needs independent publishing, release cadence, ownership, access policy, or a materially
  different dependency/toolchain lifecycle.
- Nx project-graph or task timing evidence shows that splitting a project would materially improve
  affected-task execution or cache effectiveness.
- Coordinated root dependency upgrades create measured delivery friction that outweighs one-toolchain
  consistency.

A trigger starts a new architectural decision; it does not automatically require a split.

## References

- [Nx: Integrated vs. package-based repositories (deprecated terminology)](https://nx.dev/docs/reference/deprecated/integrated-vs-package-based)
- [Nx mental model: project graph, inferred tasks, task graph, and caching](https://nx.dev/docs/concepts/mental-model)
- [Nx: Managing configuration files](https://nx.dev/docs/concepts/types-of-configuration)
- [Angular: Workspace and project file structure](https://angular.dev/reference/configs/file-structure)
