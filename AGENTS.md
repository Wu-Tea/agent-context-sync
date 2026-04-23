# AGENTS.md

## Context Continuity

- If this workspace contains `.agent-context/handoff.md`, read it before inferring project state from `README`, `docs`, `git log`, or broad source scanning.
- If the user asks where work stopped, how to continue, to update project context, to record a decision, to prepare or integrate subagents, or to avoid polluting platform memory, use `$agent-context-sync`.
- After substantial design, implementation, review, or subagent work in this repository, propose an update to `.agent-context/`.
- Do not write `.agent-context/` files without explicit user confirmation unless the user already clearly asked to record or update project context.
