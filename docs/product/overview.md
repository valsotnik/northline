# Northline product charter

## Purpose

Northline is a B2B portfolio-operations and risk-monitoring workspace for investment and wealth-management teams. It combines portfolio context, notable changes, and operational exceptions so professionals can decide what needs attention and record the outcome.

The initial product is deliberately read-only and uses synthetic data. It does not execute trades, hold money, give financial advice, or process real customer personal data.

## Problem and value

Portfolio facts, market context, and operational exceptions are often distributed across dense screens and disconnected tools. Analysts reconstruct the situation before acting, while operations and compliance teams struggle to see why a decision was made and whether follow-up is complete.

Northline provides a shared view of portfolio state and a consistent path from noticing a concern to recording its outcome. The intended value is faster understanding, fewer lost exceptions, and a trustworthy history.

## Users and responsibilities

| User | Job to be done | May do in the MVP | Must not do in the MVP |
| --- | --- | --- | --- |
| Portfolio analyst or adviser | Understand portfolio value, movement, and concentration; identify where investigation is needed | View assigned portfolios and positions, inspect exposure, maintain a personal watchlist, and raise or comment on an exception | Change access, modify source portfolio records, or place an order |
| Operations or compliance specialist | Investigate data or policy exceptions and make the outcome traceable | View relevant evidence, assign an exception, add a decision note, resolve it, or defer it with a reason | Alter the underlying synthetic transaction history or grant workspace access |
| Workspace administrator | Keep membership, responsibilities, and reference configuration accurate | Invite or deactivate members, assign roles, and manage workspace-level operational settings | Resolve an exception on behalf of its accountable specialist solely because they are an administrator |

Access is scoped to a workspace. A person may hold multiple responsibilities, but every action remains attributable to that person and permitted by their active role.

## Shared vocabulary

| Term | Meaning and identity | Important rule |
| --- | --- | --- |
| Organization | The business customer that owns workspaces | Information is never shared across organizations |
| Workspace | The collaboration and access boundary for a team | Every portfolio, member, and exception belongs to exactly one workspace |
| Portfolio | Accounts and positions monitored as one business subject | The MVP never changes its source holdings |
| Account | A custody or reporting subdivision within a portfolio | An account belongs to one portfolio |
| Position | A holding of one instrument within an account | Quantity, value, and currency are interpreted together |
| Instrument | A financial product identified independently of any portfolio | It is distinct from a position in that instrument |
| Market quote | A timestamped synthetic price observation for an instrument | A displayed value must communicate when its supporting quote was observed |
| Exposure | Value grouped by a dimension such as instrument or currency | A total states its grouping and valuation currency |
| Watchlist | A user's saved collection of instruments or portfolios to monitor | A watchlist does not change the underlying portfolio |
| Alert rule | A persistent condition a user wants Northline to notice | An event records an occurrence; it is not the rule |
| Alert event | A timestamped occurrence produced when an alert condition is met | Acknowledging an event does not change portfolio data |
| Exception case | Traceable work created for an investigation | It has an accountable status and requires an outcome note for resolution |
| Audit event | An immutable record of who acted, when, and in which workspace | Corrections add an event rather than rewriting history |

Prefer **exception case** and **audit event**. Avoid issue, incident, task, activity, and log unless a later ticket gives them distinct meanings.

## MVP workflow: investigate a portfolio exception

1. **Start:** an analyst selects an assigned portfolio and sees valuation, daily movement, largest exposures, freshness, and unresolved exceptions.
2. **Decide where to look:** the analyst inspects a notable exposure or movement. If information is unavailable or stale, Northline explains the limitation and offers a retry without losing the selection.
3. **Escalate or stop:** if no investigation is needed, the analyst returns to monitoring. Otherwise, the analyst opens an exception case with a concise reason and the relevant portfolio context.
4. **Investigate:** an operations or compliance specialist reviews the case, assigns responsibility, records evidence, and chooses whether to resolve or defer it.
5. **Complete:** resolution requires an outcome note. Deferral requires a reason and a next-review condition. The analyst can see the current result and the complete visible history.

The outcome is either a documented decision that no escalation was needed or a traceable exception with an accountable status and next action.

## MVP boundary

### Must have

- Synthetic, deterministic portfolio information only.
- Read-only portfolio summary, position exploration, and exposure context.
- One personal watchlist and alert workflow.
- One exception-case workflow with assignment, notes, resolution or deferral, and visible history.
- Meaningful loading, empty, failure, stale-data, and unauthorized experiences.
- Keyboard-operable delivered workflows and information that does not rely on color alone.

### Later, when evidence justifies it

- Richer administration, alert types, collaboration, reporting, live data, advanced visualization, and audit reporting.

### Explicitly not building in the MVP

- Real or simulated order entry, deposits, withdrawals, or custody.
- Financial recommendations or automated investment decisions.
- Real customer personal data or connections to production brokerage systems.
- Social feeds, native mobile applications, or advanced financial charting.

## Constraints and assumptions

- Desktop web use is the first product context, with responsive behavior for narrower screens.
- Delivered workflows target WCAG 2.2 AA; usability claims require keyboard and assistive-technology checks.
- Values, dates, freshness, and status remain understandable without hidden institutional knowledge.
- Synthetic data is sufficient to validate workflow comprehension, accessibility, and engineering quality, but not investment outcomes or real-market accuracy.
- Performance and reliability statements require measurements.
- The first learning goal is whether the workflow creates clear decision context, not whether Northline can reproduce a trading platform.

## Outcome signals

1. **Time to understanding:** a first-time target user identifies the largest exposure, valuation currency, and data freshness within two minutes without assistance.
2. **Traceability quality:** 100% of resolved or deferred exceptions in evaluation have an accountable person, an outcome or deferral reason, and complete visible history.
3. **Workflow comprehension:** at least four of five representative attempts reach the correct outcome without facilitator help; repeated confusion produces a learning note.

These are learning metrics for controlled use of synthetic data. They are not claims about financial performance or production efficiency.

## Open product decision: simulated order entry

**Option A — remain read-only:** focuses on understanding and traceability, avoids implying execution or advice, and tests core value sooner. It does not exercise transaction-entry workflows.

**Option B — add simulated order entry:** provides richer validation, authorization, and state transitions, but creates a paper-trading experience before evidence connects it to the main problem.

**Recommendation:** keep the MVP read-only. Revisit after users complete the monitoring-and-exception workflow reliably and three independent evaluation sessions show that modelling a hypothetical transaction is necessary to finish an investigation. The first candidate should be clearly labelled scenario planning, not order-entry imitation.
