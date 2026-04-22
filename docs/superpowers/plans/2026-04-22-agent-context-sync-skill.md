# Agent Context Sync Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the repository-local `agent-context-sync` Codex skill package from the approved requirements and spec.

**Architecture:** The skill is a file-only package under `skills/agent-context-sync/`. `SKILL.md` stays concise and routes to focused reference files; templates live under `assets/` and can be copied into a user's `.agent-context/` workspace folder.

**Tech Stack:** Markdown, YAML frontmatter, Codex skill folder conventions, `skill-creator` validation scripts.

---

## File Structure

- Create: `skills/agent-context-sync/SKILL.md` as the trigger and core workflow.
- Create: `skills/agent-context-sync/references/context-files.md` for workspace contract, startup workflow and safety boundaries.
- Create: `skills/agent-context-sync/references/decision-record-schema.md` for decision file rules.
- Create: `skills/agent-context-sync/references/syncset-protocol.md` for write-before-confirmation protocol.
- Create: `skills/agent-context-sync/references/subagent-briefing.md` for subagent brief creation and integration.
- Create: `skills/agent-context-sync/references/reviewer-checklist.md` for pre-SyncSet review.
- Create: `skills/agent-context-sync/assets/handoff-template.md`.
- Create: `skills/agent-context-sync/assets/session-log-template.md`.
- Create: `skills/agent-context-sync/assets/decision-template.md`.
- Create: `skills/agent-context-sync/assets/subagent-brief-template.md`.
- Create: `skills/agent-context-sync/assets/syncset-template.md`.
- Create: `skills/agent-context-sync/agents/openai.yaml`.

### Task 1: Scaffold Skill Package

**Files:**
- Create: `skills/agent-context-sync/SKILL.md`
- Create: `skills/agent-context-sync/references/`
- Create: `skills/agent-context-sync/assets/`
- Create: `skills/agent-context-sync/agents/openai.yaml`

- [ ] **Step 1: Run the official skill scaffold command**

Run:

```powershell
python C:\Users\Administrator\.codex\skills\.system\skill-creator\scripts\init_skill.py agent-context-sync --path skills --resources references,assets --interface display_name="Agent Context Sync" --interface short_description="Keep AI handoffs, decisions, and subagent briefs synchronized across sessions." --interface default_prompt="Use this skill to update local project context, record decisions with rationale, prepare subagent briefs, and keep platform memory clean."
```

Expected: `skills/agent-context-sync/` exists with `SKILL.md`, `references/`, `assets/`, and `agents/openai.yaml`.

### Task 2: Write Core Skill Instructions

**Files:**
- Modify: `skills/agent-context-sync/SKILL.md`

- [ ] **Step 1: Replace scaffolded `SKILL.md`**

Write frontmatter:

```yaml
---
name: agent-context-sync
description: Use when users need AI work continuity across sessions or subagents, including resuming project progress, updating local context, recording decision rationale, preparing subagent briefs, avoiding platform memory pollution, or synchronizing project-local handoff files.
---
```

Write body sections:

```markdown
# Agent Context Sync

## Core Rules

## Startup Workflow

## SyncSet Before Writes

## Decision Records

## Subagent Briefs

## Reviewer

## References
```

Expected: `SKILL.md` is concise, references all five reference docs, and explicitly forbids task ledger, calendar, `.ics`, scheduler, external writes and platform memo pollution.

### Task 3: Write Reference Docs

**Files:**
- Create: `skills/agent-context-sync/references/context-files.md`
- Create: `skills/agent-context-sync/references/decision-record-schema.md`
- Create: `skills/agent-context-sync/references/syncset-protocol.md`
- Create: `skills/agent-context-sync/references/subagent-briefing.md`
- Create: `skills/agent-context-sync/references/reviewer-checklist.md`

- [ ] **Step 1: Write focused references**

Each reference must include:

- Clear purpose.
- Required behavior.
- Required fields or checklists.
- Safety boundaries.
- One compact example only when useful.

Expected: Reference files are one level from `SKILL.md`, avoid deep nested links, and do not duplicate full spec text.

### Task 4: Write Asset Templates

**Files:**
- Create: `skills/agent-context-sync/assets/handoff-template.md`
- Create: `skills/agent-context-sync/assets/session-log-template.md`
- Create: `skills/agent-context-sync/assets/decision-template.md`
- Create: `skills/agent-context-sync/assets/subagent-brief-template.md`
- Create: `skills/agent-context-sync/assets/syncset-template.md`

- [ ] **Step 1: Write copyable templates**

Expected templates:

- `handoff-template.md` includes current objective, state, next action, blockers, active questions, relevant decisions, files to read first, do-not-reopen list and notes.
- `session-log-template.md` includes goal, what changed, user confirmed, AI inferred, subagent results, context files updated and follow-up.
- `decision-template.md` includes status, date, confirmed by, related sessions, related files, supersedes, superseded by, context, decision, reasons, rejected alternatives, evidence, consequences and review triggers.
- `subagent-brief-template.md` includes mission, role, required context, relevant decisions, files to read, files not to read unless needed, constraints, expected output and non-goals.
- `syncset-template.md` includes summary, proposed updates, AI-inferred items, fields requiring confirmation, reviewer findings and user decision needed.

### Task 5: Validate Skill

**Files:**
- Verify: `skills/agent-context-sync/**`

- [ ] **Step 1: Run official skill validation**

Run:

```powershell
python C:\Users\Administrator\.codex\skills\.system\skill-creator\scripts\quick_validate.py skills\agent-context-sync
```

Expected: validation exits with code `0`.

- [ ] **Step 2: Scan for placeholders and forbidden V1 scope**

Run:

```powershell
Select-String -Path 'skills\agent-context-sync\**\*' -Pattern 'TBD|TODO|placeholder|fill in|task ledger|confirmed-timeblocks|export_ics|weekly-plan|today-plan'
```

Expected: no placeholder matches; forbidden scope appears only as explicit non-goals if present.

- [ ] **Step 3: Check git diff**

Run:

```powershell
git diff --check
git status --short
```

Expected: no whitespace errors; only planned files are modified or added.

### Task 6: Commit and Push

**Files:**
- Commit: `docs/superpowers/plans/2026-04-22-agent-context-sync-skill.md`
- Commit: `skills/agent-context-sync/**`

- [ ] **Step 1: Commit implementation**

Run:

```powershell
git add docs\superpowers\plans\2026-04-22-agent-context-sync-skill.md skills\agent-context-sync
git commit -m "Implement agent context sync skill"
git push origin main
```

Expected: implementation is pushed to `origin/main`.
