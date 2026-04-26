# 协作会话示例

此目录包含现实的、端到端的会话记录，展示游戏工作室 Agent 架构如何在实践中工作。
每个示例演示 **协作工作流程**，其中 Agent 提问、呈现选项，
并在自主生成内容之前等待用户批准。

---

## 视觉参考

**系统新手？从这里开始：**
[技能流程图](skill-flow-diagrams.md) — 所有 7 个阶段的可视化地图以及技能如何链接。

---

## 📚 **可用示例**

### 核心工作流程

### [技能流程图](skill-flow-diagrams.md)
**类型：** 视觉参考
**复杂度：** 所有级别

完整管线概览（从零到发布），以及以下详细链接图：
design-system、story 生命周期、UX 管线 和 brownfield 入门。
**如果想了解各部分如何组合，从这里开始。**

---

### [会话：用 /design-system 创作 GDD](session-design-system-skill.md)
**类型：** 设计（技能驱动）
**技能：** `/design-system`
**时长：** 约 60 分钟（14 轮）
**复杂度：** 中等

**场景：**
Dev 在 `/map-systems` 生成系统索引后运行 `/design-system movement`。
技能从游戏概念和依赖 GDD 加载上下文，运行技术可行性预检查，
然后逐节引导完成所有 8 个 GDD 章节 — 在进入下一节之前，
起草、批准并将每节写入磁盘。

**关键时刻：**
- 技术可行性预检查标记 Jolt 物理默认更改（Godot 4.6）
- 增量写入：每节在批准后立即写入磁盘
- 第 5 节时会话崩溃 → Agent 从第一个空节恢复
- 依赖章节时 surfaced stamina、inventory 等依赖信号
- 以明确交接结束："在下一个系统之前运行 `/design-review`"

**学习：**
- `/design-system` 与让 agent"写 GDD"有何不同
- 逐节循环如何防止 30k-token 上下文膨胀
- 增量文件写入如何在会话崩溃中存活
- 技能如何 surfaced 下游依赖契约

---

### [会话：完整 Story 生命周期](session-story-lifecycle.md)
**类型：** 完整工作流程
**技能：** `/story-readiness` → 实现 → `/story-done`
**时长：** 约 50 分钟（13 轮）
**复杂度：** 中等

**场景：**
Dev 从 sprint backlog 领取一个 story。`/story-readiness` 在任何代码编写前捕获 roll-direction 歧义。
实现后，`/story-done` 验证 9 个验收标准，识别 2 个推迟的标准（inventory 尚未集成），
并带笔记关闭 story。

**关键时刻：**
- `/story-readiness` 在第 2 轮捕获规格歧义 — 在实现开始前解决
- ADR 状态检查：如果 ADR 仍是 Proposed，story 将被 BLOCKED
- Manifest 版本检查：确认 story 的指导与当前架构没有漂移
- 推迟的标准被追踪（不丢失）当集成尚不可能时
- Story 关闭时更新 `sprint-status.yaml`，自动 surfaced 下一个就绪 story

**学习：**
- 为什么 `/story-readiness` 防止后期实现歧义
- 推迟的标准如何工作（COMPLETE WITH NOTES vs. BLOCKED）
- TR-ID 引用如何防止错误的偏差标记
- 从 backlog → 实现 → 关闭的完整循环

---

### [会话：门控检查和阶段转换](session-gate-check-phase-transition.md)
**类型：** 阶段门控
**技能：** `/gate-check`
**时长：** 约 20 分钟（7 轮）
**复杂度：** 低

**场景：**
Dev 完成系统设计阶段并运行 `/gate-check` 推进。
门控发现所有 6 个 MVP GDD 完成，跨审查以一个低严重性 concerns 通过。
门控通过，`stage.txt` 更新，Agent 为技术设置提供具体有序的检查清单。

**关键时刻：**
- 门控验证产物存在 AND 内部完整性（每个 GDD 8 节）
- CONCERNS ≠ FAIL：低严重性跨审查备注通过门控
- stage.txt 更新改变 `/help`、`/sprint-status` 和所有技能未来的看法
- Agent 将跨审查 concerns 作为具体 ADR surfaced 下一个要写
- 下一阶段检查清单是具体且有序的，不是通用

**学习：**
- 门控检查实际验证什么（不仅仅是"文件存在吗？"）
- PASS/CONCERNS/FAIL 裁决如何工作
- 为什么 stage.txt 是阶段追踪的权威
- 阶段转换后什么改变

---

### [会话：UX 管线 — /ux-design → /ux-review → /team-ui](session-ux-pipeline.md)
**类型：** UX 设计管线
**技能：** `/ux-design`、`/ux-review`、`/team-ui`
**时长：** 约 90 分钟（16 轮）
**复杂度：** 中高

**场景：**
Dev 设计 HUD 和库存屏幕。`/ux-design` 读取玩家旅程和 GDD 以玩家情绪状态为基础做决策。
`/ux-review` 捕获阻塞性无障碍差距（拖放没有键盘替代）和 advisory 色盲问题。
修复后，`/team-ui` 接受交接。

**关键时刻：**
- HUD 哲学选择（diegetic vs. persistent vs. tactical）基于生存类型 convention
- `/ux-review` 区分 BLOCKING（停止交接）vs. ADVISORY（可在视觉通道修复）
- 无障碍在实现前捕获，不是在 QA 期间
- 一轮添加键盘替代；review 重新运行并通过
- `/team-ui` 在开始视觉设计前检查是否有通过的 `/ux-review`

**学习：**
- `/ux-design` 如何使用玩家旅程上下文来支持 UI 决策
- `/ux-review` 实际检查什么（不仅仅是"规格存在吗？"）
- HUD 文档（`design/ux/hud.md`）与每屏幕规格的区别
- 无障碍问题在设计时 vs. 实现时如何处理

---

### [会话：用 /adopt 进行 Brownfield 入门](session-adopt-brownfield.md)
**类型：** Brownfield 采用
**技能：** `/adopt`
**时长：** 约 30 分钟（8 轮）
**复杂度：** 低-中等

**场景：**
Dev 有 3 个月的现有代码和粗略设计笔记但没有正确格式的内容。
`/adopt` 审计格式合规性（不仅仅是文件存在），按严重性分类 4 个差距，
构建有序的 7 步迁移计划，并立即修复 BLOCKING 差距（缺少系统索引）
通过从代码库推断。

**关键时刻：**
- FORMAT 审计区分"文件存在"和"文件具有必需的内部结构"
- BLOCKING 差距确定：缺少系统索引阻止 4+ 技能运行
- 迁移计划是有序的：阻塞差距第一，然后高，然后中
- 系统索引从代码结构引导 — brownfield 代码包含答案
- Retrofit 模式 vs. 新创作：`/design-system retrofit` 填补差距而不覆盖

**学习：**
- `/adopt` 和 `/project-stage-detect` 的区别
- 格式合规性如何检查（章节检测，不仅仅是文件存在）
- Brownfield 项目如何在不丢失现有工作的情况下入门
- 何时使用 retrofit 模式 vs. 完整创作

---

### 基础示例

### [会话：设计合成系统](session-design-crafting-system.md)
**类型：** 设计
**Agent：** game-designer
**时长：** 约 45 分钟（12 轮）
**复杂度：** 中等

**场景：**
Solo dev 需要设计一个为支柱 2（"通过实验的 Emergent Discovery"）服务的合成系统。
Agent 通过提问/回答引导他们，呈现 3 个带游戏理论分析的 设计选项，
合并用户修改，并在每一步迭代起草 GDD 并获得批准。

**关键协作时刻：**
- Agent 提前提出 5 个澄清问题
- 呈现 3 个带优缺点 + MDA 对齐的 distinct 选项
- 用户修改推荐选项，Agent 立即合并
- 主动标记边缘情况（"如果非配方组合呢？"）
- 每个 GDD 章节在进入下一节前显示批准
- 创建文件前有明确的"可以写入 [文件] 吗？"

**学习：**
- 设计 Agent 如何询问目标、约束、参考
- 如何使用游戏设计理论呈现选项（MDA、SDT、Bartle）
- 如何逐节迭代起草
- 何时委托给专家（systems-designer、economy-designer）

---

### [会话：实现战斗伤害计算](session-implement-combat-damage.md)
**类型：** 实现
**Agent：** gameplay-programmer
**时长：** 约 30 分钟（10 轮）
**复杂度：** 低-中等

**场景：**
用户有完整的设计文档，想要实现伤害计算。
Agent 读取规格，识别 7 个歧义/差距，提问澄清，
提议架构供批准，实现时执行规则，并主动写测试。

**关键协作时刻：**
- Agent 首先读取设计文档，识别 7 个规格歧义
- 在实现之前用代码样本提议架构
- 用户要求类型安全，Agent 细化并重新提议
- 规则捕获问题（硬编码值），Agent 透明修复
- 遵循验证驱动开发主动写测试
- 提供下一步选项而不是假设

**学习：**
- 实现 Agent 如何在编码前澄清规格
- 如何用代码样本提议架构供批准
- 规则如何自动执行标准
- 如何处理规格差距（问，不假设）
- 验证驱动开发（测试证明它有效）

---

### [会话：范围危机 — 战略决策](session-scope-crisis-decision.md)
**类型：** 战略决策
**Agent：** creative-director
**时长：** 约 25 分钟（8 轮）
**复杂度：** 高

**场景：**
Solo dev 面临危机：Alpha 里程碑还有 2 周，合成系统需要 3 周，投资者演示成败在此一举。
创意总监收集上下文，正确框架决策，呈现 3 个带诚实权衡分析的战略选项，
做推荐但 defer 给用户，然后用 ADR 和演示脚本记录决策。

**关键协作时刻：**
- Agent 在提议解决方案前读取上下文文档
- 提出 5 个问题以了解决策约束
- 正确框架决策（ stakes 是什么，评估标准是什么）
- 呈现 3 个带风险分析和历史先例的选项
- 做强烈推荐但明确："这是你的决定"
- 记录决策 + 提供演示脚本以支持用户

**学习：**
- 领导 Agent 如何框架战略决策
- 如何呈现带权衡分析的选项
- 如何在推荐中使用游戏开发先例和理论
- 如何记录决策（ADR）
- 如何将决策级联到受影响的部门

---

### [反向文档工作流程](reverse-document-workflow-example.md)
**类型：** Brownfield 文档
**Agent：** game-designer
**时长：** 约 20 分钟
**复杂度：** 低

**场景：**
开发者构建了一个技能树系统但从未写设计文档。
Agent 读取代码，推断设计意图，提问模糊决策的澄清问题，
并生成追溯性 GDD。

---

## 🎯 **这些示例演示什么**

所有示例遵循 **协作工作流程模式：**

```
提问 → 选项 → 决策 → 草稿 → 审批
```

> **注意：** 这些示例将协作模式显示为会话文本。
> 在实践中，Agent 现在在决策点使用 `AskUserQuestion` 工具来呈现
> 结构化选项选择器（带标签、描述和多选）。
> 模式是 **解释 → 捕获**：Agent 首先在对话中解释他们的分析，
> 然后呈现结构化 UI 选择器供用户决策。

### ✅ **展示的协作行为：**

1. **Agent 在假设前提问**
   - 设计 Agent 询问目标、约束、参考
   - 实现 Agent 澄清规格歧义
   - 领导 Agent 在推荐前收集完整上下文

2. **Agent 呈现选项，不是命令**
   - 2-4 个带优缺点的选项
   - 基于理论、先例、项目支柱的推理
   - 做推荐，但用户决定

3. **Agent 在定稿前展示工作**
   - 设计草稿逐节显示
   - 架构提议在实现前显示
   - 战略分析在决策前呈现

4. **Agent 在写入文件前获得批准**
   - 使用 Write/Edit 工具前明确"可以写入 [文件] 吗？"
   - 多文件更改首先列出所有受影响的文件
   - 用户说"是"后才创建任何文件

5. **Agent 迭代反馈**
   - 用户修改立即合并
   - 用户更改推荐时没有防御性
   - 当用户改进 Agent 建议时表示赞赏

---

## 📖 **如何使用这些示例**

### 对于新用户：
在第一次会话前阅读这些示例。它们展示 Agent 如何工作的现实期望：
- Agent 是顾问，不是自主执行者
- 你做所有创意/战略决策
- Agent 提供专家指导和选项

### 用于理解特定工作流程：
- **系统新手？** → 首先阅读 skill-flow-diagrams.md
- **第一次运行 `/design-system`？** → 阅读 session-design-system-skill.md
- **领取 story？** → 阅读 session-story-lifecycle.md
- **完成一个阶段？** → 阅读 session-gate-check-phase-transition.md
- **开始 UI 工作？** → 阅读 session-ux-pipeline.md
- **有现有项目？** → 阅读 session-adopt-brownfield.md
- **设计一个系统（agent 驱动）？** → 阅读 session-design-crafting-system.md
- **实现代码？** → 阅读 session-implement-combat-damage.md
- **做战略决策？** → 阅读 session-scope-crisis-decision.md

### 用于培训：
如果你在教某人使用此系统，逐轮走过一个示例来展示：
- 好的问题是什么样的
- 如何评估呈现的选项
- 何时批准 vs. 请求更改
- 如何在利用 AI 专业知识的同时保持创意控制

---

## 🔍 **所有示例中的常见模式**

### 第 1-2 轮： **在行动前理解**
- Agent 读取上下文（设计文档、规格、约束）
- Agent 提问澄清
- 不假设或猜测

### 第 3-5 轮： **呈现带推理的选项**
- 2-4 个不同方法
- 每个的优缺点
- 支持分析的理论和先例
- 做推荐，决策 defer 给用户

### 第 6-8 轮： **迭代草稿**
- 增量显示工作
- 立即合并反馈
- 主动标记边缘情况或歧义

### 第 9-10 轮： **批准和完成**
- "可以写入 [文件] 吗？"
- 用户："是"
- Agent 写入文件
- Agent 提供下一步（测试、审查、集成）

---

## 🚀 **自己尝试**

阅读这些示例后，尝试这个练习：

1. 选择你的游戏系统之一（战斗、库存、进度等）
2. 让相关 Agent 设计或实现它
3. 注意 Agent 是否：
   - ✅ 首先提问澄清问题
   - ✅ 呈现带推理的选项
   - ✅ 在定稿前显示草稿
   - ✅ 在写入文件前请求批准

如果 Agent 跳过其中任何一个，提醒它：
> "请遵循 docs/COLLABORATIVE-DESIGN-PRINCIPLE.md 中的协作协议"

---

## 📝 **其他资源**

- **完整原则文档：** [docs/COLLABORATIVE-DESIGN-PRINCIPLE.md](../COLLABORATIVE-DESIGN-PRINCIPLE.md)
- **工作流程指南：** [docs/WORKFLOW-GUIDE.md](../WORKFLOW-GUIDE.md)
- **Agent 名册：** [.claude/docs/agent-roster.md](../../.claude/docs/agent-roster.md)
- **CLAUDE.md（协作协议）：** [CLAUDE.md](../../CLAUDE.md#collaboration-protocol)