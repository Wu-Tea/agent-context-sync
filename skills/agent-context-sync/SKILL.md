---
name: agent-context-sync
description: Use when users need AI work continuity across sessions or subagents, including resuming project progress, updating local context, recording decision rationale, preparing subagent briefs, avoiding platform memory pollution, or synchronizing project-local handoff files.
---

# Agent Context Sync

## Overview

Maintain project-local AI continuity in `.agent-context/`: a short handoff, append-only session log, decision records, and scoped subagent briefs. Keep platform memory clean by storing project state in files that future sessions and subagents can read.

## Core Rules

1. Prefer project-local context over platform memory. Write to platform memory only when the user explicitly asks to remember a stable cross-project preference or fact.
2. Use `.agent-context/` in the current workspace as the default write target.
3. Before editing `.agent-context/`, show a SyncSet and run the reviewer checklist.
4. Do not write until the user clearly confirms, unless the current user request already explicitly authorizes writing the described context files.
5. Mark AI-inferred information as inferred. Do not record it as user-confirmed or accepted.
6. Keep `handoff.md` short and current. Move history to `session-log.md` and rationale to `decisions/`.
7. Do not create task ledgers, calendars, `.ics` files, schedulers, reminders, external issues, Trello cards, Feishu tasks, or external writes in V1.
8. Never store secrets, tokens, cookies, private keys, credentials, or unnecessary personal data.

## Startup Workflow

When resuming work or answering "where did we leave off", first inspect `.agent-context/` before relying on README, docs, git history, or source scanning.

1. Read `.agent-context/handoff.md` if present.
2. Read decision files referenced by the handoff.
3. Read recent relevant entries from `.agent-context/session-log.md`.
4. Read repository docs, git status, git log, or source files only as supplemental context.
5. Report current objective, current state, next action, blockers, active questions, stale context, and related decisions.

If `.agent-context/` is missing, say so and propose a SyncSet to initialize it. See `references/context-files.md`.

## SyncSet Before Writes

A SyncSet is the proposed write set for context files. It must list handoff updates, session-log entries, decision records, subagent briefs, inferred items, sensitive exclusions, and reviewer findings.

Clear confirmation examples: "confirm write", "apply this SyncSet", "write it", "accept after changing X".

Ambiguous examples that are not confirmation: "looks good", "maybe", "almost", "sounds reasonable".

See `references/syncset-protocol.md`.

## Decision Records

Create a decision record for important product, architecture, scope, safety, automation, or process decisions. Record context, decision, reasons, rejected alternatives, evidence, consequences, and review triggers.

Use `proposed` unless the user clearly confirms the decision or committed project docs already establish it. Do not delete decision records; supersede them with links.

See `references/decision-record-schema.md`.

## Subagent Briefs

Before dispatching a subagent, create a scoped brief when context size, role clarity, or prior decisions matter. The brief should include mission, role, required context, related decisions, files to read, files not to read unless needed, constraints, expected output, non-goals, and expiry.

After a subagent returns, summarize useful results into `session-log.md`. Create or update decision records only after user confirmation.

See `references/subagent-briefing.md`.

## Reviewer

Run the reviewer checklist before showing any SyncSet. It checks source quality, confirmation state, scope drift, memo pollution, secrets, oversized handoffs, weak decisions, and subagent brief focus.

Reviewer decisions: `accept_draft`, `needs_user_input`, `revise_once`, `manual_only`, `blocked`.

See `references/reviewer-checklist.md`.

## References

- Use `references/context-files.md` when creating, reading, repairing, or condensing `.agent-context/`.
- Use `references/decision-record-schema.md` when recording, updating, superseding, or reviewing decisions.
- Use `references/syncset-protocol.md` before editing any context file.
- Use `references/subagent-briefing.md` before dispatching or integrating subagents.
- Use `references/reviewer-checklist.md` before presenting a SyncSet.
