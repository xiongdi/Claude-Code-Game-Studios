---
name: team-ui
description: "Orchestrate the UI team through the full UX pipeline: from UX spec authoring through visual design, implementation, review, and polish. Integrates with /ux-design, /ux-review, and studio UX templates."
argument-hint: "[UI feature description] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
model: sonnet
---
当此 skill 被调用时，通过结构化流水线编排 UI 团队。

**决策点：** 在每个阶段转换时，使用 `AskUserQuestion` 向用户展示 subagent 的提案作为可选项。先在对话中写出 agent 的完整分析，然后用简洁的标签捕获决策。用户必须批准才能进入下一阶段。

## 阶段 0：解析审查模式

1. 如果传入了 `--review [mode]` 参数，使用该模式。
2. 否则读取 `production/review-mode.txt` — 使用其中写入的内容。
3. 否则默认为 `lean`。

模式：
- `full` — 生成所有 director 和 lead 门控，如下所述
- `lean` — 跳过 director 门控，除非它们是 PHASE-GATE 类型（CD-PHASE-GATE, TD-PHASE-GATE, PR-PHASE-GATE, AD-PHASE-GATE）
- `solo` — 完全跳过所有 director 门控生成；在没有任何 agent 门控的情况下运行 skill

存储解析后的模式供所有后续阶段使用。

**director 门控跳过规则**：在生成 creative-director、art-director 或任何其他 Tier 1/2 director 进行审查之前（PHASE-GATE 触发之外），应用解析后的模式：如果是 solo 模式则跳过；如果是 lean 模式且这不是 PHASE-GATE 则跳过。

## 团队组成
- **ux-designer** — 用户流程、线框图、无障碍、输入处理
- **ui-programmer** — UI 框架、屏幕、组件、数据绑定、实现
- **art-director** — 视觉风格、布局打磨、与 art bible 一致性
- **engine UI specialist** — 根据引擎特定最佳实践验证 UI 实现模式（从 `.claude/docs/technical-preferences.md` Engine Specialists → UI Specialist 读取）
- **accessibility-specialist** — 在阶段 4 审计无障碍合规性

**此流水线使用的模板：**
- `ux-spec.md` — 标准屏幕/流程 UX 规范
- `hud-design.md` — HUD 专用 UX 规范
- `interaction-pattern-library.md` — 可复用的交互模式
- `accessibility-requirements.md` — 承诺的无障碍层级和要求

## 如何委托

使用 Task 工具将每个团队成员生成为 subagent：
- `subagent_type: ux-designer` — 用户流程、线框图、无障碍、输入处理
- `subagent_type: ui-programmer`  — UI 框架、屏幕、组件、数据绑定
- `subagent_type: art-director` — 视觉风格、布局打磨、art bible 一致性
- `subagent_type: [UI engine specialist]` — 引擎特定 UI 模式验证（例如 unity-ui-specialist、ue-umg-specialist、godot-specialist）
- `subagent_type: accessibility-specialist` — 无障碍合规性审计

始终在每个 agent 的提示中提供完整上下文（功能需求、现有 UI 模式、平台目标）。在流水线允许的地方并行启动独立的 agent（例如阶段 4 的审查 agent 可以同时运行）。

## 流水线

### 阶段 1a：上下文收集

在设计任何东西之前，阅读并综合：
- `design/gdd/game-concept.md` — 平台目标和预期受众
- `design/player-journey.md` — 玩家到达此屏幕时的状态和上下文
- 与此功能相关的所有 GDD UI Requirements 部分
- `design/ux/interaction-patterns.md` — 要复用的现有模式（不是重新发明）
- `design/accessibility-requirements.md` — 承诺的无障碍层级（例如 Basic、Enhanced、Full）

**如果 `design/ux/interaction-patterns.md` 不存在**，立即暴露缺口：
> "interaction-patterns.md does not exist — no existing patterns to reuse."

然后使用 `AskUserQuestion` 提供选项：
- (a) 先运行 `/ux-design patterns` 建立模式库，然后继续
- (b) 在没有模式库的情况下继续 — ui-programmer 将把所有创建的模式视为新的，并在完成后将每个模式添加到新的 `design/ux/interaction-patterns.md`

不要仅从功能名称或 GDD 中发明或假设模式。如果用户选择 (b)，在阶段 3 明确指示 ui-programmer 将所有模式视为新的，并在实现完成时将它们记录在 `design/ux/interaction-patterns.md` 中。在最终总结报告中注明模式库状态（已创建 / 缺失 / 已更新）。

为 ux-designer 在简报中总结上下文：玩家在做什么、他们需要什么、适用哪些约束、哪些现有模式是相关的。

### 阶段 1b：UX 规范编写

调用 `/ux-design [feature name]` skill 或直接委托给 ux-designer 按照 `ux-spec.md` 模板生成 `design/ux/[feature-name].md`。

如果设计 HUD，使用 `hud-design.md` 模板而不是 `ux-spec.md`。

> **特殊情况说明：**
> - 对于 HUD 设计，用 `argument: hud` 调用 `/ux-design`（例如 `/ux-design hud`）。
> - 对于交互模式库，在项目开始时运行一次 `/ux-design patterns`，并在后续阶段引入新模式时更新它。

输出：`design/ux/[feature-name].md`，所有必需规范部分已填写。

### 阶段 1c：UX 审查

规范完成后，调用 `/ux-review design/ux/[feature-name].md`。

**门控**：在结论为 APPROVED 之前不要进入阶段 2。如果结论是 NEEDS REVISION，ux-designer 必须解决标记的问题并重新运行审查。用户可以明确接受 NEEDS REVISION 风险并继续，但这必须是一个有意识的决定 — 在询问是否继续之前，通过 `AskUserQuestion` 呈现具体问题。

### 阶段 2：视觉设计

委托给 **art-director**：
- 审查完整的 UX 规范（流程、线框图、交互模式、无障碍说明）— 不仅仅是线框图图像
- 从 art bible 应用视觉处理：颜色、排版、间距、动画风格
- 检查视觉设计保持无障碍合规性：验证颜色对比度，确认颜色从来不是状态的唯一指示器（形状、文本或图标必须强化它）
- 指定从美术流水线需要的所有资产需求：指定尺寸的图标、背景纹理、字体、装饰元素 — 带有精确的尺寸和格式要求
- 确保与现有已实现的 UI 屏幕的一致性
- 输出：带样式说明和资产清单的视觉设计规范

### 阶段 3：实现

在实现开始之前，生成 **engine UI specialist**（从 `.claude/docs/technical-preferences.md` Engine Specialists → UI Specialist 读取）来审查 UX 规范和视觉设计规范以获取引擎特定实现指导：
- 此屏幕应使用哪个引擎 UI 框架？（例如 Unity 中的 UI Toolkit vs UGUI，Godot 中的 Control nodes vs CanvasLayer，Unreal 中的 UMG vs CommonUI）
- 提议的布局或交互模式是否有任何引擎特定的注意事项？
- 引擎推荐的 widget/node 结构？
- 输出：在 ui-programmer 开始之前交给他们的引擎 UI 实现说明

如果未配置引擎，跳过此步骤。

委托给 **ui-programmer**：
- 按照 UX 规范和视觉设计规范实现 UI
- **使用 `design/ux/interaction-patterns.md` 中的模式** — 不要重新发明已指定的模式。如果某个模式几乎符合但需要修改，注明偏差并标记给 ux-designer 审查。
- **UI 永远不拥有或修改游戏状态** — 仅显示；为所有玩家操作发出事件
- 所有文本通过本地化系统 — 没有硬编码的面向玩家的字符串
- 支持两种输入方式（键盘/鼠标和手柄）
- 按照 `design/accessibility-requirements.md` 中承诺的层级实现无障碍功能
- 绑定数据到游戏状态
- **如果在实现过程中创建了新的交互模式**（即模式库中尚未存在的内容），在标记实现完成之前将其添加到 `design/ux/interaction-patterns.md`
- 输出：已实现的 UI 功能

### 阶段 4：审查（并行）

并行委托：
- **ux-designer**：验证实现与线框图和交互规范匹配。测试纯键盘和纯手柄导航。检查无障碍功能正常运行。
- **art-director**：验证与 art bible 的视觉一致性。检查最低和最高支持的分辨率。
- **accessibility-specialist**：验证符合 `design/accessibility-requirements.md` 中记录的无障碍承诺层级。将任何违规标记为阻塞项。

所有三个审查流必须在进入阶段 5 之前报告。

### 阶段 5：打磨

- 处理所有审查反馈
- 验证动画可跳过并尊重玩家的动作减少偏好
- 确认 UI 声音通过音频事件系统触发（无直接音频调用）
- 在所有支持的分辨率和宽高比下测试
- **验证 `design/ux/interaction-patterns.md` 是最新的** — 如果在此功能的实现过程中引入了任何新模式，确认它们已添加到库中
- **确认所有 HUD 元素遵守 `design/ux/hud.md` 中定义的视觉预算**（元素数量、屏幕区域分配、最大不透明度值）

## 快速参考 — 何时使用哪个 Skill

- `/ux-design` — 从头开始为屏幕、流程或 HUD 编写新的 UX 规范
- `/ux-review` — 在实现之前验证已完成的 UX 规范
- `/team-ui [feature]` — 从概念到打磨的完整流水线（内部调用 `/ux-design` 和 `/ux-review`）
- `/quick-design` — 不需要全新 UX 规范的小型 UI 更改

## 错误恢复协议

如果任何生成的 agent（通过 Task）返回 BLOCKED、错误或无法完成：

1. **立即暴露**：在继续到依赖阶段之前向用户报告 "[AgentName]: BLOCKED — [原因]"
2. **评估依赖**：检查被阻塞 agent 的输出是否是后续阶段所需的。如果是，未经用户输入不要超过该依赖点。
3. **提供选项** 通过 AskUserQuestion 选择：
   - 跳过此 agent 并在最终报告中注明缺口
   - 用更窄的范围重试
   - 停在这里先解决阻塞项
4. **始终生成部分报告** — 输出任何已完成的内容。永远不要因为一个 agent 阻塞而丢弃工作。

常见阻塞项：
- 输入文件缺失（未找到 story，GDD 不存在） → 重定向到创建它的 skill
- ADR 状态为 Proposed → 不要实现；先运行 `/architecture-decision`
- 范围太大 → 通过 `/create-stories` 拆分为两个 story
- ADR 和 story 之间的指令冲突 → 暴露冲突，不要猜测

## 文件写入协议

所有文件写入（UX 规范、交互模式库更新、实现文件）都委托给 sub-agent 和 sub-skill（`/ux-design`、`ui-programmer`）。每个都执行 "May I write to [path]?" 协议。此编排器不直接写入文件。

## 输出

一份总结报告，涵盖：UX 规范状态、UX 审查结论、视觉设计状态、实现状态、无障碍合规性、输入方式支持、交互模式库更新状态和任何未解决的问题。

结论：**COMPLETE** — UI 功能通过完整流水线交付（UX 规范 → 视觉 → 实现 → 审查 → 打磨）。
结论：**BLOCKED** — 流水线已停止；在停止之前暴露阻塞项及其阶段。

## 下一步

- 如果尚未批准，对最终规范运行 `/ux-review`。
- 在关闭 story 之前对 UI 实现运行 `/code-review`。
- 如果需要视觉或音频打磨，运行 `/team-polish`。
