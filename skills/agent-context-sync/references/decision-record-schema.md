# Decision Record Schema

Use this reference when recording, updating, superseding, or reviewing decisions.

## When To Create A Decision

Create a decision record for important choices about:

- product positioning
- V1 scope
- architecture
- safety or privacy boundaries
- automation boundaries
- workflow policy
- accepting or rejecting major alternatives
- adding or removing core file contracts

Do not create a decision record for formatting edits, ordinary notes, minor wording changes, or brainstorming that has not converged.

## Filename

Use:

```text
.agent-context/decisions/DEC-YYYY-MM-DD-NNN-short-title.md
```

Default numbering is date-local monotonic:

1. List existing `DEC-YYYY-MM-DD-*.md` files for the same date.
2. Pick the next `NNN`.
3. Use a short lowercase hyphenated title.

## Status Values

- `proposed`: AI recommendation, candidate decision, or unconfirmed direction.
- `accepted`: user clearly confirms or committed project docs already establish the decision.
- `superseded`: replaced by a newer decision.
- `rejected`: explicitly considered and rejected as the decision itself.

## Required Fields

Every record must include:

- title
- status
- date
- confirmed by
- related sessions
- related files
- supersedes
- superseded by
- context
- decision
- reasons
- rejected alternatives
- evidence
- consequences
- review triggers

## Accepted Decision Rules

Only mark `accepted` when:

- the user explicitly says the decision is made, accepted, confirmed, or should be recorded as decided
- committed requirements or specs already define the decision

If the agent infers the best direction, use `proposed`.

## Superseding

When replacing a decision:

1. Create or update the new decision.
2. Set the old decision to `superseded`.
3. Add `Superseded by: DEC-...` to the old record.
4. Add `Supersedes: DEC-...` to the new record.
5. Do not delete the old file.

## Evidence Rules

Evidence can be:

- concise user statement
- committed doc reference
- subagent result summary
- repository fact
- observed workflow pain

Do not paste long copyrighted text, secrets, or raw private data.

## Template

Use `assets/decision-template.md` when creating a new decision record.
