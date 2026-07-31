---
name: team-combat
description: "编排战斗团队：协调 game-designer、gameplay-programmer、ai-programmer、technical-artist、sound-designer 和 qa-tester，端到端地设计、实现和验证战斗功能。"
argument-hint: "[combat feature description] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
model: sonnet
---
**参数检查：** 如果未提供战斗功能描述，输出：
> "Usage: `/team-combat [combat feature description]` — 提供要设计和实现的战斗功能的描述（如 `melee parry system`、`ranged weapon spread`）。"
然后立即停止，不生成任何子 agent 或读取任何文件。

当此 skill 带有效参数调用时，通过结构化管线编排战斗团队。

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
- **game-designer** — 设计机制，定义公式和边界情况
- **gameplay-programmer** — 实现核心游戏代码
- **ai-programmer** — 实现 NPC/敌人 AI 行为
- **technical-artist** — 创建 VFX、着色器效果和视觉反馈
- **sound-designer** — 定义音频事件、冲击声音和环境战斗音频
- **engine specialist**（主要） — 验证架构和实现模式对引擎是惯用的（从 `.claude/docs/technical-preferences.md` 的 Engine Specialists 部分读取）
- **qa-tester** — 编写测试用例并验证实现

## 如何委托

使用 Task 工具将每个团队成员生成为子 agent：
- `subagent_type: game-designer` — 设计机制，定义公式和边界情况
- `subagent_type: gameplay-programmer` — 实现核心游戏代码
- `subagent_type: ai-programmer` — 实现 NPC/敌人 AI 行为
- `subagent_type: technical-artist` — 创建 VFX、着色器效果、视觉反馈
- `subagent_type: sound-designer` — 定义音频事件、冲击声音、环境音频
- `subagent_type: [primary engine specialist]` — 架构和实现的引擎惯用验证
- `subagent_type: qa-tester` — 编写测试用例并验证实现

始终在每个 agent 的提示中提供完整上下文（设计文档路径、相关代码文件、约束）。在管线允许的地方并行启动独立 agent（如 Phase 3 的 agent 可以同时运行）。

## 管线

### Phase 1: 设计
委托给 **game-designer**：
- 在 `design/gdd/` 中创建或更新设计文档，涵盖：机制概览、玩家幻想、详细规则、带变量定义的公式、边界情况、依赖关系、带安全范围的调优旋钮，以及验收标准
- 输出：完成的设计文档

### Phase 2: 架构
委托给 **gameplay-programmer**（如果涉及 AI 则连同 **ai-programmer**）：
- 审查设计文档
- 设计代码架构：类结构、接口、数据流
- 识别与现有系统的集成点
- 输出：带文件列表和接口定义的架构草图

然后生成**主要引擎专家**来验证提议的架构：
- 类/节点/组件结构对固定引擎来说是惯用的吗？（如 Godot 节点层级、Unity MonoBehaviour vs DOTS、Unreal Actor/Component 设计）
- 是否有引擎原生系统应该用来替代自定义实现？
- 固定引擎版本中是否有任何提议的 API 已弃用或更改？
- 输出：引擎架构说明 — 在 Phase 3 开始之前纳入架构

使用 `AskUserQuestion`：
- 提示："Architecture sketch complete. Approve to proceed with parallel implementation."
- 选项：
  - `[A] Proceed — spawn implementation agents (gameplay-programmer, ai-programmer, technical-artist, sound-designer)`
  - `[B] Revise the architecture first — I'll describe what needs to change`
  - `[C] Stop here — I'll continue later`

仅当用户选择 [A] 时生成实现 agent。

### Phase 3: 实现（尽可能并行）
并行委托：
- **gameplay-programmer**：实现核心战斗机制代码
- **ai-programmer**：实现 AI 行为（如果功能涉及 NPC 反应）
- **technical-artist**：创建 VFX 和着色器效果
- **sound-designer**：定义音频事件列表和混音说明

### Phase 4: 集成
- 将游戏代码、AI、VFX 和音频连接在一起
- 确保所有调优旋钮都已暴露且是数据驱动的
- 验证功能与现有战斗系统配合工作

### Phase 5: 验证
委托给 **qa-tester**：
- 从验收标准编写测试用例
- 测试设计中记录的所有边界情况
- 验证性能影响在预算内
- 为发现的任何问题提交 bug 报告

### Phase 6: 签字
- 收集团队成员的所有结果
- 报告功能状态：COMPLETE / NEEDS WORK / BLOCKED
- 列出任何未解决的问题及其分配的负责人

## 错误恢复协议

如果任何生成的 agent（通过 Task）返回 BLOCKED、错误或无法完成：

1. **立即展示**：在继续到依赖阶段之前向用户报告"[AgentName]: BLOCKED — [原因]"
2. **评估依赖关系**：检查被阻塞 agent 的输出是否是后续阶段所需的。如果是，未经用户输入不要超过该依赖点。
3. **通过 AskUserQuestion 提供选项**，选项为：
   - 跳过此 agent 并在最终报告中注明缺口
   - 以更窄的范围重试
   - 在此停止并首先解决阻塞
4. **始终生成部分报告** — 输出任何已完成的内容。绝不因为一个 agent 阻塞而丢弃工作。

常见阻塞：
- 输入文件缺失（story 未找到、GDD 缺失） → 重定向到创建它的 skill
- ADR 状态为 Proposed → 不要实现；先运行 `/architecture-decision`
- 范围太大 → 通过 `/create-stories` 拆分为两个 stories
- ADR 和 story 之间的冲突指令 → 展示冲突，不要猜测

## 文件写入协议

所有文件写入（设计文档、实现文件、测试用例）都委托给
通过 Task 生成的子 agent。每个子 agent 执行
"May I write to [path]?" 协议。此编排器不直接写入文件。

## 输出

涵盖以下内容的摘要报告：设计完成状态、每个团队成员的实现状态、测试结果，以及任何开放问题。

裁决：**COMPLETE** — 战斗功能已设计、实现和验证。
裁决：**BLOCKED** — 一个或多个阶段无法完成；已生成部分报告，列出未解决的项目。

## 后续步骤

- 在关闭 stories 之前对实现的战斗代码运行 `/code-review`。
- 运行 `/balance-check` 验证战斗公式和调优值。
- 如果需要 VFX、音频或性能打磨，运行 `/team-polish`。
