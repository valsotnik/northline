# Ticket workflow

Use one Notion ticket, one Codex chat, one Git branch, and normally one pull request per outcome.

## 1. Start a ticket

1. Open the existing **Northline** project in Codex.
2. Create a **New chat** inside the project using the **Local** environment.
3. Name it `SFE-N — ticket title`.
4. Send:

```text
Start SFE-N.
Mode: mentor.
Available time: 60–90 minutes.
```

Codex should read Notion and Git, confirm dependencies, create `codex/SFE-N-short-title`, and move the ticket to `In Progress` only when work starts.

## 2. Work one step at a time

Start each step with:

```text
Explain Step 1: why it matters, what I should do, useful commands,
and how to verify it. Do not edit files yet.
```

Then choose one path.

### I implement the step

```text
I will implement Step 1. Do not edit files. Wait for my progress.
```

When finished:

```text
Step 1 is done. Evidence: ...
Review it. If it is correct, explain Step 2 without implementing it.
```

### Codex implements the step

```text
I want to skip manual implementation of Step 1.
Explain it first, then delegate implementation for Step 1 only.
Show the changes and verification. Keep later steps in mentor mode.
```

Other useful requests:

```text
Give me the next hint without implementing.
Show pseudocode or a small example only.
Review my plan before I implement it.
Diagnose this error without fixing it: ...
Review my current changes.
```

Hint levels run from 0 (criteria only) to 5 (implementation). Delegation applies only to the named step unless you explicitly delegate the entire ticket.

## 3. Report progress

```text
Ticket: SFE-N
Status: working | blocked | ready for review
Completed: ...
Evidence: tests, command output, commit, or PR
Stuck on: ... | none
Next: ...
Hint level requested: 0–5
Sync Notion: yes | no
```

Useful Git checks:

```bash
git status --short --branch
git branch --show-current
git diff
git log --oneline --decorate -5
```

## 4. Request review

Send:

```text
SFE-N is ready for review.
Sync Notion: yes.
```

Codex checks the acceptance criteria, diff, tests, and evidence, then reports **must fix**, **should fix**, and **optional** feedback. The ticket moves to `In Review`, not directly to `Done`.

## 5. Approve completion

After review feedback is resolved, send:

```text
SFE-N approved.
Sync Notion: yes.
```

Codex integrates the approved branch, records the evidence, marks the ticket `Done`, and makes the next dependency-ready ticket `Ready`.

## Rules

- Do not mix unrelated tickets in one branch or chat.
- Do not reveal every hint before attempting the task.
- Do not mark a ticket `Done` before review.
- Start a new chat for the next ticket; keep using the same project folder.
- Notion holds requirements and progress; Git holds code history; the repository holds durable technical truth.

Reference: [OpenAI — Projects and chats](https://learn.chatgpt.com/docs/projects)
