---
name: retrospective
description: "Generates a sprint or milestone retrospective by analyzing completed work, velocity, blockers, and patterns. Produces actionable insights for the next iteration."
argument-hint: "[sprint-N|milestone-name]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Bash, AskUserQuestion
model: sonnet
---

## Phase 1: 解析参数

确定这是 sprint 回顾（`sprint-N`）还是里程碑回顾（`milestone-name`）。

---

## Phase 1b: 检查现有回顾

在加载任何数据之前，glob 搜索现有回顾文件：

- 对于 sprint 回顾：`production/retrospectives/retro-[sprint-slug]-*.md`
  （同时检查 `production/sprints/sprint-[N]-retrospective.md` 作为备用位置）
- 对于里程碑回顾：`production/retrospectives/retro-[milestone-name]-*.md`

如果找到匹配的文件，使用 `AskUserQuestion`：
- 提示："找到现有回顾：[文件名]。您想如何继续？"
- 选项：
  - `[A] 更新现有 — 加载它并用新数据添加/修订部分`
  - `[B] 重新开始 — 生成新回顾（归档旧的）`

如果选择 [A]：读取现有文件并携带其内容向前，用新数据修订部分。
如果选择 [B]：以空白状态继续到 Phase 2。在写入新文件之前，将现有文件重命名为带 `-archived-[日期]` 后缀的名称。

---

## Phase 2: 加载 Sprint 或 Milestone 数据

从适当的位置读取 sprint 或 milestone 计划：

- Sprint 计划：`production/sprints/`
- Milestone 定义：`production/milestones/`

**还要检查 `production/sprint-status.yaml`**：如果存在，将其与 sprint 计划一起读取。它是实际故事完成状态的权威来源（status: done、完成日期、阻塞项）。将其用作 Phase 3 中完成指标的主要来源。仅在 yaml 不存在时回退到 markdown 扫描。注意 yaml 和 sprint 计划之间的差异（例如，yaml 中但不在计划中的故事，反之亦然）。

**如果文件不存在或为空**，输出：

> "未找到 [sprint/milestone] 的 sprint 数据。运行 `/sprint-status` 先生成
> sprint 数据，或手动提供 sprint 详情。"

然后使用 `AskUserQuestion` 展示两个选项：

- **[A] 手动提供数据** — 请用户粘贴或描述 sprint
  任务、日期和结果；将其作为回顾的真实来源。
- **[B] 停止** — 中止 skill。判定：**BLOCKED** — 无 sprint 数据可用。

如果用户选择 [A]，收集数据并使用他们提供的内容继续到 Phase 3。
如果用户选择 [B]，在此停止。

提取：计划任务、估算工作量、负责人和目标。

为 sprint 期间运行 git log 以了解实际提交了什么以及何时提交。使用 Bash 工具（在 Windows 上使用 Git Bash — `2>/dev/null` 是 bash 语法，不是 PowerShell）：

```
Bash: git log --oneline --since="4 weeks ago" 2>/dev/null || git log --oneline -20
```

如果从 sprint 计划知道 sprint 持续时间，调整 `--since` 日期以匹配。

---

## Phase 3: 分析完成情况和趋势

通过将计划与实际交付物进行比较来扫描已完成和未完成的任务。检查：

- 按计划完成的任务
- 完成但与计划有变的任务
- 结转的任务（未完成）
- sprint 中添加的任务（计划外工作）
- 移除或缩减范围的任务

扫描代码库的 TODO/FIXME 趋势：

- 统计当前 TODO/FIXME/HACK 注释
- 与上一个 sprint 的数量比较（如果可用，检查之前的回顾）
- 注意技术债务是在增长还是缩减

如果存在，从 `production/retrospectives/` 读取之前的回顾以检查：

- 之前的行动项是否已解决？
- 同样的问题是否在重复出现？
- 速度趋势如何？

---

## Phase 4: 生成回顾

```markdown
## 回顾: [Sprint N / Milestone 名称]
期间: [开始日期] -- [结束日期]
生成时间: [日期]

### 指标

| 指标 | 计划 | 实际 | 差值 |
|--------|---------|--------|-------|
| 任务 | [X] | [Y] | [+/- Z] |
| 完成率 | -- | [Z%] | -- |
| 故事点 / 工作日 | [X] | [Y] | [+/- Z] |
| 发现的 Bug | -- | [N] | -- |
| 修复的 Bug | -- | [N] | -- |
| 添加的计划外任务 | -- | [N] | -- |
| 提交数 | -- | [N] | -- |

### 速度趋势

| Sprint | 计划 | 完成 | 比率 |
|--------|---------|-----------|------|
| [N-2] | [X] | [Y] | [Z%] |
| [N-1] | [X] | [Y] | [Z%] |
| [N] (当前) | [X] | [Y] | [Z%] |

**趋势**: [上升 / 稳定 / 下降]
[一句话解释趋势]

### 进展顺利的部分
- [有具体数据或示例支持的观察]
- [另一个积极观察]
- [认可产生回报的具体贡献或决策]

### 进展不佳的部分
- [有可衡量影响的具体问题 — 例如，"功能 X 花了 5 天
  而不是估算的 2 天，阻塞了任务 Y 和 Z"]
- [另一个有影响的问题]
- [不要指责个人 — 关注系统性原因]

### 遇到的阻塞项

| 阻塞项 | 持续时间 | 解决方案 | 预防 |
|---------|----------|------------|------------|
| [阻塞进展的内容] | [多长时间] | [如何解决] | [如何防止再次发生] |

### 估算准确性

| 任务 | 估算 | 实际 | 差异 | 可能原因 |
|------|-----------|--------|----------|--------------|
| [高估最多的任务] | [X] | [Y] | [+Z] | [为什么] |
| [低估最多的任务] | [X] | [Y] | [-Z] | [为什么] |

**整体估算准确性**: [X%] 的任务在估算的 +/- 20% 内

[分析: 我们是否持续高估或低估？对于哪种类型的任务？
我们应该应用什么调整？]

### 结转分析

| 任务 | 原始 Sprint | 结转次数 | 原因 | 操作 |
|------|----------------|---------------|--------|--------|
| [未完成的任务] | [Sprint N-X] | [N] | [为什么] | [完成 / 缩减范围 / 重新设计] |

### 技术债务状态
- 当前 TODO 数量: [N]（之前: [N]）
- 当前 FIXME 数量: [N]（之前: [N]）
- 当前 HACK 数量: [N]（之前: [N]）
- 趋势: [增长 / 稳定 / 缩减]
- [注意任何关注领域]

### 上一轮行动项跟进

| 行动项（来自 Sprint N-1） | 状态 | 备注 |
|-------------------------------|--------|-------|
| [之前的行动] | [已完成 / 进行中 / 未开始] | [上下文] |

### 下一轮迭代的行动项

| # | 行动 | 负责人 | 优先级 | 截止日期 |
|---|--------|-------|----------|----------|
| 1 | [具体、可衡量的行动] | [谁] | [高/中/低] | [何时] |
| 2 | [另一个行动] | [谁] | [优先级] | [何时] |

### 流程改进
- [我们工作方式的具体变更，以及预期收益]
- [另一个改进 — 保持在 2-3 个可操作的项，不是愿望清单]

### 总结
[2-3 句话整体评估: 这是一个好的 sprint/milestone 吗？
向前推进要改变的最重要的一件事是什么？]
```

---

## Phase 5: 保存回顾

向用户展示回顾和主要发现（完成率、速度趋势、首要阻塞项、最重要的行动项）。

询问："我可以将此写入 `production/retrospectives/retro-sprint-[N]-[日期].md` 吗？"（或里程碑回顾的 `production/retrospectives/retro-[milestone-名称]-[日期].md`）

如果同意，写入文件，必要时创建 `production/retrospectives/` 目录。判定：**COMPLETE** — 回顾已保存。

如果不同意，在此停止。判定：**BLOCKED** — 用户拒绝写入。

---

## Phase 6: 后续步骤

使用 `AskUserQuestion`：
- 提示："回顾完成。行动项和速度数据已准备好。您现在要开始 sprint 计划并预加载此数据吗？"
- 选项：
  - `[A] 是 — 用预填充的 retro 行动项和速度增量打开 sprint 计划`
  - `[B] 否 — 我准备好时会手动引用回顾文件`

如果用户选择 [A]：继续调用 `/sprint-plan new`，传递回顾文件路径以及行动项和速度变化的摘要，以便 sprint 计划器可以引用它们。

- 如果这是里程碑回顾，运行 `/gate-check` 以正式评估下一阶段的准备情况。

### 指南

- 诚实且具体。模糊的回顾（"沟通可以更好"）产生模糊的改进。使用数据和示例。
- 关注系统性问题，而不是个人指责。
- 将行动项限制在 3-5 个。更多会稀释焦点。
- 每个行动项必须有负责人和截止日期。
- 检查之前的行动项是否已完成。反复未解决的项目是流程问题的信号。
- 如果这是里程碑回顾，还要评估里程碑目标是否实现以及这对整体项目时间线意味着什么。
