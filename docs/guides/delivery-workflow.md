# Delivery workflow

Northline uses one stable ticket identifier to connect product intent in Notion with implementation
and review evidence in GitHub. The delivery chain is:

```text
Notion ticket
→ Codex task
→ Git branch
→ commits
→ GitHub pull request
→ checks and review
→ merge
→ Notion completion
```

## Sources of truth

- Notion is the source of truth for requirements, priority, learning progress, and completion status.
- The GitHub pull request is the source of truth for automated-check and code-review evidence.
- Do not create a GitHub issue merely to duplicate a Notion ticket.

The `SFE-N` ticket identifier carries the relationship across both systems. Store the exact branch
name in the Notion ticket's `Branch` value and the exact pull request URL in its `Pull Request` value
so the implementation and review remain directly discoverable from the requirement.

## Identity conventions

| Artifact   | Convention                       |
| ---------- | -------------------------------- |
| Codex task | `SFE-N — ticket title`           |
| Git branch | `codex/SFE-N-short-kebab-case`   |
| Commit     | `<type>(SFE-N): concise outcome` |
| PR title   | `SFE-N: concise outcome`         |

For now, `codex/` is the only branch namespace. Use one of these commit types to describe the kind
of change: `feat`, `fix`, `docs`, `test`, `refactor`, `chore`, or `ci`.

Each branch belongs to one Notion ticket, starts from the current `main`, and remains short-lived:
merge it after review and delete it after the outcome is integrated. Keep every commit focused on
that ticket, use the message convention above, and exclude credentials, generated output, and
unrelated changes. Record verification evidence in the pull request rather than duplicating it in
commit messages.

## Main branch policy

Protect `main` with an active repository ruleset named `Protect main`. Target the repository's
default branch and do not add a bypass actor.

Enable these rules now:

- Restrict deletions.
- Require a pull request before merging, with zero required approvals.
- Require conversation resolution before merging.
- Block force pushes.

Zero approvals keeps the pull request as auditable delivery evidence without requiring a second
developer to approve a one-developer project. Conversation resolution still prevents unresolved
review findings from being silently merged.

Do not require status checks until SFE-5 creates stable CI check names. Also leave signed commits,
linear history, merge queues, deployments, and code-scanning gates disabled until the project has a
demonstrated need for them.

The initial push of `main` at `3c1e7dc` predates the ruleset and is the one-time repository bootstrap
exception. After the ruleset becomes active, every material change to `main` must use a pull request.
The live GitHub ruleset
[`Protect main`](https://github.com/valsotnik/northline/rules/22100317) is the current verification
source for its enforcement state.

## Completion contract

1. Begin with the approved Notion ticket and use its `SFE-N` identifier for the Codex task.
2. Create the Git branch and commits using the identity conventions above.
3. Open a GitHub pull request whose title carries the same ticket identifier.
4. Record the exact branch name and pull request URL in the Notion ticket.
5. Complete the required checks and review in the pull request, then merge it.
6. Mark the Notion ticket `Done` only after review and merge are complete.

## Ceremony review

The workflow retains controls that provide direct traceability or risk reduction:

- one stable `SFE-N` mapping across the ticket, task, branch, commits, and pull request;
- exact Notion `Branch` and `Pull Request` links;
- focused commits and an evidence-oriented pull request;
- a protected `main` branch; and
- resolved review conversations before merge.

The workflow deliberately omits or defers ceremony that does not yet justify its cost:

- duplicate GitHub issues for requirements already owned by Notion;
- a required approval for a solo developer;
- required status checks until SFE-5 establishes stable check names;
- unconditional screenshots or manual checks when a change has no relevant visible or manual
  behavior; and
- signed commits, linear-history enforcement, merge queues, deployment gates, and code-scanning
  gates without a demonstrated need.

## SFE-3 traceability evidence

- Notion ticket:
  [SFE-3 — Establish Notion-to-GitHub delivery traceability and PR workflow](https://app.notion.com/p/3ce45285941b8150a967f3989b8801e5)
- Branch: `codex/SFE-3-delivery-workflow`
- Initial implementation commits before review:
  - `174b48a` — `docs(SFE-3): restore ticket workflow guide`
  - `a3bdc07` — `docs(SFE-3): document delivery workflow`
  - `326c818` — `chore(SFE-3): add pull request template`
- Review follow-up commits: the
  [PR #1 commit history](https://github.com/valsotnik/northline/pull/1/commits) is authoritative and
  includes the commit containing this review record.
- Pull request: [#1](https://github.com/valsotnik/northline/pull/1)
- Automated gate: `pnpm check` — passed (formatting, linting, type-checking, one unit test, and the
  production build)
- Branch protection evidence:
  [`Protect main`](https://github.com/valsotnik/northline/rules/22100317)
- Review disposition: the Step 6 review retained the evidence-bearing controls and resolved the
  guidance-placement and low-evidence ceremony findings in a focused follow-up.
- Merge state: pull request #1's GitHub state is authoritative. Merge remains intentionally pending
  until the user approves completion.
