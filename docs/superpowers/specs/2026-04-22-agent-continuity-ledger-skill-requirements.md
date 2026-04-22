# Agent Context Sync and Decision Memory Skill - 需求说明

日期: 2026-04-22
状态: 需求说明草案
目标形态: 本地 Codex skill
推荐 skill 名称: `agent-context-sync`
历史名称: `agent-continuity-ledger`

## 1. 背景

这个项目最初从“AI Trello / 自动任务拆解 / 日程规划 / 自动执行”开始讨论。经过多轮设计和反对意见检视后，我们认为完整任务管理应用在第一阶段风险过高，也容易与 Trello、Notion、Todoist、Linear、日历产品、Codex 自身任务拆解能力以及各类 agent 平台重叠。

新的切入点更窄，也更贴近真实痛点:

> 让多 session、多 subagent 的 AI 工作能够快速同步必要上下文，并持续记录关键决策、决策由来和后续影响。

用户观察到一个具体问题: Codex 开辟 subagent 时，子 agent 并不继承当前 session 的 memory，也常常需要重新读取当前目录文件来恢复上下文。这说明我们不应该把产品价值建立在“模型天然记得一切”上，而应该建立一个本地、显式、可审计、可被新 session 和 subagent 快速读取的上下文同步层。

这也带来第二个需求: 不能只做临时 handoff。agent 还需要适当维护每次重要 decision 的由来、证据、相关记录和后续影响，方便用户和 agent 回顾“当时为什么这么设计”。

## 2. 产品定位

`agent-context-sync` 是一个本地 Codex skill，用来维护项目内的 AI 工作上下文同步包和决策记忆。

一句话定位:

> 把 AI 工作中真正需要跨 session、跨 subagent 共享的信息，整理成 `.agent-context/` 下的 handoff、session log、decision records 和 subagent briefs。

它不是:

- 任务管理 SaaS
- 看板应用
- 自动日程应用
- 后台 scheduler
- 自动执行 agent 平台
- 外部系统 connector
- 通用个人长期记忆库
- 替代 Codex/ChatGPT 自身推理能力的任务拆分器

它是:

- AI 工作交接协议
- 多 session 上下文同步层
- subagent briefing 工具
- 设计决策记忆系统
- 本地文件型 project memory
- memo / memory 的清洁分流机制

## 3. 核心问题

第一版要解决的问题不是“AI 能不能拆任务”，而是:

- 新 session 不知道上次讨论到哪里。
- subagent 不继承当前 session 的 memory。
- 子 agent 为了补上下文，反复扫描 README、docs、git log 和大量源码。
- 重要设计决策散落在聊天记录里，后续很难知道当时为什么这么选。
- 用户的长期 memo 不应该被项目内临时细节污染。
- AI 常常把自己推断的内容和用户确认过的内容混在一起。
- 多个 agent 的结论缺少一个统一的同步点。

## 4. 目标用户和使用场景

第一版目标用户是频繁使用 Codex 或类似 AI agent 推进项目的人，尤其是会在同一项目中反复开启 session、使用 subagent、讨论设计、修改 spec、做代码实现和 review 的个人用户。

典型使用场景:

- 用户问“这个项目上次做到哪里了”。
- 用户让 AI “把刚才的讨论整理一下，方便下次继续”。
- 用户做了一个产品或技术决策，希望记录“为什么这么决定”。
- 用户准备派发 subagent，希望只给它必要上下文，而不是让它从零扫全仓库。
- subagent 返回结果后，需要把关键结论并入主 session 的可审计记录。
- 用户希望 memo 保持干净，只保存长期偏好和稳定事实，把项目上下文放回项目目录。
- 用户需要回顾一个月前为什么放弃某个方案、接受某个折中。

典型触发语句:

- “更新一下上下文。”
- “把今天的进展写到项目记忆里。”
- “记录这个决策和原因。”
- “生成一个 subagent brief。”
- “下次 session 应该从哪里开始？”
- “帮我整理 handoff。”
- “这个决定以后怎么追溯？”
- “不要污染 memo，把项目上下文写到本地文件。”
- “让子 agent 少扫一点文件，给它一份上下文包。”

## 5. V1 核心目标

第一版必须达成:

- 在当前 workspace 下维护 `.agent-context/` 目录。
- 用 `handoff.md` 保存当前目标、当前状态、下一步、阻塞、活跃问题和优先读取文件。
- 用 `session-log.md` 追加记录重要 session 摘要、subagent 结果和上下文变化。
- 用 `decisions/` 保存 ADR-like decision records，记录 decision、原因、证据、备选方案、影响和复审触发条件。
- 用 `briefs/` 保存 subagent briefs，降低子 agent 反复全量读取仓库的成本。
- 区分用户确认内容、AI 推断内容、待确认内容。
- 在写入上下文文件前生成 `SyncSet`，让用户确认要同步什么。
- 引入 reviewer 规则，对 SyncSet 做自检，避免污染上下文、误记决策或泄露敏感信息。
- 让新 session 能优先读取 `.agent-context/handoff.md` 和相关 decisions，而不是只靠 README/git 推断。
- 让 subagent prompt 能引用 briefing 文件，保持上下文最小且任务聚焦。

## 6. V1 非目标

第一版不做:

- 任务看板 UI
- 完整 task ledger
- daily plan / weekly plan
- `.ics` 日历导出
- 后台定时任务
- 自动提醒
- 自动执行任务
- 外部 SaaS 写入
- 飞书、Notion、Trello、GitHub Issues connector
- websearch / MCP connector
- 云端同步
- 多用户协作
- 团队隔离
- 权限系统
- 长期个人 memory 管理

这些能力可以作为后续方向，但不进入第一版。V1 的成功标准是“上下文恢复和决策追溯是否明显变好”，不是“任务管理功能是否完整”。

## 7. 记忆分层

该 skill 必须明确区分四类记忆，避免把所有信息都塞进 memo 或单一文件。

### 7.1 Platform memo / memory

用途:

- 长期稳定偏好
- 跨项目通用习惯
- 用户明确要求长期记住的信息

不应写入:

- 当前项目临时状态
- 某次 session 的详细过程
- subagent 临时结论
- 尚未确认的设计判断
- 大段代码上下文

### 7.2 `.agent-context/handoff.md`

用途:

- 当前工作交接
- 新 session 启动入口
- 让 agent 快速知道“现在到哪一步”

特点:

- 短
- 可覆盖更新
- 应标注失效条件
- 不保存完整历史

### 7.3 `.agent-context/session-log.md`

用途:

- 追加记录重要 session 结果
- 记录 subagent 返回摘要
- 记录上下文同步发生过什么

特点:

- append-only 为主
- 可以有 correction entry
- 不追求记录所有聊天细节

### 7.4 `.agent-context/decisions/*.md`

用途:

- 记录重要产品、技术、范围和流程决策
- 解释当时为什么这样选
- 保存被拒绝的备选方案
- 给后续复审提供依据

特点:

- 每个重要 decision 单独成文
- 可被 supersede，但不物理删除
- 记录证据和后果

### 7.5 `.agent-context/briefs/*.md`

用途:

- 给 subagent 的最小必要上下文包
- 明确任务边界、读取文件、不要读取的内容、输出格式和相关 decisions

特点:

- 任务专用
- 可过期
- 不替代 handoff 和 decisions

## 8. Workspace 文件契约

默认目录:

```text
.agent-context/
  handoff.md
  session-log.md
  decisions/
    DEC-YYYY-MM-DD-NNN-short-title.md
  briefs/
    subagent-YYYY-MM-DD-NNN-topic.md
```

V1 默认只维护这些文件。若用户要求，也可以在后续版本增加:

```text
.agent-context/
  index.md
  review-log.md
  sources/
  metrics.md
```

但 V1 不应因为结构过重而降低使用频率。

## 9. `handoff.md` 需求

`handoff.md` 必须回答:

- 当前目标是什么。
- 当前状态是什么。
- 下一步最应该做什么。
- 有哪些阻塞。
- 有哪些活跃问题。
- 哪些 decisions 最相关。
- 新 session 应优先读取哪些文件。
- 哪些内容不需要重新打开。
- 当前 handoff 什么时候会过期。

`handoff.md` 不应该:

- 记录完整历史。
- 复制所有 decision 内容。
- 充当任务管理器。
- 保存敏感 token、密钥、隐私数据。
- 把 AI 推断写成用户已确认事实。

## 10. `session-log.md` 需求

`session-log.md` 用于追加记录重要节点。每条记录应包含:

- 时间
- 本次目标
- 发生了什么
- 用户确认了什么
- AI 推断了什么
- subagent 输出了什么
- 写入了哪些 context 文件
- 后续动作
- 需要复审的问题

日志应保持摘要级别，不要变成聊天全文转录。若记录错误，不直接重写历史，而是追加 correction。

## 11. Decision record 需求

每个重要 decision 都应单独保存为:

```text
.agent-context/decisions/DEC-YYYY-MM-DD-NNN-short-title.md
```

必须记录:

- decision 标题
- status: `proposed` / `accepted` / `superseded` / `rejected`
- date
- confirmed by
- context
- decision
- reasons
- rejected alternatives
- evidence
- consequences
- related files
- related sessions
- supersedes / superseded by
- review triggers

需要记录 decision 的情况:

- 产品定位变化
- V1 范围变化
- 技术架构选择
- 放弃某个重要方案
- 接受某个重要折中
- 引入或移除核心文件契约
- 对安全边界、隐私边界、自动化边界做出选择

不需要记录 decision 的情况:

- 普通措辞修改
- 临时 todo
- 纯格式调整
- 还没有形成明确选择的头脑风暴片段

## 12. Subagent brief 需求

在准备派发 subagent 时，skill 应能生成 brief，帮助主 agent 把上下文压缩为“够用但不过载”的任务包。

每份 brief 应包含:

- mission
- role
- required context
- relevant decisions
- files to read first
- files not to read unless needed
- constraints
- expected output
- non-goals
- expiry / stale condition

brief 的目标不是让子 agent 完全继承主 agent 的记忆，而是让它少读无关文件、少误解当前设计、少重复已经被否定的方案。

## 13. SyncSet 协议

`SyncSet` 是所有写入前的审查单。skill 不应直接修改 `.agent-context/`，而应先展示将要同步的内容。

SyncSet 应包含:

- `proposed_handoff_updates`
- `proposed_session_log_entries`
- `proposed_decision_records`
- `proposed_subagent_briefs`
- `fields_requiring_user_confirmation`
- `ai_inferred_items`
- `sensitive_or_excluded_items`
- `reviewer_findings`

用户确认方式:

- “确认写入”
- “按这个同步”
- “接受这个 SyncSet”
- “修改 X 后写入”

模糊回复不算确认，例如:

- “看起来可以”
- “差不多”
- “先这样吧”

除非用户已经明确授权当前步骤可以写入，否则 skill 应先等待确认。

## 14. Reviewer 需求

Reviewer 是规则化审查流程，不一定是独立 agent。它在展示 SyncSet 前运行，用来保护上下文质量。

Reviewer 必须检查:

- 是否把 AI 推断写成用户确认事实。
- 是否记录了没有来源的 decision。
- 是否把临时噪音写进 handoff。
- 是否把长期偏好误写入项目上下文。
- 是否把项目临时状态误建议写进 memo。
- 是否泄露 secrets、token、账号、个人隐私。
- 是否创建了过大的 brief，导致 subagent 仍然需要读太多。
- 是否遗漏了重要 rejected alternatives。
- 是否存在与已有 decision 冲突但没有 supersede 关系的记录。
- 是否让 V1 范围重新滑向 task manager / calendar / scheduler。

Reviewer 输出:

- `accept_draft`
- `needs_user_input`
- `revise_once`
- `manual_only`
- `blocked`

## 15. 安全和隐私边界

允许:

- 读取当前 workspace 的 `.agent-context/`。
- 读取用户明确要求或任务必要的项目文档。
- 生成 SyncSet。
- 在用户确认后创建或更新 `.agent-context/` 文件。
- 在用户确认后生成 subagent brief。

禁止:

- 未确认写入上下文文件。
- 写入平台 memo，除非用户明确要求长期记住。
- 保存 secret、token、cookie、私钥、个人身份信息。
- 删除历史 decision record。
- 把 superseded decision 物理删除。
- 自动写入外部系统。
- 自动执行任务本身。
- 自动创建日历、issue、Trello card 或飞书任务。

## 16. 成功指标

两周 dogfood 成功标准:

- 新 session 能在 2 分钟内通过 `.agent-context/` 说清当前状态、下一步和重要决策。
- subagent brief 能明显减少子 agent 的无关文件扫描。
- 用户可以回顾至少 5 个重要 decision 的原因和备选方案。
- memo 没有被项目临时信息污染。
- handoff 保持短，不变成大杂烩。
- 至少 7 次真实 session 后，用户仍愿意维护 `.agent-context/`。
- reviewer 至少捕获过一次不该写入或需要澄清的内容。
- 没有把 AI 推断误记为用户确认事实。

失败信号:

- 用户仍然让 agent 从 README/git log 重新推断所有进展。
- handoff 文件越来越长，没人愿意读。
- decision records 没有记录 rejected alternatives，无法解释当时为什么选。
- subagent brief 变成复制整份 spec。
- 维护成本超过重新解释的成本。

## 17. 版本路线

### V0 - 文档和设计验证

- 更新需求说明。
- 更新 spec。
- 用当前项目 dogfood 一次 `.agent-context/` 结构。
- 决定 skill 名称是否从 `agent-continuity-ledger` 调整为 `agent-context-sync`。

### V1 - 本地 Context Sync Skill

- 实现 `SKILL.md`。
- 实现 references 和 templates。
- 支持初始化 `.agent-context/`。
- 支持 SyncSet。
- 支持 handoff、session log、decision records、subagent briefs。
- 支持 reviewer checklist。

### V1.1 - 轻量验证工具

- 增加 context 文件结构检查脚本。
- 检查 decision 文件字段完整性。
- 检查 brief 是否过大或缺少任务边界。

### V2 - 外部资料读取

- 支持用户授权的 websearch、飞书、GitHub、Notion 等资料读取。
- 仍默认只写本地 `.agent-context/`。
- 外部写入继续保持人工确认。

### V3 - 应用化或云端化

- 在本地 skill 被证明有效后，再考虑 UI、云端、团队空间、权限隔离和商业化。

## 18. 开放问题

- skill 最终名称是否改为 `agent-context-sync`，还是保留 `agent-continuity-ledger`。
- V1 是否需要 `index.md` 作为目录入口，还是只用 `handoff.md`。
- SyncSet 是否必须每次都等待确认，还是可以允许用户授权“本 session 内自动写 session-log”。
- decision record 的编号是否按日期全局递增，还是按目录内现有文件递增。
- subagent brief 是否默认写文件，还是只在用户要求时写入。
- 当前仓库是否要马上 dogfood 创建 `.agent-context/`，还是先只完成 skill 设计。

## 19. 设计结论

推荐将第一版产品从“任务账本 + 日程导出”收敛为“Agent Context Sync + Decision Memory”。这个方向更贴近用户刚观察到的真实痛点: 多 session 和 subagent 之间缺少可靠、干净、可审计的上下文同步机制。

V1 不和 Trello、日历、Todoist 或 Codex 任务拆分能力正面竞争。它的价值在于让 AI 工作过程有一个本地、清晰、可复盘的上下文协议: 当前做到哪里、下一步是什么、为什么这样设计、哪些方案已被否定、subagent 应该知道什么。
