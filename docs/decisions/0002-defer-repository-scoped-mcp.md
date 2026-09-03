# ADR 0002: Defer repository-scoped MCP configuration

- **Status:** Accepted
- **Date:** 2026-09-03
- **Scope:** SFE-4 AI engineering working agreement

## Context

Codex can load MCP servers from a trusted project's `.codex/config.toml`. A shared configuration can
make a repeatable external-tool workflow easier to discover, but it also expands the tools, data,
network endpoints, and authentication assumptions available during repository work.

Northline currently uses two external workflows around the repository:

| Workflow              | Required capability                                                                        | Owner                 | Authentication and secrets                                   | Smoke test                                                                                         | Current path                             |
| --------------------- | ------------------------------------------------------------------------------------------ | --------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| Ticket traceability   | Read an approved Notion ticket; update its status, branch, pull-request link, and progress | Project maintainer    | User/workspace OAuth outside Git; no token in the repository | Fetch the active ticket and confirm its ID/status before work; verify changed properties afterward | Authenticated Notion plugin or connector |
| Pull-request delivery | Push a ticket branch; create, inspect, and review its GitHub pull request                  | Repository maintainer | User OAuth or GitHub CLI credential store outside Git        | Confirm repository identity and authentication, then inspect the created PR and its checks         | Git plus authenticated GitHub tooling    |

Neither workflow currently requires a repository-launched MCP server. Their authentication is
contributor-specific, and committing a server definition would not make access portable without
also introducing account setup and broader tool-policy questions.

## Decision

Do not add `.codex/config.toml` or another repository-scoped MCP server configuration now. Continue
to use authenticated user/workspace integrations for Notion and GitHub while the repository keeps
the workflow contract in `docs/guides/delivery-workflow.md`.

A future MCP proposal must name all of the following before configuration is committed:

1. The concrete workflow and why repository scope improves it over the existing UI, connector, or
   CLI path.
2. The maintainer who owns the server, its upgrades, and access review.
3. The smallest required server tools, data scope, network access, and approval policy.
4. Secret storage outside Git, using environment-variable references or OAuth rather than embedded
   credentials.
5. A reproducible read smoke test and, if writes are required, a narrowly scoped reversible write
   test.
6. Failure and removal behavior so repository work remains possible when the server is unavailable.

If these conditions are met, configure only the required tools, keep write operations approval-gated
where practical, verify the configuration from a fresh trusted checkout, and record the decision in
a new or superseding ADR.

## Consequences

### Positive

- The repository contains no MCP endpoint, command, token, or contributor-specific path without a
  demonstrated workflow need.
- Notion and GitHub access remain available through their existing authenticated paths.
- Future integrations have explicit ownership, least-privilege, secret-storage, and verification
  criteria.

### Negative and accepted trade-offs

- Contributors must configure or authenticate their own Notion and GitHub tooling.
- External-tool discovery is documented rather than automatically provisioned by the repository.
- A useful future MCP workflow will require another deliberate configuration decision.

## Revisit triggers

Revisit when at least two contributors need the same MCP-backed workflow, manual setup repeatedly
causes delivery errors, or a required developer workflow cannot be reproduced with the current
repository commands and authenticated integrations.

## References

- [OpenAI: Model Context Protocol](https://learn.chatgpt.com/docs/extend/mcp)
- [Northline delivery workflow](../guides/delivery-workflow.md)
