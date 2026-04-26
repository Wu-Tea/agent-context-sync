# agent-context-sync

`agent-context-sync` 是一个本地优先的 Codex skill，用来维护项目级 AI 上下文连续性。

它解决的不是任务看板、日历或自动执行，而是更基础的一层能力：

- 让新的 session 快速接手已有项目
- 让 subagent 拿到最小必要上下文
- 让关键设计决策可追溯
- 让项目记忆落在仓库本地，而不是散落在平台 memory 里

## 这个 skill 做什么

当项目进入持续协作状态后，`agent-context-sync` 会把 AI 需要反复读取的关键信息整理到 `.agent-context/` 中，形成一套轻量但可持续的上下文协议。

它的核心职责包括：

- 读取和维护项目 handoff
- 记录 session 推进摘要
- 记录设计决策及其理由
- 为 subagent 准备精简 brief
- 在写入上下文前先展示 `SyncSet`
- 用 reviewer 规则检查上下文写入是否干净、克制、可追溯

## 适用场景

适合这些场景：

- “上次做到哪里了？”
- “继续这个项目。”
- “把这次讨论整理进项目上下文。”
- “记录这个设计决策和原因。”
- “给 subagent 准备一个 brief。”

不适合这些场景：

- 一次性的代码解释
- 单条报错排查且不需要项目连续性
- 临时文本处理
- 任务看板、日历、scheduler、自动执行

## 仓库里的两个 skill

本仓库当前包含两个相关 skill：

### 1. `agent-context-sync`

主 skill，负责真正的上下文协议与写入流程，例如：

- `.agent-context/handoff.md`
- `.agent-context/session-log.md`
- `.agent-context/decisions/`
- `.agent-context/briefs/`
- `SyncSet`
- reviewer checklist

### 2. `using-agent-context-sync`

bootstrap skill，用来提高自动触发概率。

它的职责不是直接写上下文，而是在会话开始时判断：

- 当前请求是不是 ongoing project continuity 场景
- 是否应该先读取 `.agent-context/handoff.md`
- 是否应该先路由到 `agent-context-sync`

## `.agent-context/` 目录约定

默认的项目本地上下文目录如下：

```text
.agent-context/
├─ handoff.md
├─ session-log.md
├─ decisions/
└─ briefs/
```

各文件职责：

- `handoff.md`
  当前目标、当前状态、下一步、阻塞、活跃问题
- `session-log.md`
  重要推进记录，按时间追加
- `decisions/`
  关键决策、原因、备选方案和影响
- `briefs/`
  发给 subagent 的最小必要上下文包

## 工作方式

`agent-context-sync` 的推荐流程是：

1. 先读 `.agent-context/handoff.md`
2. 再读相关 decision 和最近 session 记录
3. 只在需要时补读 README、docs、git log 或源码
4. 如果要写入上下文，先给出 `SyncSet`
5. 经过 reviewer 检查后，再由用户确认写入

这能减少两类常见问题：

- 每个新 session 都重新扫描仓库，成本高且容易漏上下文
- AI 把未经确认的推断直接写成“项目事实”

## 快速安装

把这两个目录复制到本机 Codex skill 目录：

```text
C:\Users\Administrator\.codex\skills\codex-primary-runtime\agent-context-sync
C:\Users\Administrator\.codex\skills\codex-primary-runtime\using-agent-context-sync
```

建议同时配置全局：

```text
C:\Users\Administrator\.codex\AGENTS.md
```

可参考的规则：

```markdown
- 当任务涉及继续已有项目、读取上次进展、更新项目上下文、记录设计决策、准备或整合 subagent 时，优先使用 Agent Context Sync。
- 如果当前 workspace 存在 .agent-context/handoff.md，先读它，再看 README / docs / git log。
- 完成较大设计、实现、review 或 subagent 工作后，提议更新 .agent-context/。
```

安装后建议新开 Codex 会话再测试。

## 使用示例

### 接手项目

```text
这个项目你接手一下，具体进度自己读取。
```

### 继续推进

```text
上次做到哪里了？
继续这个项目。
```

### 记录决策

```text
把这个设计决策整理一下，记录原因和备选方案。
```

### 准备 subagent brief

```text
给一个负责 review 的 subagent 准备 brief。
```

### 更新项目上下文

```text
把今天这段推进写进项目上下文。
```

## 自动触发依赖什么

当前自动触发主要依赖三层：

1. `using-agent-context-sync` 的 `description`
2. `agent-context-sync` 的 `description`
3. 仓库级或全局 `AGENTS.md`

它的目标不是强制在所有任务里出现，而是在明显需要 continuity 的场景下，更早、更稳定地被调用。

## 验证方法

Windows 下建议启用 UTF-8 再跑校验：

```powershell
$env:PYTHONUTF8='1'
python C:\Users\Administrator\.codex\skills\.system\skill-creator\scripts\quick_validate.py skills\agent-context-sync
python C:\Users\Administrator\.codex\skills\.system\skill-creator\scripts\quick_validate.py skills\using-agent-context-sync
```

预期输出：

```text
Skill is valid!
```

然后新开一个 Codex 会话，直接输入：

```text
上次做到哪里了？
```

如果自动触发正常，通常会先命中 `using-agent-context-sync`，再路由到 `agent-context-sync`，并优先读取 `.agent-context/handoff.md`。

## 非目标

当前版本明确不做：

- Trello-like 看板
- 日程系统
- `.ics` 导出
- scheduler / automation runner
- 外部 SaaS 写入
- 云端团队协作隔离

## 相关文件

- [AGENTS.md](E:/AI/Trello_AI/AGENTS.md)
- [agent-context-sync SKILL.md](E:/AI/Trello_AI/skills/agent-context-sync/SKILL.md)
- [using-agent-context-sync SKILL.md](E:/AI/Trello_AI/skills/using-agent-context-sync/SKILL.md)
- [requirements](E:/AI/Trello_AI/docs/superpowers/specs/2026-04-22-agent-continuity-ledger-skill-requirements.md)
- [spec](E:/AI/Trello_AI/docs/superpowers/specs/2026-04-22-agent-continuity-ledger-skill-spec.md)
