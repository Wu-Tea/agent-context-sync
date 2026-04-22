# Local Agent Task OS - 程序设计文档

日期: 2026-04-22
状态: 初版设计草案
目标形态: 本地优先的任务代理应用, 未来可演进为云端 SaaS

## 1. 背景与目标

本项目要做的不是传统待办清单, 而是一个本地运行的工作代理应用。用户可以输入需求资料、会议记录、文档、想法或项目背景, 系统自动将其转化为目标、任务、子任务、依赖关系和日程安排。系统可以在本地定时唤醒, 自动执行低风险知识工作, 并把执行结果回写到任务库。

第一版优先服务个人用户, 但数据模型预留团队和多租户字段。后续如果产品验证成功, 可以迁移为云端方案, 增加团队隔离、权限控制、审计和托管 worker。

核心目标:

- 从非结构化资料中自动创建任务和子任务。
- 与用户讨论并确定优先级、依赖关系、风险等级和执行顺序。
- 根据任务优先级、截止时间、依赖关系和可用时间自动生成每日计划。
- 使用本地定时任务或后台 scheduler 自动执行低风险知识任务。
- 对中高风险任务设置人工确认关卡。
- 引入独立 Reviewer, 对任务拆解、排期和执行结果进行质量审查。
- 将所有任务、来源、计划、执行记录和审查记录写入本地任务库。

非目标:

- 第一版不做完整 SaaS 多租户系统。
- 第一版不优先做复杂团队协同。
- 第一版不自动执行高风险外部操作, 例如删除文件、发送正式消息、修改生产系统、执行未知脚本。
- 第一版不以程序员工作流为唯一目标, 代码和终端能力作为后续 executor 插件扩展。

## 2. 产品定位

第一版产品可以定义为:

> 本地运行的 AI 工作代理。它能接收资料, 拆解任务, 生成计划, 定时执行低风险知识工作, 并通过独立 Reviewer 保障任务质量和执行安全。

它更接近 Trello + AI planner + local automation runner, 但第一版不需要复制 Trello 的完整协作能力。看板只是任务状态和计划的可视化承载, 任务大脑和执行引擎才是核心。

## 3. MVP 范围

### 3.1 输入能力

MVP 支持:

- 用户粘贴文本。
- 用户上传 Markdown、TXT、PDF、Word、CSV 或 Excel 文件。
- 用户通过对话补充背景、限制条件、截止日期和偏好。

MVP 预留:

- 飞书、Notion、Google Drive、Trello、邮箱等外部 source connector。
- Web search connector。
- MCP server 作为资料来源和工具执行入口。

### 3.2 任务生成能力

系统从输入资料中识别:

- 项目目标。
- 里程碑。
- 任务。
- 子任务。
- 依赖关系。
- 截止时间。
- 预估工作量。
- 风险等级。
- 所需资料。
- 验收标准。

任务生成后不会直接入执行队列, 必须先经过 Reviewer 审查。

### 3.3 排期能力

系统根据以下信息生成每日计划:

- 任务优先级。
- 任务依赖关系。
- 截止日期。
- 预估工时。
- 用户每天可用时间。
- 用户偏好的工作节奏。
- 任务是否可自动执行。
- 已完成和失败的历史记录。

每日计划生成后需要再次经过 Reviewer 审查, 避免计划不可执行、任务顺序错误或负载过高。

### 3.4 自动执行能力

MVP 自动执行低风险知识工作:

- 总结资料。
- 提取待办。
- 生成日报、周报、会议纪要草稿。
- 生成研究提纲。
- 整理任务描述和验收标准。
- 更新本地任务状态。
- 生成待人工确认的外部操作草稿。

MVP 不自动执行:

- 删除或覆盖用户文件。
- 发送邮件、飞书、Slack、微信等正式消息。
- 修改外部系统数据。
- 执行任意 shell 命令。
- 修改代码仓库。
- 访问敏感凭据。

这些动作未来可以作为高风险 executor, 默认进入人工确认队列。

## 4. 总体架构

核心流程:

```text
Input Sources
  -> Ingestion Service
  -> Task Brain
  -> Reviewer
  -> Task Store
  -> Planner
  -> Reviewer
  -> Scheduler
  -> Approval Gate
  -> Executor
  -> Reviewer
  -> Task Store
  -> User Review Queue
```

核心模块:

- `Local App`: 本地 Web 或桌面应用, 提供输入、任务库、看板、日程、审批队列和执行日志。
- `Ingestion Service`: 接收文本和文件, 抽取内容, 记录 source metadata。
- `Task Brain`: 从资料和对话中生成目标、任务树、依赖、优先级建议和验收标准。
- `Reviewer`: 独立审核代理, 只审查不执行。
- `Planner`: 将任务转化为每日计划和执行候选队列。
- `Scheduler`: 本地定时唤醒 planner 和 executor。
- `Approval Gate`: 根据风险等级、权限和用户策略判断是否自动执行。
- `Executor`: 执行低风险知识任务, 未来通过插件扩展到 MCP、SaaS、浏览器、终端和代码。
- `Task Store`: 本地 SQLite 数据库, 存储任务、计划、来源、执行记录、审查记录和用户偏好。
- `Audit Log`: 记录每一次 AI 决策、审查、执行和人工确认。

## 5. Reviewer 设计

Reviewer 是第一版的核心安全和质量模块。它不是 UI 上的一个按钮, 而是独立于 Task Brain、Planner 和 Executor 的审核代理。

Reviewer 原则:

- Reviewer 只审查, 不执行。
- Reviewer 使用独立 prompt 和独立输出 schema。
- Reviewer 不能调用高风险工具。
- Reviewer 不能批准自己生成的任务。
- Reviewer 的输出必须写入审查记录。
- Reviewer 可以要求补充信息、退回重拆、降低自动化等级或进入人工确认。

### 5.1 任务生成后审查

检查点:

- 是否存在重复任务。
- 任务是否可执行。
- 子任务是否过粗或过细。
- 是否缺少关键上下文。
- 任务描述是否包含明确完成条件。
- 依赖关系是否合理。
- 风险等级是否低估。
- 是否需要用户确认目标或边界。

输出:

- `approved`: 可进入任务库。
- `needs_revision`: 退回 Task Brain 重写。
- `needs_user_input`: 请求用户补充信息。
- `blocked`: 不应创建任务。

### 5.2 排期后审查

检查点:

- 每日工作量是否超出用户可用时间。
- 是否违反依赖顺序。
- 是否把高风险任务放入自动执行队列。
- 是否忽略截止日期。
- 是否过度碎片化。
- 是否缺少缓冲时间。

输出:

- `approved`: 可进入 schedule。
- `revise_plan`: 退回 Planner 调整。
- `ask_user`: 请求用户选择优先级或时间偏好。

### 5.3 执行后审查

检查点:

- 执行产出是否满足验收标准。
- 资料摘要是否覆盖关键事实。
- 生成文档是否结构完整。
- 是否产生幻觉或无来源结论。
- 是否需要人工确认后才能标记完成。
- 是否应创建后续任务。

输出:

- `pass`: 标记任务完成或等待用户查看。
- `retry`: 退回 Executor 重做。
- `partial`: 标记部分完成并创建剩余任务。
- `needs_user_review`: 进入人工审阅队列。

## 6. 风险分级和审批规则

### 6.1 风险等级

`L0 - 只读分析`

- 读取已授权资料。
- 总结、分类、提取待办。
- 不修改外部系统。

默认可自动执行。

`L1 - 本地低风险写入`

- 创建本地任务。
- 更新本地任务状态。
- 生成本地草稿文件。
- 写入应用自己的数据库。

默认可自动执行, 但需要审计记录。

`L2 - 外部草稿`

- 生成邮件草稿。
- 生成飞书/Slack 消息草稿。
- 生成外部系统更新建议。

可自动生成, 但发送或提交前需要人工确认。

`L3 - 外部写入`

- 修改外部 SaaS 数据。
- 创建日历事件。
- 发出消息。
- 调用带副作用的 API。

默认需要人工确认。

`L4 - 高风险系统操作`

- 删除文件。
- 覆盖用户文档。
- 执行 shell 命令。
- 修改代码仓库。
- 使用敏感凭据。
- 执行付款、采购、生产系统操作。

MVP 禁止自动执行, 必须显式确认并记录。

### 6.2 Approval Gate

Approval Gate 根据以下条件做决定:

- 任务风险等级。
- 用户配置的自动化策略。
- Reviewer 审查结果。
- executor 所需权限。
- 是否访问外部系统。
- 是否会产生不可逆副作用。

决策结果:

- `auto_run`: 自动执行。
- `draft_only`: 只生成草稿。
- `manual_approval`: 等待用户确认。
- `deny`: 拒绝执行。

## 7. 数据模型草案

SQLite 表建议:

### 7.1 workspaces

- `id`
- `name`
- `owner_id`
- `created_at`
- `updated_at`

MVP 只有一个默认 workspace, 但保留字段用于未来团队隔离。

### 7.2 users

- `id`
- `display_name`
- `email`
- `preferences_json`
- `created_at`

MVP 可以只有本地用户。

### 7.3 sources

- `id`
- `workspace_id`
- `type`
- `title`
- `uri`
- `content_hash`
- `metadata_json`
- `created_at`

### 7.4 artifacts

- `id`
- `workspace_id`
- `source_id`
- `type`
- `title`
- `content`
- `metadata_json`
- `created_at`

用于存储摘要、草稿、报告、会议纪要等 AI 产物。

### 7.5 tasks

- `id`
- `workspace_id`
- `parent_task_id`
- `source_id`
- `title`
- `description`
- `status`
- `priority`
- `risk_level`
- `estimated_minutes`
- `due_at`
- `assignee_id`
- `acceptance_criteria`
- `metadata_json`
- `created_at`
- `updated_at`

### 7.6 task_dependencies

- `id`
- `workspace_id`
- `task_id`
- `depends_on_task_id`
- `type`
- `created_at`

### 7.7 plans

- `id`
- `workspace_id`
- `date`
- `status`
- `generated_by_run_id`
- `review_status`
- `created_at`

### 7.8 plan_items

- `id`
- `workspace_id`
- `plan_id`
- `task_id`
- `scheduled_start`
- `scheduled_end`
- `execution_mode`
- `status`
- `created_at`

### 7.9 agent_runs

- `id`
- `workspace_id`
- `agent_type`
- `input_json`
- `output_json`
- `status`
- `model`
- `started_at`
- `finished_at`

### 7.10 reviews

- `id`
- `workspace_id`
- `target_type`
- `target_id`
- `review_stage`
- `decision`
- `findings_json`
- `created_by_run_id`
- `created_at`

### 7.11 approvals

- `id`
- `workspace_id`
- `target_type`
- `target_id`
- `risk_level`
- `decision`
- `requested_at`
- `decided_at`
- `decided_by`
- `notes`

### 7.12 audit_logs

- `id`
- `workspace_id`
- `actor_type`
- `actor_id`
- `action`
- `target_type`
- `target_id`
- `metadata_json`
- `created_at`

## 8. Agent 和服务边界

### 8.1 Ingestion Service

职责:

- 接收文本和文件。
- 抽取纯文本。
- 生成 source 和 artifact。
- 标记来源和内容 hash。

不负责:

- 判断任务优先级。
- 自动执行任务。

### 8.2 Task Brain

职责:

- 从资料中提取目标和任务。
- 生成任务树。
- 生成依赖关系建议。
- 估算优先级、工作量和风险等级。
- 输出结构化 JSON。

不负责:

- 直接写入最终任务库。
- 自动执行任务。
- 审批自己的结果。

### 8.3 Planner

职责:

- 根据任务库和用户偏好生成计划。
- 处理依赖、截止时间和工作量。
- 输出 plan 和 plan_items。

不负责:

- 执行任务。
- 绕过 Reviewer 或 Approval Gate。

### 8.4 Scheduler

职责:

- 定时触发每日计划生成。
- 定时扫描可执行任务。
- 定时生成总结。
- 支持手动 trigger。

MVP 可使用:

- 应用内 Node/Python scheduler。
- SQLite job table。
- Windows Task Scheduler 作为外层唤醒器。

### 8.5 Executor

职责:

- 执行低风险知识任务。
- 生成 artifacts。
- 更新任务进度。
- 将执行结果交给 Reviewer。

Executor 插件接口:

```text
executor.run(task, context, permissions) -> execution_result
executor.required_permissions(task) -> permission_request
executor.risk_hint(task) -> risk_level
```

MVP executor:

- `summarize_document`
- `extract_todos`
- `draft_report`
- `draft_daily_review`
- `normalize_task`
- `update_local_task`

未来 executor:

- `mcp_tool_executor`
- `web_search_executor`
- `feishu_executor`
- `notion_executor`
- `browser_executor`
- `code_executor`
- `shell_executor`

## 9. 本地应用形态

MVP 页面:

- `Inbox`: 粘贴文本、上传文件、查看资料处理状态。
- `Tasks`: 树状任务、看板状态、优先级、风险等级、依赖关系。
- `Plan`: 今日计划、未来计划、可自动执行任务、待确认任务。
- `Review Queue`: Reviewer 发现的问题、需要用户确认的任务、执行结果待审阅。
- `Artifacts`: 摘要、报告、草稿、会议纪要、执行产物。
- `Settings`: 工作时间、自动化策略、模型配置、connector 配置。
- `Audit`: agent run、review、approval、execution log。

## 10. 推荐技术栈

由于当前目标是本地优先, 推荐:

- Frontend: React + TypeScript + Vite。
- Desktop shell: 先用本地 Web app, 后续可封装 Tauri 或 Electron。
- Backend: Python FastAPI 或 Node.js。
- Database: SQLite。
- Scheduler: APScheduler, node-cron, 或应用内 job runner。
- File parsing: 按文件类型接入 PDF、docx、xlsx、markdown parser。
- AI provider: 先抽象为 `ModelProvider` 接口, 支持 OpenAI-compatible API。
- Tool integration: 后续通过 MCP client 或 connector plugin 接入。

如果希望更快验证, 可以先采用单体架构:

```text
local-app/
  frontend/
  backend/
  data/app.db
  storage/sources/
  storage/artifacts/
```

后续云端化时拆分:

```text
web-app
api-service
planner-service
executor-worker
reviewer-worker
connector-service
tenant-aware database
object storage
```

## 11. 关键工作流

### 11.1 资料到任务

1. 用户在 Inbox 输入文本或上传文件。
2. Ingestion Service 提取文本并创建 source。
3. Task Brain 生成任务树和结构化建议。
4. Reviewer 审查任务质量和风险等级。
5. 通过审查的任务写入 Task Store。
6. 未通过的任务退回 Task Brain 或进入用户补充信息队列。

### 11.2 任务到每日计划

1. Scheduler 每天早上或用户手动触发 Planner。
2. Planner 读取任务、依赖、截止日期和偏好。
3. Planner 生成 plan_items。
4. Reviewer 审查计划可行性。
5. 通过审查的计划写入 Plan。
6. 高风险或冲突计划进入 Review Queue。

### 11.3 自动执行

1. Scheduler 扫描到可执行 plan_item。
2. Approval Gate 判断是否允许自动执行。
3. 低风险任务进入 Executor。
4. Executor 生成结果 artifact 或更新本地任务。
5. Reviewer 审查执行结果。
6. 通过审查则标记完成或等待用户查看。
7. 未通过则 retry、partial 或 needs_user_review。

### 11.4 动态插入任务

1. 用户随时新增资料或任务。
2. Task Brain 判断新任务是否影响已有依赖和计划。
3. Planner 执行局部重排。
4. Reviewer 检查重排是否破坏关键截止日期。
5. 系统展示变更摘要, 对低风险重排自动应用, 对重大重排请求用户确认。

## 12. 错误处理和恢复

失败类型:

- 文件解析失败。
- LLM 输出格式错误。
- 任务拆解质量不合格。
- 计划冲突。
- executor 失败。
- Reviewer 判定不通过。
- 本地 scheduler 未运行。

处理策略:

- 所有 agent 输出使用 schema validation。
- 失败任务保留输入、输出和错误信息。
- executor 需要幂等设计, 避免重复生成不可控副作用。
- 自动 retry 仅适用于 L0 和 L1。
- L2 以上失败进入人工队列。
- 每天生成系统健康摘要。

## 13. 安全和权限

MVP 安全原则:

- 默认本地存储。
- 默认最小权限。
- 高风险 executor 默认关闭。
- 外部 connector 需要显式配置凭据。
- 每次外部写入都需要 approval record。
- 所有 AI 决策和执行结果记录 audit log。
- 用户可以查看为什么某个任务被自动执行或被阻止。

未来云端化需要增加:

- tenant_id 隔离。
- workspace 权限模型。
- RBAC。
- secrets manager。
- per-tenant encryption。
- audit export。
- organization policy。
- worker sandbox。

## 14. 未来扩展

### 14.1 MCP

MCP 可以同时作为 source connector 和 tool executor:

- 从 MCP server 获取资料。
- 调用 MCP tool 完成外部查询。
- 将风险等级绑定到 tool metadata。
- 对所有 MCP tool call 走 Approval Gate。

### 14.2 外部资料

可扩展 connectors:

- 飞书文档。
- 飞书多维表格。
- Notion。
- Google Drive。
- Trello。
- Web search。
- 邮箱。
- GitHub issues。

### 14.3 云端商业化

云端版本可增加:

- 团队 workspace。
- 组织策略。
- 多用户任务分派。
- 团队 Reviewer policy。
- 托管调度器。
- 托管 executor worker。
- 连接器市场。
- 审计和合规报表。

## 15. MVP 里程碑建议

`M1 - 本地任务库和资料输入`

- SQLite schema。
- Inbox。
- 文件和文本导入。
- source 和 artifact 存储。

`M2 - Task Brain + Reviewer`

- 资料生成任务树。
- Reviewer 审查任务质量。
- 任务写入库。

`M3 - Planner + 每日计划`

- 工作时间偏好。
- 依赖和截止日期处理。
- Reviewer 审查计划。

`M4 - Scheduler + 低风险 Executor`

- 本地定时触发。
- 自动总结、提取、生成草稿。
- 执行后 Reviewer。

`M5 - Review Queue + Audit`

- 人工确认队列。
- 审查记录。
- 执行日志。
- 失败恢复。

## 16. 待验证问题

- 第一版应该做成本地 Web app, 还是直接封装桌面应用。
- 后端应选择 Python FastAPI 还是 Node.js。
- LLM provider 是否需要第一版就支持多模型。
- 自动执行的最低可用任务集应该保留几个 executor。
- Reviewer 是否使用同一个模型不同 prompt, 还是允许使用更便宜或更严格的模型。
- 本地 scheduler 是否需要脱离应用进程独立运行。
- 第一版是否需要日历视图, 还是只做今日计划和计划队列。

## 17. 初版设计结论

推荐执行方案:

- 采用本地优先、云端预留的单体架构。
- 第一版聚焦知识工作自动化, 不把程序员任务作为唯一场景。
- 以文本和文件输入作为 MVP source。
- 使用 SQLite 作为任务库。
- 使用独立 Reviewer 作为任务生成、排期和执行后的质量与安全门。
- 使用风险分级和 Approval Gate 控制自动执行边界。
- 只自动执行 L0 和 L1 任务。
- L2 生成草稿, L3 和 L4 必须人工确认或禁止。

这版设计的核心判断是: 自动执行能力必须建立在任务库、审查机制、风险边界和审计记录之上。否则系统会变成一个会定时运行的聊天机器人, 而不是可靠的工作代理应用。
