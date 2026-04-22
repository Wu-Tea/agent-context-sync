# Agent Continuity Ledger Skill - 需求说明

日期: 2026-04-22
状态: 需求说明草案
目标形态: Codex skill
推荐 skill 名称: `agent-continuity-ledger`

## 1. 背景

最初设想是做一个类似 Trello 的本地 AI 任务应用, 能从资料中自动拆解任务、整理优先级、定制日程并自动执行。经过评审后, 这个方向被认为过重, 且容易与现有任务管理、日历、自动化和 AI agent 产品重叠。

新的收敛方向是先做一个 skill, 不急着做完整应用。这个 skill 不把重点放在“AI 会拆任务”上, 因为 Codex、ChatGPT、Notion AI 等工具本身已经能做到任务拆分。它要解决的是更窄但更有差异化的问题:

> 让 AI 在跨会话、跨任务的工作中不失忆, 并以可审查、可追溯的方式维护任务账本、执行顺序和日程投影。

因此, 这个 skill 的核心不是自动执行, 也不是替代 Todoist、Trello、Notion、Google Calendar 或 Apple Reminders。它的核心是建立一个本地的 AI 工作连续性协议:

- 任务从哪里来。
- 哪些任务被用户确认过。
- 哪些任务只是 AI 推断。
- 下一步行动是什么。
- 为什么今天应该做这些任务。
- 计划变化的依据是什么。
- 哪些时间块可以导出到日历。

## 2. 产品定位

`agent-continuity-ledger` 是一个 Codex skill, 用于维护本地任务连续性账本。

一句话定位:

> 把用户与 AI 讨论出的工作事项编译为可审查的任务变更集、长期任务账本、每日/每周计划和可选日历投影。

它不是:

- 任务管理 SaaS。
- 看板应用。
- 双向日历同步工具。
- 后台自动执行器。
- 通用个人日程管理器。
- 外部系统写入自动化工具。

它是:

- AI 工作交接协议。
- 本地任务记忆账本。
- 任务变更审阅流程。
- 每日/每周计划生成器。
- 已确认时间块的 `.ics` 导出器。

## 3. 目标用户和使用场景

### 3.1 目标用户

第一版目标用户是频繁使用 Codex 或类似 AI agent 进行项目推进、研究、写作、开发、运营或知识工作的个人用户。

用户特征:

- 经常和 AI 讨论任务、方案和执行顺序。
- 任务来源分散在聊天、文档、代码、会议记录或临时想法中。
- 不希望马上迁移到新的完整任务管理系统。
- 希望 AI 能在下次会话中知道上次说到哪里。
- 希望计划进入 Markdown 或日历, 但不希望 AI 擅自承诺时间。

### 3.2 典型触发语句

Skill 应在类似请求中触发:

- “帮我记一下这些任务。”
- “把刚才讨论的事情整理进任务账本。”
- “根据任务账本安排一下今天。”
- “生成本周计划。”
- “看看哪些任务应该先做。”
- “把确认过的计划导出成日历。”
- “更新一下这个项目的下一步。”
- “上次我们做到哪里了?”
- “帮我维护一个跨会话的任务列表。”
- “把这些新增需求变成可审查的任务变更。”

## 4. 核心目标

第一版必须达成:

- 维护本地 `task-ledger.md` 作为任务账本源。
- 从用户输入中提取新增任务、任务更新、阻塞项和澄清问题。
- 在修改账本前生成 `ChangeSet`, 让用户确认。
- 区分用户确认内容和 AI 推断内容。
- 为每个任务保留来源、下一步行动和验收标准。
- 根据任务账本生成 `today-plan.md` 和 `weekly-plan.md`。
- 只将用户确认过的时间块导出为 `.ics`。
- 明确 `.ics` 只是日历投影, 不是任务真相源。
- 不执行外部写入、命令运行、消息发送、删除文件等高风险动作。

## 5. 非目标

第一版不做:

- 完整 app UI。
- SQLite 数据库。
- 看板拖拽。
- 多用户协作。
- 团队权限。
- CalDAV。
- Google / Microsoft / Apple OAuth。
- 双向日历同步。
- 后台 scheduler。
- 自动提醒 daemon。
- 自动执行任务。
- 外部 SaaS 写入。
- 自动创建 GitHub issue、Trello card、Notion task。
- 复杂依赖图、甘特图或资源负载计算。
- 自然语言 recurring task 解析。
- `.ics` 订阅服务器或 `webcal://` feed。
- 把未确认任务自动放入日历。

这些能力可以作为后续版本考虑, 但不进入第一版。

## 6. Skill 形态

推荐目录结构:

```text
agent-continuity-ledger/
  SKILL.md
  scripts/
    export_ics.py
    validate_ledger.py
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

第一版可以先实现:

- `SKILL.md`
- `references/task-ledger-schema.md`
- `references/changeset-protocol.md`
- `references/planning-rules.md`
- `references/calendar-export-rules.md`
- `assets/task-ledger-template.md`
- `assets/today-plan-template.md`
- `assets/weekly-plan-template.md`
- `scripts/export_ics.py`

`validate_ledger.py` 可以作为第二阶段实现。

## 7. 工作区文件契约

Skill 运行时应在当前项目或用户指定目录下维护一个任务工作区。默认建议:

```text
.agent-ledger/
  task-ledger.md
  decision-log.md
  plans/
    today-plan.md
    weekly-plan.md
  calendar/
    confirmed-timeblocks.ics
```

### 7.1 `task-ledger.md`

任务账本源文件。它是任务状态的主要来源。

必须包含:

- 任务 ID。
- 任务标题。
- 状态。
- 来源。
- 用户确认状态。
- 下一步行动。
- 验收标准。
- 优先级。
- 风险等级。
- 截止日期。
- 预估耗时。
- 阻塞项。
- 最近更新时间。

推荐状态:

- `inbox`: 已捕获, 未整理。
- `ready`: 可执行。
- `active`: 正在推进。
- `waiting`: 等待外部输入或依赖。
- `scheduled`: 已安排时间块。
- `done`: 已完成。
- `dropped`: 明确放弃。

### 7.2 `decision-log.md`

记录用户对 AI 建议的决策。

必须记录:

- 时间。
- 决策类型。
- 涉及任务 ID。
- AI 建议摘要。
- 用户选择。
- 修改说明。

典型决策:

- 接受新增任务。
- 修改后接受。
- 拒绝任务。
- 推迟任务。
- 标记完成。
- 导出时间块到日历。

### 7.3 `today-plan.md`

当天计划输出。

必须包含:

- 今日目标。
- 已确认时间块。
- 候选任务。
- 不安排的任务及原因。
- 阻塞项。
- 需要用户确认的问题。
- 计划生成假设。

### 7.4 `weekly-plan.md`

本周计划输出。

必须包含:

- 本周 1 到 3 个 outcomes。
- 每日 focus。
- 关键任务顺序。
- 等待项。
- 风险。
- 被排除任务及原因。

### 7.5 `confirmed-timeblocks.ics`

可选日历导出文件。

只允许包含:

- 用户确认过的时间块。
- 用户确认过的重要截止日期。

不允许包含:

- 未确认 backlog。
- 阻塞任务。
- AI 推断但用户未确认的任务。
- 外部会议邀请。
- 参会人。
- RSVP。
- 自动提醒, 除非用户明确要求。

## 8. ChangeSet 协议

Skill 不应直接修改 `task-ledger.md`。任何账本变更前必须先生成 ChangeSet。

ChangeSet 应包含:

- `proposed_additions`: 建议新增任务。
- `proposed_updates`: 建议更新任务。
- `proposed_status_changes`: 建议状态变化。
- `proposed_schedule_blocks`: 建议时间块。
- `clarification_questions`: 需要用户补充的问题。
- `blocked_items`: 不应直接写入的项目。
- `risk_notes`: 风险说明。

每个 ChangeSet item 必须包含:

- 建议动作。
- 任务 ID 或新任务临时 ID。
- 标题。
- 描述。
- 来源证据。
- AI 推断字段。
- 用户需要确认的字段。
- 风险等级。
- 接受后的写入位置。

用户决策方式:

- 全部接受。
- 逐条接受。
- 修改后接受。
- 拒绝。
- 稍后处理。

只有用户确认后, Skill 才能使用 `apply_patch` 或等价安全编辑方式更新账本文件。

## 9. Reviewer 规则

Reviewer 在本 skill 中是规则化审查流程, 不一定是独立 agent。它的职责是帮助 AI 在输出 ChangeSet 前做自检。

Reviewer 必须检查:

- 任务是否有来源。
- 来源是否足以支持任务。
- 任务是否可执行。
- 下一步行动是否明确。
- 验收标准是否清楚。
- 是否存在重复任务。
- 是否缺少截止日期、范围或依赖。
- 是否把 AI 推断误写成用户确认事实。
- 是否包含高风险动作。
- 是否不应进入日历。

Reviewer 输出建议:

- `accept_draft`: 可展示给用户确认。
- `needs_user_input`: 需要用户补充信息。
- `revise_once`: 需要 AI 重写一次。
- `manual_only`: 只能作为人工处理项, 不进入自动计划或日历。
- `blocked`: 不应写入账本。

## 10. 排序和计划规则

排序必须透明, 不做黑盒优化。

计划生成流程:

1. 排除 `done`、`dropped` 和明显阻塞任务。
2. 将 `waiting` 任务放入等待区, 不安排执行时间块。
3. 优先处理有硬截止日期或外部承诺的任务。
4. 优先处理能解锁多个后续任务的任务。
5. 优先处理高影响且验收标准清楚的任务。
6. 将高风险、低置信度、范围不清的任务降权。
7. 对延期多次但仍重要的任务提高权重。
8. 每日计划默认只占用用户可用时间的 60% 到 70%。
9. 每天建议最多 3 个主要 focus。
10. 每个时间块必须说明排序依据。

计划输出必须解释:

- 为什么这些任务被安排。
- 为什么某些任务没有安排。
- 哪些任务需要用户确认。
- 哪些假设可能导致计划变化。

## 11. Calendar 和 `.ics` 要求

`.ics` 是可选输出, 不是核心账本。

第一版只生成 `VEVENT`, 不生成 `VTODO`。

原因:

- 主流日历客户端对事件支持更一致。
- 待办任务语义在不同客户端中差异较大。
- 任务状态、来源、验收标准和依赖不适合依赖 `.ics` 表达。

导出规则:

- 只导出用户确认过的时间块。
- 每个事件必须有稳定 `UID`。
- 每个事件必须有 `DTSTAMP`。
- 使用明确时区。
- 全天截止日期使用全天 `VEVENT`。
- `DESCRIPTION` 包含任务 ID、来源摘要、验收标准和风险提示。
- 默认不生成提醒。
- 不生成参会人、会议链接、RSVP 或外部邀请。
- 重新导出时应尽量保持同一任务时间块的 `UID` 稳定, 避免重复导入。

用户提示:

- 导入 `.ics` 通常是快照, 不保证同步。
- 订阅 URL 才更像持续更新, 但第一版不提供订阅服务。
- 建议用户导入到单独 calendar, 方便删除和重建。
- 日历显示不代表任务已完成。

## 12. 脚本需求

### 12.1 `export_ics.py`

职责:

- 从确认后的 plan/timeblocks 生成 `.ics`。
- 保证基础 iCalendar 格式正确。
- 使用稳定 UID。
- 处理转义和 CRLF 换行。
- 输出到 `.agent-ledger/calendar/confirmed-timeblocks.ics`。

输入:

- 结构化 timeblocks JSON 或 Markdown 中的确认时间块。

输出:

- `.ics` 文件。

不负责:

- 决定任务排序。
- 读取用户日历空闲时间。
- 连接 Google/Apple/Microsoft 日历。
- 自动同步。

### 12.2 `validate_ledger.py`

第二阶段脚本。

职责:

- 检查 `task-ledger.md` 是否符合 schema。
- 检查任务 ID 是否重复。
- 检查状态值是否合法。
- 检查必要字段是否缺失。
- 检查计划中引用的任务 ID 是否存在。

## 13. 安全和权限

Skill 默认只操作当前工作区内 `.agent-ledger/` 目录。

禁止:

- 删除用户文件。
- 覆盖外部文件。
- 执行 shell 命令完成任务本身。
- 发送消息。
- 写入外部 SaaS。
- 自动创建外部 issue/card/task。
- 未经确认标记任务完成。
- 未经确认导出 `.ics`。

允许:

- 读取 `.agent-ledger/` 文件。
- 创建 `.agent-ledger/` 模板文件。
- 生成 ChangeSet。
- 在用户确认后更新任务账本。
- 在用户确认后生成 Markdown 计划。
- 在用户确认后生成 `.ics`。

## 14. 成功指标

第一版不以功能数量为成功标准, 而以连续使用和任务质量为标准。

两周 dogfood 指标:

- 至少连续使用 10 次。
- AI 生成任务建议接受率达到 60% 以上。
- 平均每条被接受任务的人工修改量低于 30%。
- 用户能通过账本回答“上次做到哪里了”。
- 用户认为计划解释足够清楚。
- `.ics` 导出没有产生重复事件或明显时间错误。
- 用户没有因为账本维护感到额外负担。

如果这些指标不成立, 不应继续扩展 app、scheduler 或外部连接器。

## 15. 版本路线

### V0 - Requirements and Skill Draft

- 完成本需求说明。
- 编写 skill 设计文档。
- 编写最小 `SKILL.md`。
- 定义 `task-ledger.md` schema。

### V1 - Manual Ledger and Plans

- 支持读取/创建 `.agent-ledger/`。
- 支持 ChangeSet。
- 支持用户确认后更新账本。
- 支持生成 `today-plan.md` 和 `weekly-plan.md`。

### V1.1 - Calendar Export

- 增加 `export_ics.py`。
- 只导出确认时间块。
- 加入 `.ics` 基础验证。

### V1.2 - Ledger Validation

- 增加 `validate_ledger.py`。
- 检查任务账本一致性。
- 检查计划引用一致性。

### V2 - External Export

- 导出 Markdown、JSON、CSV。
- 可选导出到 Todoist、Notion、Trello 或 GitHub Issues 的 patch 文件。
- 仍不自动写入外部系统。

### V3 - Automation

- 在任务账本验证稳定后, 再考虑 heartbeat 或 scheduler。
- 自动生成日报/周报草稿。
- 外部写入仍需人工确认。

## 16. 开放问题

- 第一版账本应使用 Markdown 还是 YAML。
- `.agent-ledger/` 应默认放在项目根目录, 还是用户主目录。
- 是否需要支持多个 project ledger。
- 是否默认生成 `.ics`, 还是只有用户显式要求才生成。
- `source_quote` 对于口头讨论应如何记录。
- 是否需要把本需求拆成 skill 设计文档和实现计划。

## 17. 设计结论

推荐先做 `agent-continuity-ledger` skill, 而不是 app。

第一版的价值判断是:

- Codex 已经能拆任务, 所以 skill 不应只做任务拆分。
- 市面已有大量任务和日历产品, 所以 skill 不应替代它们。
- 真正缺口是 AI 工作连续性、任务来源证据、变更审阅和下一步契约。
- `.ics` 有用, 但只能作为已确认计划的日历投影。
- 任务账本必须是源, 日历和计划都是派生物。

如果用户连续两周愿意使用这个 skill 维护真实任务, 才值得继续做 app、scheduler、connector 或更强自动化。
