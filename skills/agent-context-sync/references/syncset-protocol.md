# SyncSet Protocol

Use this reference before editing `.agent-context/`.

## Purpose

A SyncSet is a proposed write set. It makes context updates reviewable before they become durable project memory.

## Required Sections

Each SyncSet should include:

- summary counts
- proposed handoff updates
- proposed session-log entries
- proposed compaction actions
- proposed decision records
- proposed subagent briefs
- AI-inferred items
- fields requiring user confirmation
- sensitive or excluded items
- reviewer findings
- user decision needed

Use `assets/syncset-template.md` as the base.

## SyncSet ID

Use:

```text
sync-YYYYMMDD-NNN
```

Default numbering is date-local monotonic within the current session. If no prior SyncSet is visible, start at `001`.

## Allowed Actions

- `create_context_dir`
- `update_handoff`
- `append_session_log`
- `condense_handoff`
- `compact_session_log`
- `create_archive_file`
- `create_decision`
- `update_decision_status`
- `create_subagent_brief`
- `mark_stale`
- `add_correction`

## Forbidden Actions

- `delete_decision`
- `write_platform_memo`
- `external_write`
- `run_command`
- `send_message`
- `delete_file`
- `create_calendar_event`
- `create_task_ledger`
- `export_ics`

## Confirmation Rules

Treat these as clear confirmation:

- "confirm write"
- "write it"
- "apply this SyncSet"
- "accept this SyncSet"
- "sync this"
- "write after changing X"

Do not treat these as confirmation:

- "looks good"
- "maybe"
- "almost"
- "sounds reasonable"
- "we can consider it"

If the same user request explicitly asks you to update or write the context files and the SyncSet exactly matches that request, you may apply after showing the SyncSet summary. If there is any ambiguity, stop and ask one concise question.

## Applying A SyncSet

Before applying:

1. Run the reviewer checklist.
2. Ensure all inferred items are marked.
3. Ensure sensitive items are excluded.
4. Ensure writes are inside `.agent-context/` unless explicitly approved.
5. If compaction is proposed, ensure history remains traceable and no archive-backed detail is silently discarded.

After applying:

1. Summarize files changed.
2. Mention decisions created or updated.
3. Mention any archive file created and what date range or history slice it preserves.
4. Mention remaining open questions.
5. Do not claim broader completion than was verified.
