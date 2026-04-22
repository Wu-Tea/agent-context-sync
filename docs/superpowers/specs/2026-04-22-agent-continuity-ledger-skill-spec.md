# Agent Context Sync and Decision Memory Skill - Spec

日期: 2026-04-22
状态: 实施前规格草案
需求来源: `docs/superpowers/specs/2026-04-22-agent-continuity-ledger-skill-requirements.md`
目标产物: Codex skill `agent-context-sync`
历史名称: `agent-continuity-ledger`

## 1. Spec 结论

第一版实现一个本地 Codex skill，用 `.agent-context/` 帮助 AI 在多 session、多 subagent 工作中同步必要上下文，并记录重要 decision 的由来。

最小闭环:

```text
用户讨论项目或要求同步上下文
  -> 读取 .agent-context/handoff.md、session-log.md 和相关 decisions
  -> 整理本次变化、决策和 subagent 需求
  -> 生成 SyncSet
  -> Reviewer 自检
  -> 用户确认
  -> 更新 handoff.md、追加 session-log.md、创建或更新 decision records、生成 subagent briefs
  -> 下次 session 或 subagent 先读取这些文件再继续
```

核心设计决策:

- 使用 Markdown 作为 V1 存储格式，优先保证人类可读、git diff 友好、AI 易编辑。
- 使用 `.agent-context/` 而不是 `.agent-ledger/`，避免产品语义滑向任务账本。
- `handoff.md` 只保存当前交接状态，不保存完整历史。
- `session-log.md` 用追加日志保存重要进展和 subagent 结果。
- `decisions/*.md` 用 ADR-like 文件保存重要决策、原因、备选方案和影响。
- `briefs/*.md` 用最小上下文包支持 subagent，不追求继承完整 memory。
- 使用 `SyncSet` 作为所有写入前的用户确认层。
- 使用 reviewer checklist 作为上下文质量门，不把 reviewer 包装成绝对安全保证。
- V1 不做 task ledger、calendar、`.ics`、scheduler 或外部 connector。

## 2. Skill 包结构

目标目录:

```text
agent-context-sync/
  SKILL.md
  references/
    context-files.md
    decision-record-schema.md
    syncset-protocol.md
    subagent-briefing.md
    reviewer-checklist.md
  assets/
    handoff-template.md
    decision-template.md
    session-log-template.md
    subagent-brief-template.md
    syncset-template.md
  agents/
    openai.yaml
```

V1 不包含:

- `scripts/export_ics.py`
- `validate_ledger.py`
- task planning rules
- calendar export rules
- external connector
- scheduler
- app UI

## 3. `SKILL.md` 规格

### 3.1 Frontmatter

```yaml
---
name: agent-context-sync
description: Keep AI work continuity across sessions and subagents by maintaining local handoff files, session logs, decision records, and subagent briefs. Use when users ask to remember project progress, update context, record why a decision was made, prepare subagents, resume prior work, avoid polluting long-term memory, or synchronize necessary project context in local files.
---
```

触发描述必须覆盖:

- cross-session continuity
- subagent briefing
- handoff update
- decision memory
- project-local context
- memo cleanliness
- resume work
- decision rationale

### 3.2 Body 结构

`SKILL.md` 正文应保持短，只放核心流程和引用导航。

推荐结构:

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

`SKILL.md` 必须明确:

- 默认上下文目录是当前 workspace 的 `.agent-context/`。
- 新 session 恢复工作时，先读 `handoff.md`，再读相关 decisions 和最近 `session-log.md`。
- 写入 `.agent-context/` 前必须先展示 SyncSet。
- 用户未确认前，不得把 AI 推断写成 accepted decision。
- 长期 memo 只保存稳定偏好和跨项目事实，项目临时上下文写入 `.agent-context/`。
- subagent brief 应最小化，只包含任务必要上下文。
- V1 不做任务看板、日程、`.ics`、后台自动执行或外部写入。

### 3.3 Reference 加载规则

`SKILL.md` 按需读取 references:

- 需要创建或修复 `.agent-context/` 文件时，读 `references/context-files.md`。
- 需要记录、更新、复审 decision 时，读 `references/decision-record-schema.md`。
- 需要生成或应用 SyncSet 时，读 `references/syncset-protocol.md`。
- 需要准备 subagent 时，读 `references/subagent-briefing.md`。
- 展示 SyncSet 前，读或执行 `references/reviewer-checklist.md` 中的 checklist。

## 4. Workspace 文件结构

Skill 在用户当前项目根目录维护:

```text
.agent-context/
  handoff.md
  session-log.md
  decisions/
    DEC-YYYY-MM-DD-NNN-short-title.md
  briefs/
    subagent-YYYY-MM-DD-NNN-topic.md
```

创建规则:

- 如果 `.agent-context/` 不存在，先展示将创建的文件列表并请求确认。
- `handoff.md` 和 `session-log.md` 在初始化时创建。
- `decisions/` 和 `briefs/` 在初始化时创建空目录。
- decision 和 brief 只在有实际内容时创建文件。
- 不创建 task ledger、plan、calendar 文件。

安全边界:

- 默认只写 `.agent-context/`。
- 不删除 decision 历史文件。
- 不覆盖用户未确认的上下文内容。
- 如需写入 `.agent-context/` 之外的路径，必须显示目标路径并等待用户明确确认。

## 5. Startup workflow

当用户提出以下请求时触发 startup workflow:

- “上次做到哪里了？”
- “继续这个项目。”
- “读取上下文。”
- “根据项目记忆接着做。”
- “帮我同步一下当前状态。”
- “给 subagent 准备上下文。”

执行顺序:

1. 检查 `.agent-context/handoff.md` 是否存在。
2. 如果存在，先读取 `handoff.md`。
3. 根据 handoff 中的相关 decision 链接读取 `decisions/*.md`。
4. 读取 `session-log.md` 中最近的相关条目。
5. 再按任务需要读取 README、spec、git status、git log 或源码。
6. 输出当前目标、当前状态、下一步、阻塞、活跃问题和可能过期的上下文。
7. 如果 `.agent-context/` 与 repo 状态冲突，生成 SyncSet 或澄清问题，不直接覆盖。

缺失上下文规则:

1. 如果 `.agent-context/` 不存在，说明没有发现本地上下文同步包。
2. 说明当前只能基于仓库文件、git 状态和对话内容推断。
3. 提供初始化 `.agent-context/` 的 SyncSet。
4. 等待用户确认后再创建文件。

## 6. `handoff.md` 规格

模板:

```markdown
# Agent Handoff

Last updated: 2026-04-22T18:00:00+08:00
Updated by: Codex
Staleness: stale after the current design/spec phase changes

## Current Objective

One or two sentences.

## Current State

Short summary of where the project stands.

## Next Action

The single best next action.

## Blockers

- None

## Active Questions

- Question or assumption that still needs user input.

## Relevant Decisions

- DEC-2026-04-22-001-context-sync-scope.md

## Files To Read First

- docs/superpowers/specs/...

## Do Not Reopen Unless Needed

- Files or topics that were already rejected or are not needed for the next step.

## Notes

Short, current-only notes.
```

Rules:

- Keep under roughly 150 lines.
- Prefer links to decision records over copying full rationale.
- Include stale condition.
- Keep one clear next action.
- Do not store secrets.
- Do not store long chat transcripts.
- Do not list every task unless directly needed for handoff.

## 7. `session-log.md` 规格

Template:

```markdown
# Agent Session Log

## 2026-04-22T18:00:00+08:00 - Context sync requirements update

Goal: Update requirements and spec around context sync and decision memory.

What changed:
- Reframed V1 away from task ledger and calendar export.
- Introduced .agent-context/ as the workspace contract.

User confirmed:
- Decision memory is required.

AI inferred:
- V1 should avoid .ics and scheduler until context sync is proven.

Subagent results:
- None in this session.

Context files updated:
- handoff.md
- decisions/DEC-...

Follow-up:
- Review the updated spec before implementation.
```

Rules:

- Append entries chronologically.
- Summarize only important changes.
- Record subagent outcomes after integration.
- Use correction entries rather than silently rewriting history when meaning changes.
- Avoid storing raw sensitive text.

## 8. Decision record 规格

Filename:

```text
.agent-context/decisions/DEC-YYYY-MM-DD-NNN-short-title.md
```

Template:

```markdown
# DEC-YYYY-MM-DD-NNN: Decision Title

Status: proposed
Date: 2026-04-22
Confirmed by: pending
Related sessions:
- 2026-04-22T18:00:00+08:00
Related files:
- docs/superpowers/specs/...
Supersedes: none
Superseded by: none

## Context

What was happening when this decision was considered.

## Decision

The chosen option.

## Reasons

- Reason 1

## Rejected Alternatives

- Alternative: why rejected.

## Evidence

- Source, observation, subagent result, user statement, or repo fact.

## Consequences

- Expected tradeoff or follow-up.

## Review Triggers

- Condition that should cause this decision to be revisited.
```

Status values:

- `proposed`
- `accepted`
- `superseded`
- `rejected`

Rules:

- Do not mark as `accepted` unless user clearly confirms or the decision is already established by committed project docs.
- AI-generated recommendations start as `proposed`.
- When replacing a decision, update the old record to `superseded` and link the new record.
- Do not delete old decision files.
- Always include rejected alternatives for important product or architecture choices.
- Evidence can be a concise paraphrase; do not copy long copyrighted or sensitive text.

## 9. Subagent brief 规格

Filename:

```text
.agent-context/briefs/subagent-YYYY-MM-DD-NNN-topic.md
```

Template:

```markdown
# Subagent Brief: Topic

Created: 2026-04-22T18:00:00+08:00
Expires: after the current design review
Owner session: main

## Mission

What the subagent should accomplish.

## Role

Supporter, light skeptic, strong skeptic, explorer, worker, reviewer, or other.

## Required Context

- Minimal facts the subagent must know.

## Relevant Decisions

- DEC-...

## Files To Read

- Specific files.

## Files Not To Read Unless Needed

- Paths or areas that are likely irrelevant.

## Constraints

- Scope, safety, write boundaries, or assumptions.

## Expected Output

- The format and level of detail expected.

## Non-goals

- What the subagent should not solve.
```

Rules:

- Brief should be scoped to one subagent mission.
- Do not include full repo summaries.
- Do not include private user memory unless necessary and confirmed.
- Prefer file links and decision IDs over long copied context.
- After subagent returns, summarize useful results into `session-log.md`.
- Create or update decisions only if the result leads to a user-confirmed decision.

## 10. SyncSet 规格

SyncSet is the write proposal shown before editing `.agent-context/`.

Template:

```markdown
## SyncSet sync-YYYYMMDD-NNN

### Summary

- Handoff updates: 1
- Session log entries: 1
- Decision records: 1 new, 0 updated
- Subagent briefs: 0
- Items needing confirmation: 1
- Excluded sensitive items: 0

### Proposed Handoff Updates

- Update Current Objective to: ...
- Update Next Action to: ...

### Proposed Session Log Entries

- Add entry for 2026-04-22T18:00:00+08:00 summarizing ...

### Proposed Decision Records

#### DEC-2026-04-22-001-context-sync-scope.md

Status: proposed
Decision: V1 focuses on .agent-context/ rather than task ledger/calendar.
Requires confirmation: yes

### Proposed Subagent Briefs

- None

### AI-Inferred Items

- V1 should defer .ics because the current pain is context continuity, not scheduling.

### Fields Requiring User Confirmation

- Whether to rename the skill to agent-context-sync.

### Reviewer Findings

- No secrets detected.
- V1 scope remains outside task manager/calendar.

### User Decision Needed

Confirm whether to apply this SyncSet.
```

SyncSet ID:

```text
sync-YYYYMMDD-NNN
```

Allowed actions:

- `create_context_dir`
- `update_handoff`
- `append_session_log`
- `create_decision`
- `update_decision_status`
- `create_subagent_brief`
- `mark_stale`
- `add_correction`

Forbidden actions:

- `delete_decision`
- `write_platform_memo`
- `external_write`
- `run_command`
- `send_message`
- `delete_file`
- `create_calendar_event`

Applying rules:

- Require explicit user confirmation before writes.
- If user says “修改 X 后写入”, adjust SyncSet and apply the adjusted version.
- If instruction is ambiguous, ask one concise clarifying question.
- After applying, summarize files changed and any remaining open questions.

## 11. Reviewer 规格

Reviewer runs before showing a SyncSet.

Checklist:

- Does every proposed decision have context and evidence?
- Are AI-inferred items marked as inferred?
- Is any user confirmation being overstated?
- Is the handoff short enough to be useful?
- Is the handoff focused on current state rather than full history?
- Are rejected alternatives recorded for major decisions?
- Are superseded decisions linked instead of overwritten?
- Does the subagent brief have a narrow mission?
- Does the subagent brief avoid dumping unnecessary repo context?
- Are secrets, tokens, credentials or personal data excluded?
- Is any project-local fact being proposed for platform memo without explicit user request?
- Is the scope drifting into task manager, calendar, scheduler or external connector?
- Are file paths and write targets inside `.agent-context/` unless explicitly approved?

Reviewer decisions:

- `accept_draft`: show SyncSet to user.
- `needs_user_input`: ask user before showing or applying.
- `revise_once`: revise the SyncSet once and rerun reviewer.
- `manual_only`: explain that the item should stay as a manual note or discussion, not written as durable memory.
- `blocked`: do not write; explain why.

## 12. Memo cleanliness rules

Skill must protect platform memory from project-local noise.

Write to platform memo only when:

- User explicitly asks to remember something long term.
- The information is stable across projects.
- The information is a preference, identity, durable workflow rule or durable constraint.

Write to `.agent-context/` when:

- It is about the current project.
- It helps future sessions resume.
- It helps subagents understand scope.
- It explains a design or product decision.
- It may become stale after this project or phase.

Do not write anywhere when:

- It is a secret.
- It is unverified sensitive data.
- It is a transient thought with no future value.
- It is a long raw transcript.

## 13. Error handling

### 13.1 Missing `.agent-context/`

Expected behavior:

1. State that no context sync folder exists.
2. Offer a SyncSet to create it.
3. Wait for confirmation.
4. Create `handoff.md`, `session-log.md`, `decisions/` and `briefs/`.

### 13.2 Malformed context files

Expected behavior:

1. Do not overwrite.
2. Report the malformed section.
3. Propose a repair SyncSet.
4. Preserve original content unless user explicitly asks to replace it.

### 13.3 Conflicting decisions

Expected behavior:

1. Identify the conflicting decision records.
2. Ask whether the new decision supersedes the old one.
3. If confirmed, update statuses and links.
4. If not confirmed, record as open question, not accepted decision.

### 13.4 Oversized handoff

Expected behavior:

1. Propose condensing handoff.
2. Move durable rationale to decisions.
3. Move historical summary to session log.
4. Keep only current objective, state and next action in handoff.

### 13.5 High-risk or external action

Expected behavior:

1. Do not execute external action.
2. Record only a manual note or proposed decision if appropriate.
3. Ask for explicit user confirmation before any future external write.

## 14. Acceptance criteria

The skill is acceptable when:

- `SKILL.md` frontmatter triggers on context sync, handoff, decision memory and subagent briefing requests.
- Skill initializes `.agent-context/` only after user confirmation.
- A new session reads `handoff.md` before relying on README/git inference.
- Relevant decisions are loaded when resuming a project.
- Session summaries are appended to `session-log.md`.
- Important decisions are stored as individual files in `decisions/`.
- Decision records include reasons, rejected alternatives, evidence and review triggers.
- Subagent briefs include mission, role, required context, files to read, constraints and expected output.
- SyncSet is shown before any `.agent-context/` write.
- Reviewer checklist runs before SyncSet is shown.
- AI-inferred items are clearly marked.
- Platform memo remains clean unless user explicitly requests long-term memory.
- No task ledger, calendar, `.ics`, scheduler or external connector is implemented in V1.

## 15. Test cases

### 15.1 Initialize context folder

Input:

```text
帮我开始维护这个项目的 AI 上下文。
```

Expected:

- Skill reports `.agent-context/` is missing.
- Skill proposes a SyncSet to create `handoff.md`, `session-log.md`, `decisions/` and `briefs/`.
- Skill waits for confirmation before writing.

### 15.2 Resume work

Input:

```text
上次做到哪里了？
```

Expected:

- Skill reads `handoff.md` first.
- Skill reads relevant decisions.
- Skill summarizes current objective, current state, next action and blockers.
- README/git status are supplemental, not primary memory.

### 15.3 Record decision

Input:

```text
记录一下，我们决定 V1 不做日历导出，先做上下文同步。
```

Expected:

- Skill proposes a decision record.
- Decision includes context, decision, reasons, rejected alternatives and review triggers.
- Status can be `accepted` because the user explicitly said “我们决定”。
- Skill writes only after SyncSet confirmation unless current interaction already includes explicit write approval.

### 15.4 Proposed decision from AI inference

Input:

```text
你觉得这个 skill 要不要以后支持飞书？
```

Expected:

- Skill can discuss options.
- If recording, decision status is `proposed`, not `accepted`.
- Requires user confirmation before becoming accepted.

### 15.5 Generate subagent brief

Input:

```text
给一个强烈反对方 subagent 准备 brief，让它评审这个 V1。
```

Expected:

- Skill creates a focused brief with role, mission, context, files to read and non-goals.
- Brief references relevant decisions instead of copying all content.
- Brief tells subagent not to re-solve unrelated app/calendar/scheduler scope unless needed.

### 15.6 Integrate subagent result

Input:

```text
把这个 subagent 的结论同步进上下文。
```

Expected:

- Skill summarizes useful result into `session-log.md`.
- If result changes product direction, proposes a decision record.
- Does not mark decision accepted unless user confirms.

### 15.7 Prevent memo pollution

Input:

```text
记住这个项目现在先不做 .ics。
```

Expected:

- Skill asks or infers that this is project-local context.
- Writes to `.agent-context/decisions/` or `handoff.md`, not platform memo.
- Explains that memo is reserved for durable cross-project preferences.

## 16. Dogfood protocol

Before building an app, scheduler or connector:

1. Use the skill in at least 10 real project sessions.
2. Use it for at least 3 subagent dispatches.
3. Record at least 5 important decisions.
4. Start at least 2 fresh sessions and test recovery from `.agent-context/`.
5. Track whether handoff stays short.
6. Track whether subagent briefs reduce irrelevant repo reading.
7. Track whether memo remains clean.

Pass criteria:

- Fresh session recovers context in under 2 minutes.
- User can explain why major decisions were made by reading decision records.
- Subagent prompts become shorter and more focused.
- At least 7 of 10 sessions produce useful continuity.
- Handoff remains readable.
- No accepted decision lacks rationale.

Fail criteria:

- User stops reading handoff.
- Decisions are still buried in chat history.
- Subagents continue scanning the whole repo unnecessarily.
- SyncSet feels slower than manual explanation.
- Context files become noisy enough that future agents ignore them.

## 17. Implementation order

Implementation should proceed in this order:

1. Create skill scaffold for `agent-context-sync`.
2. Write concise `SKILL.md`.
3. Add `references/context-files.md`.
4. Add `references/decision-record-schema.md`.
5. Add `references/syncset-protocol.md`.
6. Add `references/subagent-briefing.md`.
7. Add `references/reviewer-checklist.md`.
8. Add templates under `assets/`.
9. Generate or update `agents/openai.yaml`.
10. Validate skill folder.
11. Dogfood on this repository by creating `.agent-context/`.
12. Run fresh-session and subagent-brief simulations.

Do not implement calendar, scheduler, external connector or app UI before dogfood passes.

## 18. Open decisions resolved by this spec

- V1 product type: local Codex skill, not app.
- V1 workspace contract: `.agent-context/`.
- V1 memory model: handoff + session log + decision records + subagent briefs.
- V1 write protocol: SyncSet before writes.
- V1 quality gate: reviewer checklist.
- V1 scope: no task ledger, no `.ics`, no scheduler, no external writes.
- Decision memory is core, not optional.
- Subagent context sync is core, not optional.

## 19. Remaining open questions

- Whether to rename repository/docs from `agent-continuity-ledger` to `agent-context-sync`.
- Whether SyncSet confirmation can be relaxed for low-risk `session-log.md` append operations after user opt-in.
- Whether `index.md` is needed after dogfood, or `handoff.md` is enough.
- Whether decision IDs should be globally monotonic or date-local monotonic.
- Whether `.agent-context/` should be committed to git by default or ignored per project.
- Whether future cloud/team versions should treat `.agent-context/` as sync source or export artifact.
