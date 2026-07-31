---
name: team-live-ops
description: "编排 live-ops 团队进行发售后内容规划：协调 live-ops-designer、economy-designer、analytics-engineer、community-manager、writer 和 narrative-director，设计和规划赛季、活动或实时内容更新。"
argument-hint: "[season name or event description] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
model: sonnet
---
**参数检查：** 如果未提供赛季名称或活动描述，输出：
> "Usage: `/team-live-ops [season name or event description]` — 提供要规划的赛季或实时活动的名称或描述。"
然后立即停止，不生成任何子 agent 或读取任何文件。

当此 skill 带有效参数调用时，通过结构化规划管线编排 live-ops 团队。

**决策点：** 在每个阶段转换时，使用 `AskUserQuestion` 向用户
展示子 agent 的提案作为可选项。在对话中写入 agent 的
完整分析，然后用简洁标签捕获决策。
用户必须批准才能进入下一阶段。

## Phase 0: 解析 Review 模式

1. 如果传入了 `--review [mode]` 作为参数，使用该模式。
2. 否则读取 `production/review-mode.txt` — 使用那里写的内容。
3. 否则默认为 `lean`。

模式：
- `full` — 按所述生成所有 director 和 lead gates
- `lean` — 跳过 director gates，除非它们是 PHASE-GATE 类型（CD-PHASE-GATE, TD-PHASE-GATE, PR-PHASE-GATE, AD-PHASE-GATE）
- `solo` — 完全跳过所有 director gate 生成；在没有任何 agent gates 的情况下运行 skill

存储解析后的模式供所有后续阶段使用。

## 团队组成
- **live-ops-designer** — 赛季结构、活动节奏、留存机制、battle pass
- **economy-designer** — 实时经济平衡、商店轮换、货币定价、保底计时器
- **analytics-engineer** — 成功指标、A/B 测试设计、活动追踪、仪表板规范
- **community-manager** — 面向玩家的公告、活动描述、赛季消息
- **narrative-director** — 赛季叙事主题、故事弧线、世界活动框架
- **writer** — 活动描述、奖励物品名称、赛季风味文本、公告文案

## 如何委托

使用 Task 工具将每个团队成员生成为子 agent：
- `subagent_type: live-ops-designer` — 赛季/活动结构和留存机制
- `subagent_type: economy-designer` — 实时经济平衡和奖励定价
- `subagent_type: analytics-engineer` — 成功指标、A/B 测试、活动仪表化
- `subagent_type: community-manager` — 面向玩家的沟通和消息
- `subagent_type: narrative-director` — 赛季主题和叙事框架
- `subagent_type: writer` — 所有面向玩家的文本：活动描述、物品名称、文案

始终在每个 agent 的提示中提供完整上下文（游戏概念路径、现有赛季文档、道德政策路径、当前经济状态）。在管线允许的地方并行启动独立 agent（Phase 3 和 Phase 4 可以同时运行）。

## 管线

### Phase 1: 赛季/活动范围界定
委托给 **live-ops-designer**：
- 定义赛季或活动：类型（赛季性、限时活动、挑战）、时长、主题方向
- 概述内容列表：什么是新的（模式、物品、挑战、故事节拍）
- 定义留存钩子：在这个赛季中每天/每周把玩家带回来的是什么
- 识别资源预算：有多少新内容需要创建 vs. 复用
- 输出：赛季简报，包含范围、内容列表和留存机制概览

### Phase 2: 叙事主题
委托给 **narrative-director**：
- 读取 Phase 1 的赛季简报
- 设计赛季叙事主题：这个活动如何与游戏世界连接？
- 定义玩家在活动期间将发现的核心故事钩子
- 识别这个赛季可以推进的哪些现有传说线索
- 输出：叙事框架文档（主题、故事钩子、传说连接）

### Phase 3: 经济设计（如果主题明确可与 Phase 2 并行）
委托给 **economy-designer**：
- 读取赛季简报和 `design/live-ops/economy-rules.md` 中的现有经济规则
- 设计奖励轨道：免费层级进度、付费层级价值主张
- 规划赛季内经济：赛季货币、商店轮换、定价
- 为任何随机元素定义保底计时器和坏运气保护机制
- 验证付费轨道中没有 pay-to-win 物品
- 输出：经济设计文档，包含奖励表、定价和货币流

### Phase 4: 分析和成功指标（与 Phase 3 并行）
委托给 **analytics-engineer**：
- 读取赛季简报
- 定义成功指标：参与率目标、留存提升目标、battle pass 完成率
- 设计赛季期间运行的任何 A/B 测试（如不同的奖励节奏）
- 指定此赛季内容所需的新遥测事件
- 输出：分析计划，包含成功标准和仪表化要求

### Phase 5: 内容写作（并行）
并行委托：
- **narrative-director**（如需要）：编写任何游戏内叙事文本（过场动画脚本、NPC 对话、世界活动描述）供赛季使用
- **writer**：编写所有面向玩家的文本 — 活动名称、奖励物品描述、挑战目标文本、赛季风味文本
- 两者都应阅读 Phase 2 的叙事框架文档

### Phase 6: 玩家沟通计划
委托给 **community-manager**：
- 读取赛季简报、经济设计和叙事框架
- 起草赛季发布公告（基调、关键亮点、平台专属版本）
- 规划沟通节奏：发布前预热、发布日帖子、赛季中期提醒、最后一周 FOMO 推送
- 起草第一天补丁说明的已知问题部分占位符
- 输出：沟通日历，包含每个接触点的草稿文案

### Phase 7: 审查和签字
收集所有阶段的输出并展示整合的赛季计划：
- 赛季简报（Phase 1）
- 叙事框架（Phase 2）
- 经济设计和奖励表（Phase 3）
- 分析计划和成功指标（Phase 4）
- 书面内容清单（Phase 5）
- 沟通日历（Phase 6）

向用户展示摘要，包括：
- **内容范围**：正在创建什么
- **经济健康检查**：奖励轨道感觉公平且非掠夺性吗？
- **分析准备就绪**：成功标准是否已定义并仪表化？
- **道德审查**：对照 `design/live-ops/ethics-policy.md` 检查 Phase 3 的经济设计
  - 如果文件不存在：标记"ETHICS REVIEW SKIPPED: `design/live-ops/ethics-policy.md` not found. Economy design was not reviewed against an ethics policy. Recommend creating one before production begins." 在赛季设计输出文档中包含此标记。添加到后续步骤：创建 `design/live-ops/ethics-policy.md`。
  - 如果文件存在且发现违规：标记"ETHICS FLAG: [element] in Phase 3 economy design violates [policy rule]. Approval is blocked until this is resolved." 不要发出 COMPLETE 裁决或写入输出文档。使用 `AskUserQuestion`，选项为：修订经济设计 / 用记录的理据覆盖 / 取消。如果用户选择修订：重新生成 economy-designer 生成修正后的设计，然后返回 Phase 7 审查。如果用户选择 Cancel：以 Verdict: BLOCKED 结束 — "Live ops design cancelled due to unresolved ethics violation. Resolve the flagged issues and re-run /team-live-ops."
- **开放问题**：生产开始前仍需要的决策

在委托生产团队之前请用户批准赛季计划。仅在用户批准且没有未解决的道德违规后发出 COMPLETE 裁决。如果道德违规未解决，以 Verdict: **BLOCKED** 结束。

## 输出文档

所有文档保存到 `design/live-ops/`：
- `seasons/S[N]_[name].md` — 赛季设计文档（来自 Phase 1-3）
- `seasons/S[N]_[name]_analytics.md` — 分析计划（来自 Phase 4）
- `seasons/S[N]_[name]_comms.md` — 沟通日历（来自 Phase 6）

## 错误恢复协议

如果任何生成的 agent（通过 Task）返回 BLOCKED、错误或无法完成：

1. **立即展示**：在继续到依赖阶段之前向用户报告"[AgentName]: BLOCKED — [原因]"
2. **评估依赖关系**：检查被阻塞 agent 的输出是否是后续阶段所需的。如果是，未经用户输入不要超过该依赖点。
3. **通过 AskUserQuestion 提供选项**，选项为：
   - 跳过此 agent 并在最终报告中注明缺口
   - 以更窄的范围重试
   - 在此停止并首先解决阻塞
4. **始终生成部分报告** — 输出任何已完成的内容。绝不因为一个 agent 阻塞而丢弃工作。

如果 BLOCKED 状态无法解决，以 Verdict: **BLOCKED** 结束而非 COMPLETE。

## 文件写入协议

所有文件写入（赛季设计文档、分析计划、沟通日历）都委托给
通过 Task 生成的子 agent。每个子 agent 执行
"May I write to [path]?" 协议。此编排器不直接写入文件。

## 输出

涵盖以下内容的摘要：赛季主题和范围、经济设计亮点、成功指标、内容列表、沟通计划，以及生产前需要用户输入的任何开放决策。

裁决：**COMPLETE** — 赛季计划已生成并交接给生产。

## 后续步骤

- 运行 `/design-review` 对赛季设计文档进行一致性验证。
- 运行 `/sprint-plan` 为赛季的内容创建工作排期。
- 赛季内容准备好部署时运行 `/team-release`。
