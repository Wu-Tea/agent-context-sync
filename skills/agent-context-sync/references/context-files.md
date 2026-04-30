# Context Files

Use this reference when creating, reading, repairing, or condensing `.agent-context/`.

## Workspace Contract

Default layout:

```text
.agent-context/
  handoff.md
  session-log.md
  archive/
    session-log-YYYY-MM-DD-to-YYYY-MM-DD-pre-compaction.md
  decisions/
    DEC-YYYY-MM-DD-NNN-short-title.md
  briefs/
    subagent-YYYY-MM-DD-NNN-topic.md
```

`archive/` is optional. Older or smaller projects can remain valid without it.

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
4. If `session-log.md` links to archived detail, read the linked archive only when deeper history is needed.
5. Read README, specs, git status, git log, or source files only as supplemental context.
6. If context files conflict with repository state, report the conflict and propose a SyncSet. Do not silently overwrite context.

Treat these as startup-continuity cues:

- `.agent-context/handoff.md` exists.
- The user says "上次做到哪里", "继续这个项目", "更新上下文", "记录决策", "同步项目记忆", or asks for a subagent brief.
- The task is clearly part of an ongoing design, implementation, review, or subagent thread.

If the task is clearly one-shot and does not need continuity, handoff, or subagent context, this skill can stay out of the way.

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
- Self-check line count before writing:
  - soft suggestion threshold: over 80 lines
  - strong suggestion threshold: over 120 lines
- Completed work that no longer affects the next resume should be condensed out of the handoff and moved to milestone summaries, decision links, or archive-backed history.

## `session-log.md`

Purpose: remain the primary session-history entry point while preserving important outcomes and links to deeper history.

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

- Append new entries in chronological order until compaction is needed.
- Summarize; do not transcribe full chat.
- Use correction entries instead of silently rewriting history.
- Do not store secrets or unnecessary personal data.
- `session-log.md` stays the main file name even after compaction; do not replace it with a new primary file such as `session-index.md`.
- When detailed history becomes too large, `session-log.md` may be rewritten into:
  - an archive note
  - milestone summaries
  - recent active checkpoints
- If archive-backed history exists, keep links from `session-log.md` to the archived files.
- Self-check line count before writing:
  - soft suggestion threshold: over 100 lines
  - strong suggestion threshold: over 160 lines

## `archive/`

Purpose: optional detail layer for older session history after compaction.

Rules:

- Archive files are created only when compaction is proposed and confirmed.
- Preserve chronological traceability in archive file names, for example:
  - `session-log-2026-04-26-to-2026-04-28-pre-compaction.md`
- Do not make `archive/` a required startup read target.
- Keep `session-log.md` readable on its own; archive is for deeper recovery, not routine startup.

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

If `session-log.md` becomes too long:

1. Propose compaction instead of silently rewriting it.
2. Preserve `session-log.md` as the primary startup entry point.
3. Optionally create an archive file under `.agent-context/archive/`.
4. Rewrite the main file into archive note plus milestone summaries plus recent active checkpoints.
5. Preserve links to archived ranges so deeper history is still recoverable.
