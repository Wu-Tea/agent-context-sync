# Context Files

Use this reference when creating, reading, repairing, or condensing `.agent-context/`.

## Workspace Contract

Default layout:

```text
.agent-context/
  handoff.md
  session-log.md
  decisions/
    DEC-YYYY-MM-DD-NNN-short-title.md
  briefs/
    subagent-YYYY-MM-DD-NNN-topic.md
```

Do not create task ledger, plan, calendar, `.ics`, scheduler, or connector files in V1.

## Initialization

If `.agent-context/` is missing:

1. State that no project-local context sync folder was found.
2. Explain that recovery can only use repository files, git state, and current conversation until context is initialized.
3. Propose a SyncSet with `create_context_dir`.
4. After explicit confirmation, create `handoff.md`, `session-log.md`, `decisions/`, and `briefs/`.

Use these assets:

- `assets/handoff-template.md`
- `assets/session-log-template.md`

Create empty `decisions/` and `briefs/` directories during initialization.

## Startup Read Order

When asked to resume work:

1. Read `.agent-context/handoff.md`.
2. Read decisions listed in the handoff.
3. Read recent relevant entries from `.agent-context/session-log.md`.
4. Read README, specs, git status, git log, or source files only as supplemental context.
5. If context files conflict with repository state, report the conflict and propose a SyncSet. Do not silently overwrite context.

## `handoff.md`

Purpose: current handoff only.

Required sections:

- `Current Objective`
- `Current State`
- `Next Action`
- `Blockers`
- `Active Questions`
- `Relevant Decisions`
- `Files To Read First`
- `Do Not Reopen Unless Needed`
- `Notes`

Rules:

- Keep the file short enough to scan quickly.
- Keep one best next action.
- Include a staleness condition.
- Link to decision records instead of copying full rationale.
- Move historical details to `session-log.md`.
- Move durable rationale to `decisions/`.

## `session-log.md`

Purpose: append important session outcomes.

Each entry should include:

- timestamp and short title
- goal
- what changed
- user-confirmed items
- AI-inferred items
- subagent results
- context files updated
- follow-up

Rules:

- Append new entries in chronological order.
- Summarize; do not transcribe full chat.
- Use correction entries instead of silently rewriting history.
- Do not store secrets or unnecessary personal data.

## Repair Rules

If a context file is malformed:

1. Do not overwrite it.
2. Report the malformed section.
3. Propose a repair SyncSet.
4. Preserve original content unless the user explicitly asks to replace it.

If `handoff.md` becomes too long:

1. Propose condensing it.
2. Move history to `session-log.md`.
3. Move rationale to `decisions/`.
4. Keep current objective, state, next action, blockers, questions, and read list.
