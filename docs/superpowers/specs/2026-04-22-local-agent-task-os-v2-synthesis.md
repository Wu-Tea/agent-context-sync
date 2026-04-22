# Local Agent Task OS - 三方评审后的新方案

日期: 2026-04-22
状态: V2 综合设计草案
前置文档: `docs/superpowers/specs/2026-04-22-local-agent-task-os-design.md`

## 1. 结论

三方评审后, 推荐将原方案调整为:

> Review-first Local Agent Inbox: 一个本地优先的 AI 任务审阅与变更集应用。第一版不直接追求完整任务 OS 和后台自动执行, 而是先验证 AI 能否稳定把资料转化为用户愿意采纳的任务变更。自动化能力按成熟度逐步解锁。

这不是放弃原来的 `本地应用 + Agent 后台服务 + Reviewer + Scheduler + Executor` 方向, 而是重新排序:

- 先证明任务拆解和 Reviewer 有价值。
- 再证明计划建议有价值。
- 再加入定时唤醒。
- 最后才开放低风险自动执行和外部系统写入。

原方案更适合作为 V1.5/V2 架构蓝图。新的 MVP 应该更窄, 更容易验证, 更少依赖用户迁移现有工作流。

## 2. 三方评审摘要

### 2.1 支持方观点

支持方认为原方案可行, 核心理由是:

- 本地优先降低隐私和部署门槛。
- SQLite 适合本地任务库和 agent run 记录。
- L0/L1 风险分级和 Approval Gate 可以让自动执行保持可控。
- 独立 Reviewer 能让系统不只是一个会定时运行的聊天机器人。
- sources、artifacts、reviews、approvals、audit_logs 等模型为未来云端化留下基础。

支持方建议补强:

- 建立 MVP 评测集。
- 明确 Scheduler 和 Executor 状态机。
- 为 executor 增加能力声明, 例如权限、风险、是否支持 dry run、是否有副作用。

### 2.2 轻微反对方观点

轻微反对方认可长期方向, 但认为当前 MVP 太宽:

- 同时验证任务生成、排期、执行、审查、本地任务 OS, 失败后难定位原因。
- 页面和模块过多, 用户心智不清楚。
- 自动执行过早, 可能放大任务拆解噪音。
- Reviewer 设计正确但第一版不必贯穿三阶段。
- 文件输入格式过多, 会让解析工作喧宾夺主。

轻微反对方建议最小 MVP:

```text
Inbox -> Task Brain -> Minimal Reviewer -> User Accept -> Local Store/Export
```

### 2.3 强烈反对方观点

强烈反对方认为原方案最大问题是入口可能错了:

- 用户已有 Asana、Notion、Trello、ClickUp、Jira、Slack、Google Workspace 等工作流。
- 新建本地任务库会制造第二个真相源。
- 自动执行的低风险部分可能商业价值不足。
- LLM Reviewer 可能制造虚假安全感。
- 本地 scheduler 不是强卖点, 还会遇到休眠、崩溃、权限、补偿等问题。
- 云端迁移不是靠预留字段就能完成。

强烈反对方建议替代路线:

> 先做现有项目管理工具上的 AI 审阅层, 生成可审阅的任务变更集, 用户批准后再应用。

## 3. 新方案定位

新方案把产品从“自动执行任务 OS”调整为“AI 任务变更集审阅器 + 本地任务代理底座”。

第一版核心价值:

- 输入混乱资料。
- 生成结构化任务变更集。
- Reviewer 检查任务是否可执行、是否有来源、是否缺上下文、是否有风险。
- 用户像审代码 diff 一样审任务 diff。
- 批准后写入本地任务库或导出到现有工具。

产品心智:

```text
不是让用户立刻迁移到新任务系统,
而是先成为用户已有资料和任务系统之间的 AI 审阅层。
```

## 4. 新 MVP 范围

### 4.1 包含

- 本地 Web app。
- 文本粘贴输入。
- Markdown/TXT 文件输入。
- 会议纪要和需求文档场景优先。
- AI 生成摘要、任务草稿、缺失问题和建议变更集。
- Minimal Reviewer 审查任务草稿。
- 用户逐条接受、修改、拒绝任务草稿。
- 本地 SQLite 保存 sources、drafts、tasks、reviews、decisions。
- Markdown/JSON 导出。
- 简单 Trello-like 看板视图作为本地展示, 但不做完整项目管理系统。
- 手动生成今日建议, 不做后台自动排期。

### 4.2 不包含

- 后台自动执行任务。
- 独立 scheduler。
- 外部系统自动写入。
- PDF、Word、Excel、CSV 的完整解析。
- 团队协作。
- 多租户。
- 完整 Audit UI。
- L3/L4 风险任务执行。
- 代码、shell、浏览器自动化。

### 4.3 可以预留但不实现

- MCP connector。
- Trello/Notion/飞书 connector。
- Scheduler 接口。
- Executor 插件接口。
- Cloud-ready workspace/user 字段。
- 外部写入 approval 流程。

## 5. 自动化成熟度阶梯

自动化不在第一天全部打开, 而是分级验证。

`E0 - AI 建议`

- 系统只生成任务建议、摘要和问题。
- 用户手动复制或确认。
- MVP 必须支持。

`E1 - 用户批准后应用`

- 用户逐条批准任务变更。
- 系统写入本地任务库。
- 可导出 Markdown/JSON。
- MVP 必须支持。

`E2 - 手动计划`

- 用户点击生成今日计划。
- 系统根据任务、优先级、截止日期和依赖生成建议。
- 不后台自动运行。
- MVP 可做轻量版本。

`E3 - 定时建议`

- 本地 scheduler 定时生成计划、提醒、日报草稿。
- 不自动修改外部系统。
- V1 阶段实现。

`E4 - 低风险自动执行`

- 自动总结资料、整理任务描述、生成报告草稿。
- 执行后 Reviewer 检查。
- V1.5 阶段实现。

`E5 - 外部工具执行`

- 调用 MCP、飞书、Notion、Trello、Web search、浏览器或代码工具。
- 默认人工确认。
- V2 阶段实现。

这个阶梯解决两个问题:

- 不否定自动执行愿景。
- 避免在还没验证任务质量前过早引入 runner。

## 6. Reviewer 重新定义

新方案中 Reviewer 不是“AI 安全门”的唯一来源, 而是“质检信号 + 规则检查 + 人工审阅辅助”。

### 6.1 MVP Reviewer 职责

Minimal Reviewer 只审查任务草稿:

- 任务是否有来源依据。
- 任务是否可执行。
- 任务是否缺少上下文。
- 验收标准是否清楚。
- 是否疑似重复。
- 是否包含外部写入、发送消息、删除文件、执行命令等高风险动作。
- 是否需要用户补充信息。

### 6.2 MVP Reviewer 输出

只保留四种决策:

- `accept_draft`: 可以展示给用户接受。
- `needs_user_input`: 需要用户补充信息。
- `revise_once`: 退回 Task Brain 重写一次。
- `manual_only`: 不允许自动应用, 只能作为人工任务或草稿。

### 6.3 Reviewer 不能做的事

- 不能直接执行任务。
- 不能批准外部写入。
- 不能把任务标记为已完成。
- 不能替代人工判断。
- 不能作为唯一安全证明。

## 7. 变更集模型

新方案引入 `ChangeSet`, 这是比直接写任务库更稳的中间层。

流程:

```text
Source
  -> Task Brain
  -> ChangeSet Draft
  -> Minimal Reviewer
  -> User Review
  -> Apply to Local Task Store or Export
```

ChangeSet 包含:

- 待创建任务。
- 待更新任务。
- 待补充验收标准。
- 待设置依赖。
- 待标记风险。
- 待询问用户的问题。
- 被 Reviewer 拦截的项目。

用户看到的是 diff:

- 新增什么任务。
- 为什么新增。
- 来源证据在哪里。
- 风险等级是什么。
- 哪些内容需要确认。

## 8. 简化数据模型

MVP 只需要以下表:

### 8.1 sources

- `id`
- `workspace_id`
- `type`
- `title`
- `content`
- `content_hash`
- `created_at`

### 8.2 change_sets

- `id`
- `workspace_id`
- `source_id`
- `status`
- `summary`
- `created_at`
- `updated_at`

### 8.3 change_items

- `id`
- `workspace_id`
- `change_set_id`
- `item_type`
- `title`
- `description`
- `source_quote`
- `priority`
- `risk_level`
- `acceptance_criteria`
- `review_decision`
- `user_decision`
- `metadata_json`
- `created_at`

### 8.4 tasks

- `id`
- `workspace_id`
- `parent_task_id`
- `source_id`
- `title`
- `description`
- `status`
- `priority`
- `risk_level`
- `acceptance_criteria`
- `created_at`
- `updated_at`

### 8.5 reviews

- `id`
- `workspace_id`
- `target_type`
- `target_id`
- `decision`
- `findings_json`
- `created_at`

### 8.6 agent_runs

- `id`
- `workspace_id`
- `agent_type`
- `input_json`
- `output_json`
- `status`
- `started_at`
- `finished_at`

说明:

- `workspace_id` 保留, 但 MVP 只有默认 workspace。
- 暂不做完整 `approvals` 和 `audit_logs`, 先用 `agent_runs`、`reviews`、`change_items.user_decision` 覆盖基本追踪。
- 后续进入 E3/E4/E5 后再增加 `plans`、`plan_items`、`jobs`、`approvals`、`audit_logs`。

## 9. MVP 页面

### 9.1 Inbox

用户粘贴文本或上传 Markdown/TXT 文件。

显示:

- 原始资料。
- 提取摘要。
- 生成 ChangeSet 按钮。
- 历史输入。

### 9.2 Review

核心页面, 类似任务 diff 审阅。

每个 change item 显示:

- 建议动作。
- 任务标题。
- 描述。
- 来源引用。
- 风险等级。
- Reviewer 发现。
- 接受、修改、拒绝、稍后处理。

### 9.3 Tasks

展示已接受任务。

第一版只需要:

- Inbox / Todo / Doing / Done 四列。
- 任务详情。
- 来源链接。
- 风险等级和验收标准。

### 9.4 Today

手动生成今日建议。

第一版不自动后台运行, 只提供:

- 选择今日可用时间。
- 生成建议排序。
- 用户接受后形成今日列表。

## 10. 技术路线

MVP 推荐采用:

- Frontend: React + TypeScript + Vite。
- Backend: Node.js 或 Python FastAPI 二选一。
- Database: SQLite。
- AI: OpenAI-compatible ModelProvider 接口。
- File input: 只处理纯文本和 Markdown/TXT。
- Packaging: 先本地 Web app, 后续再封装桌面应用。

为了减少决策面, 建议第一版先选单体:

```text
app/
  frontend/
  backend/
  storage/
    app.db
    sources/
    exports/
```

## 11. 成功指标

MVP 不以“自动执行了多少任务”为核心指标, 而以“任务建议是否被用户采纳”为核心指标。

建议指标:

- AI 生成任务草稿的用户接受率。
- 每条任务平均人工修改量。
- Reviewer 拦截率。
- Reviewer 拦截后用户认同率。
- 每份资料平均节省时间。
- 一周内重复使用次数。
- 从资料输入到可接受任务列表的耗时。
- 用户最终是否愿意把任务导出或继续管理。

## 12. 后续阶段

### 12.1 V1 - Local Planner

- 手动今日计划。
- 依赖关系。
- 截止日期。
- 简单工作量估算。
- Planner Reviewer。

### 12.2 V1.5 - Scheduled Suggestions

- 本地 scheduler。
- 每日计划自动生成。
- 日报/周报草稿。
- missed-run 补偿。
- 健康摘要。

### 12.3 V2 - Low-risk Executor

- 自动总结资料。
- 自动整理任务描述。
- 自动生成报告草稿。
- 执行后 Reviewer。
- L0/L1 自动运行。

### 12.4 V3 - Connectors and MCP

- Trello/Notion/飞书 connector。
- Web search。
- MCP tools。
- 外部写入 approval。
- Tool capability manifest。

### 12.5 V4 - Cloud and Teams

- 云端 workspace。
- 账号和权限。
- 多租户隔离。
- Worker sandbox。
- Secrets manager。
- Organization policy。
- 审计导出。

## 13. 与原方案的差异

保留:

- 本地优先。
- 云端预留。
- Reviewer 核心地位。
- 风险分级。
- 任务库。
- 未来 MCP/connector/executor 路线。

降低:

- MVP 不做后台 scheduler。
- MVP 不做自动 executor。
- MVP 不做完整三阶段 Reviewer。
- MVP 不做完整审计和审批系统。
- MVP 不要求用户迁移全部任务。

新增:

- ChangeSet 中间层。
- Diff-style 用户审阅。
- 自动化成熟度阶梯。
- 任务采纳率作为核心指标。
- “本地任务库不是唯一真相源”的产品原则。

## 14. 参考方向

本方案吸收了以下范式:

- Local-first software: https://www.inkandswitch.com/essay/local-first/
- Trello board/card 心智: https://trello.com/guide/trello-101
- SQLite embedded database: https://www.sqlite.org/serverless.html
- MCP tool boundary: https://modelcontextprotocol.io/specification/2025-06-18/server/tools
- n8n human-in-the-loop: https://docs.n8n.io/advanced-ai/human-in-the-loop-tools/
- LangGraph durable execution: https://docs.langchain.com/oss/python/langgraph/durable-execution
- Trello REST API: https://support.atlassian.com/trello/docs/getting-started-with-trello-rest-api/
- Notion API: https://developers.notion.com/

## 15. 新方案判断

这版方案更适合开工, 因为它把最大的不确定性放在最前面验证:

> AI 能否把用户资料稳定转化为用户愿意接受的任务变更?

如果这个问题不能成立, 原方案里的 scheduler、executor、planner、MCP connector 都没有足够价值。

如果这个问题成立, 后续每一步都很自然:

- 任务草稿被接受, 就可以做任务库。
- 任务库持续增长, 就需要计划器。
- 计划器被信任, 就需要定时建议。
- 定时建议稳定, 就可以开放低风险自动执行。
- 低风险执行稳定, 就可以接外部工具和 MCP。

因此新方案不是保守, 而是把 Agent Task OS 的愿景拆成可验证的增长路径。
