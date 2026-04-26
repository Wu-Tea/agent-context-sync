---
name: using-agent-context-sync
description: Use when starting, resuming, or continuing a conversation in a workspace that might contain ongoing project context, `.agent-context/`, prior handoff files, or user requests such as "上次做到哪里", "继续这个项目", "更新上下文", "记录决策", "生成 subagent brief", or "同步项目记忆". This bootstrap skill decides whether to route the conversation through agent-context-sync before broad repository exploration.
---

<SUBAGENT-STOP>
If you were dispatched as a narrow subagent for a concrete task and the user did not ask for continuity, handoff, project memory, or decision tracking, skip this bootstrap skill.
</SUBAGENT-STOP>

# Using Agent Context Sync

## Overview

Use this bootstrap skill at the start of work to decide whether the conversation should route through `agent-context-sync`. This skill does not replace `agent-context-sync`; it decides when continuity-aware startup should happen before broad repository rediscovery.

## Routing Rule

If any strong continuity cue exists, load or follow `agent-context-sync` before broad repository exploration, README-first summaries, or large git/source scans.

If no strong continuity cue exists and the task is clearly one-shot, do not force `agent-context-sync`.

## Strong Continuity Cues

Route through `agent-context-sync` when one or more of these are true:

- The workspace contains `.agent-context/` or `.agent-context/handoff.md`.
- The user asks where work stopped, what happened last time, how to continue, or what the next step is.
- The user asks to update context, project memory, handoff, session log, or decision records.
- The user asks to prepare, brief, or integrate subagents.
- The user asks to keep platform memory clean and store project state locally.
- The conversation is clearly part of an ongoing project thread rather than a fresh one-off task.

## Negative Cues

Do not force `agent-context-sync` when all of the following are true:

- The task is clearly one-shot.
- There is no `.agent-context/` folder.
- The user is not asking about prior progress, continuity, decisions, project memory, or subagents.
- Broad repository rediscovery would not lose important prior state.

Examples:

- "What does this regex mean?" with no project continuity need.
- "Format this JSON."
- "Explain this single error message" when no ongoing project context matters.

## Startup Flow

1. Check whether `.agent-context/` or `.agent-context/handoff.md` exists.
2. Check whether the user request contains continuity cues.
3. If yes, route to `agent-context-sync`.
4. If no, continue normally.

## Fallback

If `agent-context-sync` is unavailable but `.agent-context/handoff.md` exists, read the handoff before broad repository rediscovery and say that continuity routing would normally use `agent-context-sync`.

## Scope Boundary

This bootstrap skill is intentionally narrow:

- It should not create or edit `.agent-context/` by itself.
- It should not replace `agent-context-sync`.
- It should not hijack clearly one-shot tasks.
- It should only improve the odds that continuity-aware startup happens early enough.
