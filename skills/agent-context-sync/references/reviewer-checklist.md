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

## Common Fixes

- If the handoff is too long, move history to session log and rationale to decisions.
- If a decision lacks rejected alternatives, add them or mark the record `proposed`.
- If a brief is too broad, split it into one brief per subagent mission.
- If user confirmation is unclear, ask before writing.
- If a request implies external action, record a manual note instead of executing.
