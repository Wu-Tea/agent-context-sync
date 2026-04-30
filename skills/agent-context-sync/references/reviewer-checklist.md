# Reviewer Checklist

Use this reference before showing any SyncSet.

## Decision

Set one reviewer decision:

- `accept_draft`: safe to show SyncSet.
- `needs_user_input`: ask one concise question first.
- `revise_once`: fix the SyncSet and rerun this checklist.
- `manual_only`: keep as discussion or manual note, not durable memory.
- `blocked`: do not write; explain why.

## Checklist

Check every SyncSet for:

- Evidence: every proposed decision has context and evidence.
- Confirmation: AI-inferred items are not marked as user-confirmed.
- Handoff size: `handoff.md` remains short and current.
- History placement: historical details go to `session-log.md`, not handoff.
- Session-log size: `session-log.md` remains readable as a primary startup file.
- Archive compatibility: if detailed history was compacted, `session-log.md` still links to archived ranges and remains useful without opening archive first.
- Rationale placement: durable reasoning goes to `decisions/`, not handoff.
- Alternatives: major decisions include rejected alternatives.
- Superseding: conflicting decisions are linked with supersedes or superseded-by.
- Brief focus: subagent briefs have one narrow mission.
- Brief size: briefs do not copy full specs or broad repo summaries.
- Secrets: tokens, credentials, cookies, keys, and private data are excluded.
- Memo cleanliness: project-local facts are not proposed for platform memory.
- Scope: V1 does not drift into task management, calendar, `.ics`, scheduler, reminders, or external connectors.
- Write boundary: writes stay inside `.agent-context/` unless explicitly approved.
- User burden: the SyncSet is easier to review than re-explaining the context.
- Size self-check:
  - suggest handoff compaction above 80 lines and strongly suggest it above 120 lines
  - suggest session-log compaction above 100 lines and strongly suggest it above 160 lines

## Common Fixes

- If the handoff is too long, move history to session log and rationale to decisions.
- If completed work no longer affects the next resume, remove it from handoff and keep only the milestone trace in session log or archive.
- If the session log is too long, compact it into milestone summaries and archive the deeper narrative instead of inventing a new primary file.
- If a decision lacks rejected alternatives, add them or mark the record `proposed`.
- If a brief is too broad, split it into one brief per subagent mission.
- If user confirmation is unclear, ask before writing.
- If a request implies external action, record a manual note instead of executing.
