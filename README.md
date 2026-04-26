# agent-context-sync

本仓库当前不是一个 Trello-like 任务管理应用，而是一个本地优先的 Codex skill 原型项目，核心目标是解决：

- AI 跨 session 的上下文连续性
- subagent 之间的最小必要上下文同步
- 关键设计决策的可追溯记录
- 项目局部记忆与平台长期 memory 的分层

当前 V1 方向已经收敛为两层 skill：

- `agent-context-sync`
  负责真正的项目上下文同步、handoff、session log、decision records、subagent briefs、SyncSet 和 reviewer 流程
- `using-agent-context-sync`
  负责在会话开始时判断是否需要先路由到 `agent-context-sync`

## 当前状态

仓库当前已完成：

- `agent-context-sync` skill 的设计、实现与本地安装验证
- `using-agent-context-sync` bootstrap skill 的实现
- 仓库级 `AGENTS.md`，用于提高在本项目中的自动触发率
- `.agent-context/` dogfood 目录，用于记录本项目自身的 handoff、session log 和 decision history

当前还没有做：

- 完整任务管理 UI
- 看板/拖拽交互
- 日历导出或 `.ics`
- scheduler / 自动执行
- 飞书、Notion、Trello、GitHub Issues 等外部 connector
- 云端/团队版隔离

## 仓库结构

```text
.
├─ AGENTS.md
├─ README.md
├─ docs/
│  └─ superpowers/
│     ├─ plans/
│     └─ specs/
├─ skills/
│  ├─ agent-context-sync/
│  │  ├─ SKILL.md
│  │  ├─ agents/openai.yaml
│  │  ├─ assets/
│  │  └─ references/
│  └─ using-agent-context-sync/
│     ├─ SKILL.md
│     └─ agents/openai.yaml
└─ .agent-context/
   ├─ handoff.md
   ├─ session-log.md
   └─ decisions/
```

## 核心概念

### 1. `.agent-context/`

项目局部上下文目录，而不是长期个人 memory。当前约定包括：

- `handoff.md`
  当前目标、当前状态、下一步、阻塞、活跃问题
- `session-log.md`
  重要 session 和 subagent 结果的追加记录
- `decisions/`
  ADR-like 设计决策记录
- `briefs/`
  给 subagent 的最小必要上下文包

### 2. `SyncSet`

在写入 `.agent-context/` 前先展示 proposed write set，避免 AI 直接污染上下文。

### 3. `reviewer`

在展示或应用 SyncSet 前做规则化自检，重点检查：

- AI 推断是否被误写成用户确认事实
- handoff 是否过长
- decision 是否缺少 rejected alternatives
- 是否泄露 secrets / credentials
- 是否又滑回 task manager / calendar / scheduler 范围

### 4. bootstrap routing

`using-agent-context-sync` 的职责不是写项目上下文，而是像一个“启动路由器”那样判断：

- 当前会话是不是 ongoing project continuity 场景
- 是否应该先读 `.agent-context/handoff.md`
- 是否应该先切换到 `agent-context-sync`

## 快速开始

### 方案 A：只在当前仓库里使用

1. 打开本仓库。
2. 确保仓库根目录存在 `AGENTS.md`。
3. 新开 Codex 会话。
4. 直接用自然语言提问，例如：

```text
上次做到哪里了？
继续这个项目。
更新一下上下文。
帮我记录这个决策。
给 subagent 准备 brief。
```

预期行为：

- 会优先命中 `using-agent-context-sync`
- 它会判断当前是不是 continuity 场景
- 然后路由到 `agent-context-sync`
- 如果有 `.agent-context/handoff.md`，会先读它，再补 README / docs / git / source

### 方案 B：安装到本机 Codex runtime

把这两个 skill 目录复制到本机 skill 目录，例如：

```text
C:\Users\Administrator\.codex\skills\codex-primary-runtime\agent-context-sync
C:\Users\Administrator\.codex\skills\codex-primary-runtime\using-agent-context-sync
```

建议同时配置全局：

```text
C:\Users\Administrator\.codex\AGENTS.md
```

示例规则：

```markdown
- 当任务涉及继续已有项目、读取上次进展、更新项目上下文、记录设计决策、准备或整合 subagent 时，优先使用 Agent Context Sync。
- 如果当前 workspace 存在 .agent-context/handoff.md，先读它，再看 README / docs / git log。
- 完成较大设计、实现、review 或 subagent 工作后，提议更新 .agent-context/。
```

安装后建议重启 Codex 或新开会话。

## 使用示例

### 读取项目当前状态

```text
这个项目你接手一下，具体进度自己读取。
```

### 继续已有项目

```text
上次做到哪里了？
继续这个项目。
```

### 记录设计决策

```text
把这个设计决定记录一下，并说明为什么这么选。
```

### subagent briefing

```text
给一个强烈反对方 subagent 准备 brief。
```

### 更新上下文

```text
把今天这段推进整理进项目上下文。
```

## 自动触发是怎么做的

当前自动触发主要依赖 3 层：

1. `using-agent-context-sync` 的 `description`
   覆盖 “starting / resuming / continuing work” 和一组中文触发词
2. `agent-context-sync` 的 `description`
   覆盖 `.agent-context/`、handoff、decision、subagent、project memory 等 continuity cues
3. 仓库级/全局 `AGENTS.md`
   告诉 Codex 在哪些场景下优先走 continuity 流程

它不是 100% 强制触发，而是尽量把触发概率拉高，同时避免误伤明显的一次性任务。

## 验证方法

### 验证 skill 结构

Windows 下建议显式启用 UTF-8 再跑校验脚本：

```powershell
$env:PYTHONUTF8='1'
python C:\Users\Administrator\.codex\skills\.system\skill-creator\scripts\quick_validate.py skills\agent-context-sync
python C:\Users\Administrator\.codex\skills\.system\skill-creator\scripts\quick_validate.py skills\using-agent-context-sync
```

预期输出：

```text
Skill is valid!
```

### 验证自动触发

新开一个 Codex 会话，不显式写 `$agent-context-sync`，直接输入：

```text
上次做到哪里了？
```

或：

```text
这个项目你接手一下，具体进度自己读取。
```

如果自动触发成功，通常会出现：

- 先走 `using-agent-context-sync`
- 再路由到 `agent-context-sync`
- 先读 `.agent-context/handoff.md`
- 再补 session log、decisions、docs、git 或源码

## 重要文档

- 需求说明  
  `docs/superpowers/specs/2026-04-22-agent-continuity-ledger-skill-requirements.md`
- 技术规格  
  `docs/superpowers/specs/2026-04-22-agent-continuity-ledger-skill-spec.md`
- 实现计划  
  `docs/superpowers/plans/2026-04-22-agent-context-sync-skill.md`

## V1 非目标

以下内容明确不属于当前 V1：

- 完整任务系统
- Trello-like 看板
- 日程规划器
- `.ics` 导出
- scheduler / automation runner
- 外部 SaaS 写入
- 团队协作权限系统
- 云端服务化

## 开发备注

- 当前仓库里的 `.agent-context/` 主要用于 dogfood 本项目自身的 continuity 流程。
- 是否将 `.agent-context/` 提交进 git，是一个项目策略问题，不是 skill 的硬要求。
- 如果未来要商业化或给别人安装，下一步更可能是做成更标准的 plugin / repo-local `.agents/skills/` 分发结构，而不是一直靠手工复制目录。
