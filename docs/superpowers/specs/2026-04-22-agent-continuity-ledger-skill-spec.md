# Agent Continuity Ledger Skill - Spec

日期: 2026-04-22
状态: 实施前规格草案
需求来源: `docs/superpowers/specs/2026-04-22-agent-continuity-ledger-skill-requirements.md`
目标产物: Codex skill `agent-continuity-ledger`

## 1. Spec 结论

第一版实现一个 Codex skill, 名称为 `agent-continuity-ledger`。

它的最小闭环是:

```text
用户输入任务/资料
  -> 读取 .agent-ledger/task-ledger.md
  -> 生成 ChangeSet
  -> Reviewer 自检
  -> 用户确认
  -> 更新 task-ledger.md 和 decision-log.md
  -> 生成 today-plan.md 或 weekly-plan.md
  -> 可选生成 confirmed-timeblocks.json
  -> 可选调用 export_ics.py 生成 confirmed-timeblocks.ics
```

核心设计决策:

- 使用 Markdown 作为第一版账本源, 因为它适合人工审阅、git diff 和 AI 编辑。
- 使用结构化 fenced YAML block 表达 task metadata, 避免纯自然语言账本腐烂。
- 使用 ChangeSet 作为所有写入前的中间层, 避免 AI 直接改账本。
- 使用 Reviewer checklist 作为规则化自检, 不把 Reviewer 包装成绝对安全门。
- 使用 `.ics` 作为已确认时间块的投影文件, 不作为任务状态源。
- 使用 `confirmed-timeblocks.json` 作为 `.ics` 脚本输入, 避免脚本解析自由格式 Markdown。
- 不假设 Codex 自带跨 session 任务记忆。跨 session 连续性必须通过读取 `.agent-ledger/task-ledger.md` 显式恢复。

## 2. Skill 包结构

目标目录:

```text
agent-continuity-ledger/
  SKILL.md
  scripts/
    export_ics.py
  references/
    task-ledger-schema.md
    changeset-protocol.md
    planning-rules.md
    calendar-export-rules.md
  assets/
    task-ledger-template.md
    today-plan-template.md
    weekly-plan-template.md
  agents/
    openai.yaml
```

V1 不实现:

- `validate_ledger.py`
- external connector
- scheduler
- app UI
- webcal server
- OAuth
- CalDAV

这些可以进入后续版本。

## 3. `SKILL.md` 规格

### 3.1 Frontmatter

`SKILL.md` 必须包含:

```yaml
---
name: agent-continuity-ledger
description: Maintain an evidence-based task ledger, review task changes, preserve work continuity across sessions, generate daily or weekly Markdown plans, and optionally export confirmed time blocks to .ics calendar files when users discuss tasks, priorities, schedules, next actions, planning, or AI handoff.
---
```

触发描述必须覆盖:

- task capture
- task memory
- task review
- next action
- daily plan
- weekly plan
- calendar export
- AI handoff

### 3.2 Body 结构

`SKILL.md` 正文保持短, 只放执行流程和引用导航。

建议结构:

```markdown
# Agent Continuity Ledger

## Core Rules

## Workflow

## Ledger Location

## ChangeSet Before Edits

## Planning

## Calendar Export

## References
```

`SKILL.md` 必须明确:

- 默认 ledger 目录为当前工作区的 `.agent-ledger/`。
- 修改账本前必须先生成 ChangeSet。
- 用户未确认前不得写入 `task-ledger.md`、`decision-log.md` 或 `.ics`。
- `.ics` 只导出 confirmed time blocks。
- 遇到用户要求外部写入、发消息、删除文件、执行命令时, 必须拒绝自动执行并改为生成人工任务或草稿。

### 3.3 Reference 加载规则

`SKILL.md` 应按需读取 references:

- 处理账本结构时读取 `references/task-ledger-schema.md`。
- 生成或应用 ChangeSet 时读取 `references/changeset-protocol.md`。
- 生成今日/本周计划时读取 `references/planning-rules.md`。
- 导出 `.ics` 时读取 `references/calendar-export-rules.md`。

## 4. 工作区文件结构

Skill 在用户当前项目根目录维护:

```text
.agent-ledger/
  task-ledger.md
  decision-log.md
  plans/
    today-plan.md
    weekly-plan.md
  calendar/
    confirmed-timeblocks.json
    confirmed-timeblocks.ics
```

创建规则:

- 如果 `.agent-ledger/` 不存在, 先展示将创建的文件列表并请求用户确认。
- 如果 `task-ledger.md` 不存在, 使用 `assets/task-ledger-template.md` 创建。
- 如果 `decision-log.md` 不存在, 创建空日志模板。
- `plans/` 和 `calendar/` 可在首次需要时创建。

安全边界:

- Skill 默认只写 `.agent-ledger/`。
- 如用户明确要求导出到其他路径, 必须先展示目标路径。
- 不删除 `.agent-ledger/` 以外的文件。

## 5. Cross-session continuity

Codex skill 不能依赖模型自然记住上一个 session 的任务状态。新 session 若没有账本, 只能重新读取 README、docs、git log、git status 等项目材料来推断进度。该推断可能遗漏用户确认过的任务、计划取舍和下一步契约。

`agent-continuity-ledger` 的核心差异化是将这些状态写入 `.agent-ledger/task-ledger.md` 和 `.agent-ledger/decision-log.md`, 让新 session 能从固定文件恢复上下文。

### 5.1 Startup rule

当用户提出以下请求时:

- “上次做到哪里了?”
- “继续这个项目。”
- “根据任务账本安排今天。”
- “帮我看看当前进度。”
- “用 agent-continuity-ledger 接着做。”

Skill 必须按顺序执行:

1. 检查 `.agent-ledger/task-ledger.md` 是否存在。
2. 如果存在, 先读取 task ledger 和 decision log。
3. 再读取 README、spec、git log 或当前工作区状态作为补充上下文。
4. 输出当前状态摘要、下一步行动、阻塞项和需要确认的问题。
5. 如果 ledger 与 git/docs 推断冲突, 生成 ChangeSet 或澄清问题, 不直接覆盖 ledger。

### 5.2 Missing ledger rule

如果 `.agent-ledger/task-ledger.md` 不存在:

1. 明确说明没有发现跨 session 任务账本。
2. 说明当前只能基于仓库文件和 git 状态推断进度。
3. 提供创建 `.agent-ledger/` 的 ChangeSet。
4. 等待用户确认后再创建账本。

### 5.3 Continuity acceptance

跨 session 连续性通过以下测试验收:

1. Session A 创建 `.agent-ledger/task-ledger.md`, 写入至少 3 个任务、1 个阻塞项和 1 个下一步行动。
2. 关闭 Session A。
3. Session B 在同一 workspace 中启动。
4. 用户问“上次做到哪里了?”
5. Skill 必须读取 ledger, 而不是只重新扫描 README/git。
6. 输出必须包含上次确认任务、当前状态、阻塞项、下一步行动和是否需要更新计划。

## 6. `task-ledger.md` 规格

### 5.1 文件结构

`task-ledger.md` 是任务真相源。第一版使用 Markdown + fenced YAML task card。

模板:

```markdown
# Agent Task Ledger

Ledger version: 1
Timezone: Asia/Hong_Kong
Last reviewed: 2026-04-22

## Inbox

## Ready

## Active

## Waiting

## Scheduled

## Done

## Dropped
```

每个任务使用以下格式:

````markdown
### task-20260422-001 - Write skill spec

```yaml
id: task-20260422-001
title: Write skill spec
status: ready
created_at: 2026-04-22T16:55:00+08:00
updated_at: 2026-04-22T16:55:00+08:00
source:
  type: conversation
  quote: "按照需求写个spec吧"
  reference: null
user_confirmation: confirmed
priority: P1
risk_level: low
confidence: high
due: null
effort_minutes: 90
energy: deep_work
dependencies: []
blockers: []
next_action: Draft the skill spec document from the approved requirements.
acceptance_criteria:
  - Spec defines skill structure, ledger schema, ChangeSet protocol, planning rules, and .ics export interface.
scheduled:
  date: null
  start: null
  end: null
notes: []
```
````

任务正文可在 YAML block 后追加人类备注, 但核心字段必须保留在 YAML 中。

### 5.2 必填字段

每个 task 必须有:

- `id`
- `title`
- `status`
- `created_at`
- `updated_at`
- `source`
- `user_confirmation`
- `priority`
- `risk_level`
- `confidence`
- `next_action`
- `acceptance_criteria`

### 5.3 状态枚举

合法状态:

- `inbox`
- `ready`
- `active`
- `waiting`
- `scheduled`
- `done`
- `dropped`

状态语义:

- `inbox`: 捕获但未整理。
- `ready`: 可以开始。
- `active`: 正在做。
- `waiting`: 等待外部信息、用户输入或依赖。
- `scheduled`: 已确认时间块。
- `done`: 已完成。
- `dropped`: 明确不做。

### 5.4 用户确认枚举

合法值:

- `confirmed`
- `ai_inferred`
- `needs_user_confirmation`

规则:

- 用户明确说出的任务可标记为 `confirmed`。
- AI 推断出的任务必须标记为 `ai_inferred` 或 `needs_user_confirmation`。
- `ai_inferred` 任务不能进入 `.ics`。
- `needs_user_confirmation` 任务不能被标记为 `ready`, 除非用户确认。

### 5.5 风险等级

合法值:

- `low`
- `medium`
- `high`
- `blocked`

风险定义:

- `low`: 只读、整理、写本地账本或生成草稿。
- `medium`: 需要用户判断, 可能影响计划或承诺。
- `high`: 涉及外部写入、发送消息、删除文件、执行命令、修改代码或敏感信息。
- `blocked`: 不应由 skill 自动处理。

## 7. `decision-log.md` 规格

`decision-log.md` 是按时间追加的决策日志。

模板:

```markdown
# Agent Decision Log

## 2026-04-22

### 2026-04-22T17:00:00+08:00 - accept_changeset

```yaml
changeset_id: cs-20260422-001
user_decision: accepted_with_edits
affected_tasks:
  - task-20260422-001
summary: User accepted the skill spec task and asked to continue.
notes:
  - Keep .ics export optional.
```
```

必须记录:

- 时间。
- ChangeSet ID。
- 用户决策。
- 影响的 task ID。
- 摘要。
- 修改说明或保留意见。

日志只追加, 不重写历史。若记录有误, 追加 correction entry。

## 8. ChangeSet 规格

### 7.1 ChangeSet ID

格式:

```text
cs-YYYYMMDD-NNN
```

例如:

```text
cs-20260422-001
```

### 7.2 ChangeSet 输出格式

ChangeSet 必须先展示给用户, 不直接写文件。

标准格式:

```markdown
## ChangeSet cs-20260422-001

### Summary

- Proposed additions: 1
- Proposed updates: 0
- Proposed status changes: 0
- Proposed schedule blocks: 0
- Clarification questions: 1
- Blocked items: 0

### Proposed Additions

#### temp-task-001 - Write skill spec

```yaml
action: add_task
target: task-ledger.md
proposed_id: task-20260422-001
title: Write skill spec
source_quote: "按照需求写个spec吧"
user_confirmation: confirmed
priority: P1
risk_level: low
confidence: high
next_action: Draft the skill spec document from the approved requirements.
acceptance_criteria:
  - Spec defines skill structure, ledger schema, ChangeSet protocol, planning rules, and .ics export interface.
fields_requiring_confirmation: []
reviewer_decision: accept_draft
```

### Reviewer Findings

- All proposed tasks have a source quote.
- No high-risk action detected.

### User Decision Needed

Please confirm whether to apply this ChangeSet.
```

### 7.3 ChangeSet item actions

合法 action:

- `add_task`
- `update_task`
- `change_status`
- `add_note`
- `add_blocker`
- `remove_blocker`
- `schedule_timeblock`
- `unschedule_timeblock`
- `ask_clarification`
- `mark_manual_only`

禁止 action:

- `delete_task`
- `external_write`
- `send_message`
- `run_command`
- `delete_file`

如果用户要求删除任务, skill 应使用 `change_status: dropped`, 不物理删除。

### 7.4 应用 ChangeSet

应用前必须有用户确认。

可接受确认:

- “接受”
- “全部接受”
- “按这个改”
- “确认写入”
- “修改 X 后接受”

模糊回复不算确认, 例如:

- “看起来不错”
- “可以考虑”
- “差不多”

应用后必须:

- 更新 `task-ledger.md`。
- 追加 `decision-log.md`。
- 在回复中说明写入了哪些任务或计划。

## 9. Reviewer 规格

Reviewer 是每次 ChangeSet 输出前的自检阶段。

### 8.1 Reviewer checklist

必须检查:

- 每个任务是否有 `source_quote`。
- `source_quote` 是否真的支持任务。
- 是否把 AI 推断写成用户确认。
- `next_action` 是否是具体动作。
- `acceptance_criteria` 是否可验证。
- 是否重复已有任务。
- 是否缺少关键依赖。
- 是否含高风险动作。
- 是否错误进入日历或计划。
- 是否超出 skill 范围。

### 8.2 Reviewer decisions

合法 decision:

- `accept_draft`
- `needs_user_input`
- `revise_once`
- `manual_only`
- `blocked`

处理规则:

- `accept_draft`: 可展示给用户。
- `needs_user_input`: 不写账本, 先问用户。
- `revise_once`: AI 自行重写 ChangeSet 一次, 然后再次自检。
- `manual_only`: 可进入账本, 但不能进入计划或 `.ics`。
- `blocked`: 不写账本, 解释原因。

## 10. 计划生成规格

### 9.1 输入

计划生成输入:

- `task-ledger.md`
- 用户提供的可用时间。
- 用户提供的日期范围。
- 当前日期和时区。
- 可选偏好, 例如 deep work 放上午。

如果用户没有给出可用时间:

- 今日计划默认询问一次。
- 如果用户要求直接生成, 使用保守默认值: 2 小时 deep work + 1 小时 shallow work。
- 必须在计划中标注该假设。

### 9.2 排序规则

排序流程:

1. 排除 `done` 和 `dropped`。
2. 将 `waiting` 放到等待区。
3. 将 `needs_user_confirmation` 放到澄清区。
4. 优先硬截止日期。
5. 优先外部承诺。
6. 优先能解锁后续任务的任务。
7. 优先高影响、验收清楚、耗时可控的任务。
8. 对延期多次但仍重要的任务加权。
9. 降权高风险、低置信度、范围不清任务。
10. 每日计划只使用 60% 到 70% 可用时间。

### 9.3 `today-plan.md`

输出位置:

```text
.agent-ledger/plans/today-plan.md
```

模板:

```markdown
# Today Plan - 2026-04-22

Timezone: Asia/Hong_Kong
Plan status: draft
Capacity assumption: 180 minutes available, 120 minutes scheduled

## Outcomes

- Outcome 1

## Confirmed Time Blocks

| Time | Task | Reason | Confirmed |
| --- | --- | --- | --- |
| 09:30-10:30 | task-20260422-001 - Write skill spec | P1, unblocks implementation | no |

## Candidate Tasks

- task-20260422-001 - Write skill spec

## Not Scheduled

- task-20260422-002 - Waiting for user confirmation

## Blocked or Waiting

- None

## Questions

- Confirm whether 09:30-10:30 is acceptable.

## Assumptions

- User did not provide calendar availability.
```

`Confirmed Time Blocks` 只有在用户确认后才可写入 `confirmed-timeblocks.json`。

### 9.4 `weekly-plan.md`

输出位置:

```text
.agent-ledger/plans/weekly-plan.md
```

模板:

```markdown
# Weekly Plan - 2026-W17

Timezone: Asia/Hong_Kong
Plan status: draft

## Outcomes

1. Outcome A
2. Outcome B
3. Outcome C

## Daily Focus

| Day | Focus | Candidate tasks |
| --- | --- | --- |
| Monday | Planning | task-... |

## Key Sequence

1. task-...
2. task-...

## Waiting

- task-...

## Risks

- Risk A

## Excluded

- task-... because ...
```

## 11. Calendar export 规格

### 10.1 Export boundary

`.ics` export 只导出:

- 用户确认过的 focus time block。
- 用户确认过的 deadline event。

不导出:

- backlog。
- `inbox`、`waiting`、`dropped`、`done` 任务。
- `ai_inferred` 任务。
- `needs_user_confirmation` 任务。
- 高风险任务。
- 会议邀请。
- 参与人。
- RSVP。

### 10.2 JSON 输入

AI 在用户确认时间块后写入:

```text
.agent-ledger/calendar/confirmed-timeblocks.json
```

Schema:

```json
{
  "calendar_name": "Agent Confirmed Time Blocks",
  "timezone": "Asia/Hong_Kong",
  "generated_at": "2026-04-22T17:30:00+08:00",
  "events": [
    {
      "uid": "task-20260422-001-20260423T093000@agent-continuity-ledger.local",
      "task_id": "task-20260422-001",
      "type": "timeblock",
      "title": "Write skill spec",
      "description": "Task: task-20260422-001\nSource: User requested a spec from requirements.\nAcceptance: Spec is written and reviewed.\nRisk: low",
      "start": "2026-04-23T09:30:00+08:00",
      "end": "2026-04-23T10:30:00+08:00",
      "all_day": false
    }
  ]
}
```

Deadline event:

```json
{
  "uid": "task-20260422-001-deadline-20260424@agent-continuity-ledger.local",
  "task_id": "task-20260422-001",
  "type": "deadline",
  "title": "Deadline: Write skill spec",
  "description": "Confirmed deadline for task-20260422-001",
  "date": "2026-04-24",
  "all_day": true
}
```

### 10.3 `export_ics.py` CLI

Command:

```powershell
python agent-continuity-ledger/scripts/export_ics.py `
  --input .agent-ledger/calendar/confirmed-timeblocks.json `
  --output .agent-ledger/calendar/confirmed-timeblocks.ics
```

Requirements:

- Use Python standard library only.
- Read UTF-8 JSON.
- Write UTF-8 `.ics`.
- Use CRLF line endings.
- Escape commas, semicolons, backslashes and newlines in text fields.
- Fold long iCalendar lines.
- Preserve provided `uid`.
- Generate `DTSTAMP` at export time in UTC.
- For timed events, emit `DTSTART` and `DTEND`.
- For all-day events, emit date-only `DTSTART` and exclusive date-only `DTEND`.
- Include `PRODID:-//agent-continuity-ledger//EN`.
- Do not emit `VTODO`, `ATTENDEE`, `ORGANIZER`, `VALARM`, `RRULE`, or `METHOD`.

### 10.4 Output path

Default output:

```text
.agent-ledger/calendar/confirmed-timeblocks.ics
```

The skill response after export must say:

- `.ics` is a one-way export.
- Importing may create a snapshot, not live sync.
- Use a separate calendar for import if possible.
- Re-importing may duplicate events depending on calendar client behavior.

## 12. Assets 规格

### 11.1 `task-ledger-template.md`

Must include:

- Header.
- Ledger version.
- Timezone placeholder.
- Last reviewed.
- Seven status sections.
- One commented example task.

### 11.2 `today-plan-template.md`

Must include:

- Date.
- Timezone.
- Plan status.
- Capacity assumption.
- Outcomes.
- Confirmed time blocks.
- Candidate tasks.
- Not scheduled.
- Blocked or waiting.
- Questions.
- Assumptions.

### 11.3 `weekly-plan-template.md`

Must include:

- ISO week.
- Timezone.
- Plan status.
- Outcomes.
- Daily focus.
- Key sequence.
- Waiting.
- Risks.
- Excluded.

## 13. `agents/openai.yaml` 规格

Recommended UI metadata:

```yaml
display_name: Agent Continuity Ledger
short_description: Keep AI task handoffs, task changes, plans, and optional calendar exports consistent across sessions.
default_prompt: Use this skill to capture tasks from our discussion, propose a reviewed ChangeSet, update the local task ledger after confirmation, and generate a daily or weekly plan.
```

Actual `openai.yaml` should be generated with the skill creator tooling if available.

## 14. Error handling

### 13.1 Missing ledger

If `.agent-ledger/task-ledger.md` is missing:

1. Explain that no ledger exists.
2. Offer to create `.agent-ledger/` from templates.
3. Wait for confirmation before writing files.

### 13.2 Malformed ledger

If `task-ledger.md` is malformed:

1. Do not overwrite it.
2. Produce a repair ChangeSet.
3. Ask user to confirm repair.
4. Preserve original content unless user explicitly asks to replace it.

### 13.3 Ambiguous user instruction

If user asks for plan but gives no available time:

- Ask once for available time when feasible.
- If user wants speed, proceed with conservative assumptions and label them clearly.

### 13.4 High-risk request

If a task implies external write, command execution, deletion, sensitive data, or irreversible action:

- Do not execute.
- Create a manual-only task or draft.
- Mark `risk_level: high` or `blocked`.
- Ask for explicit confirmation before any future external action.

## 15. Acceptance criteria

The skill is acceptable when:

- `SKILL.md` frontmatter triggers on task memory, planning, and calendar export requests.
- `SKILL.md` instructs Codex to read/write only `.agent-ledger/` by default.
- A new ledger can be created from templates after user confirmation.
- Existing ledger tasks can be read and summarized.
- In a new session, existing ledger tasks are read before README/git inference when the user asks for current progress.
- New user input produces a ChangeSet before file edits.
- ChangeSet includes source quote, confirmation state, next action and acceptance criteria.
- Reviewer checklist runs before showing ChangeSet.
- User-confirmed ChangeSet updates `task-ledger.md`.
- Applying ChangeSet appends `decision-log.md`.
- Today plan can be generated from ledger with transparent sorting reasons.
- Weekly plan can be generated from ledger with outcomes and daily focus.
- `.ics` export only uses `confirmed-timeblocks.json`.
- `export_ics.py` produces a valid basic VEVENT calendar from sample JSON.
- No unconfirmed task is exported to `.ics`.
- No external write, message send, command execution or deletion is performed by default.

## 16. Test cases

### 16.1 Ledger creation

Input:

```text
帮我开始维护这个项目的任务账本。
```

Expected:

- Skill explains it will create `.agent-ledger/`.
- Skill waits for confirmation.
- After confirmation, creates `task-ledger.md` and `decision-log.md`.

### 16.2 Capture task

Input:

```text
记一下: 明天要把 skill spec 写完, 并确认 .ics 只作为导出。
```

Expected:

- ChangeSet proposes one task.
- Source quote preserved.
- User confirmation state is `confirmed`.
- `.ics` boundary appears in acceptance criteria.

### 16.3 AI inferred task

Input:

```text
我们可能还需要考虑 Todoist 导出。
```

Expected:

- Task is marked `ai_inferred` or `needs_user_confirmation`.
- It does not enter today plan automatically.
- It does not enter `.ics`.

### 16.4 Today plan

Input:

```text
我今天有 3 小时, 帮我排一下。
```

Expected:

- Uses no more than 120 minutes to 130 minutes by default.
- Produces at most 3 focus items.
- Explains unscheduled tasks.

### 16.5 ICS export

Input:

```text
确认把 9:30-10:30 的 spec 写作时间块导出日历。
```

Expected:

- Creates or updates `confirmed-timeblocks.json`.
- Runs or instructs running `export_ics.py`.
- Exports only that confirmed block.
- Responds with import/sync caveat.

### 16.6 Cross-session recovery

Session A input:

```text
使用 agent-continuity-ledger, 记录三个任务: 写 skill spec、实现 skill、测试跨 session 恢复。实现 skill 依赖 spec 完成。
```

Expected Session A:

- Creates or updates `.agent-ledger/task-ledger.md` after confirmation.
- Records dependency and next action.
- Appends `decision-log.md`.

Session B input in the same workspace:

```text
使用 agent-continuity-ledger, 上次做到哪里了?
```

Expected Session B:

- Reads `.agent-ledger/task-ledger.md` first.
- Summarizes the three tasks and dependency.
- Identifies the current next action from ledger.
- Mentions any mismatch with git/docs only as supplemental context.

## 17. Two-week dogfood protocol

Before building app, scheduler or connectors:

1. Use the skill for 10 real sessions.
2. Track accepted vs rejected ChangeSet items.
3. Track manual edits per accepted task.
4. Track whether plans are followed.
5. Track whether `.ics` exports help or create clutter.
6. Track whether ledger maintenance feels burdensome.

Pass criteria:

- Accepted ChangeSet item rate >= 60%.
- Average manual edit per accepted task <= 30%.
- User can answer “上次做到哪里了” from ledger within 2 minutes.
- A fresh Codex session can recover task state from `.agent-ledger/` without relying on prior chat history.
- At least 7 of 10 sessions produce useful continuity.
- `.ics` exports do not create duplicate or wrong-time events in basic usage.

Fail criteria:

- User stops checking ledger.
- Plans are mostly ignored.
- Ledger diverges from real work.
- `.ics` creates clutter.
- Direct Codex prompting is faster with similar quality.

## 18. Implementation order

Implementation should proceed in this order:

1. Create skill scaffold.
2. Write concise `SKILL.md`.
3. Add reference docs.
4. Add templates.
5. Add `export_ics.py`.
6. Validate skill frontmatter and folder naming.
7. Run sample ledger creation workflow.
8. Run sample ChangeSet workflow.
9. Run sample today plan workflow.
10. Run sample `.ics` export workflow.

Do not implement external connectors before dogfood passes.

## 19. Open decisions resolved by this spec

- Ledger format: Markdown source with fenced YAML task cards.
- Default ledger path: current workspace `.agent-ledger/`.
- Calendar export: explicit only, never default.
- `.ics` content: `VEVENT` only, no `VTODO`.
- Script input: structured JSON, not free-form Markdown parsing.
- First version scope: manual ledger, plans and optional export only.
- Cross-session memory: explicit file-backed ledger, not model memory.

## 20. Remaining open questions

- Should the skill live in the repository for versioning, or in `$CODEX_HOME/skills` for immediate local discovery?
- Should `validate_ledger.py` be included in V1 if the Markdown YAML format proves fragile?
- Should dogfood metrics be stored in `decision-log.md` or a separate `.agent-ledger/metrics.md`?
- Should there be one ledger per repository, or a global personal ledger that links to repositories?
