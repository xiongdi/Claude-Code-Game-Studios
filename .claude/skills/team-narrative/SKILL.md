---
name: team-narrative
description: "编排叙事团队：协调 narrative-director、writer、world-builder 和 level-designer，创建连贯的故事内容、世界传说和叙事驱动的关卡设计。"
argument-hint: "[narrative content description] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Task, AskUserQuestion, TodoWrite
model: sonnet
---
如果没有提供参数，输出用法指导并退出，不生成任何 agent：
> Usage: `/team-narrative [narrative content description]` — 描述要工作的故事内容、场景或叙事区域（如 `boss encounter cutscene`、`faction intro dialogue`、`tutorial narrative`）。此处不要使用 `AskUserQuestion`；直接输出指导。

当此 skill 带参数调用时，通过结构化管线编排叙事团队。

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
- **narrative-director** — 故事弧线、角色设计、对话策略、叙事愿景
- **writer** — 对话写作、传说条目、物品描述、游戏内文本
- **world-builder** — 世界规则、派系设计、历史、地理、环境叙事
- **art-director** — 角色视觉设计、环境视觉叙事、过场动画/电影基调
- **level-designer** — 服务于叙事的关卡布局、节奏、环境叙事节拍
- **localization-lead** — 本地化准备 — 标记不可本地化的字符串、文化假设和 i18n 缺口

## 如何委托

使用 Task 工具将每个团队成员生成为子 agent：
- `subagent_type: narrative-director` — 故事弧线、角色设计、叙事愿景
- `subagent_type: writer` — 对话写作、传说条目、游戏内文本
- `subagent_type: world-builder` — 世界规则、派系设计、历史、地理
- `subagent_type: art-director` — 角色视觉档案、环境视觉叙事、电影基调
- `subagent_type: level-designer` — 服务于叙事的关卡布局、节奏
- `subagent_type: localization-lead` — 本地化准备 — 标记不可本地化的字符串、文化假设和 i18n 缺口

始终在每个 agent 的提示中提供完整上下文（叙事简报、传说依赖、角色档案）。在管线允许的地方并行启动独立 agent（如 Phase 2 的 agent 可以同时运行）。

## 管线

### Phase 1: 叙事指导
委托给 **narrative-director**：
- 定义此内容的叙事目的：它服务于什么故事节拍？
- 识别涉及的角色、他们的动机，以及这如何融入整体弧线
- 设定情感基调和节奏目标
- 指定任何传说依赖或此引入的新传说
- 输出：带有故事需求的叙事简报

### Phase 2: 世界基础（并行）
并行委托 — 在等待任何结果之前同时发出所有三个 Task 调用：
- **world-builder**：创建或更新与此内容相关的派系、地点和历史的传说条目。对照现有传说检查矛盾。为新条目设定正典级别。
- **writer**：使用角色语音档案起草角色对话。确保所有台词在 120 字符以内，对变量使用命名占位符，并准备好本地化。
- **art-director**：定义此内容中出现的关键角色的视觉设计方向（轮廓、视觉原型、区分特征）。为每个关键空间指定环境视觉叙事元素（道具布置、灯光说明、空间排列）。为任何过场动画或脚本序列定义基调调色板和电影方向。

### Phase 3: 关卡叙事集成
委托给 **level-designer**：
- 审查叙事简报和传说基础
- 在关卡中设计环境叙事元素
- 放置叙事触发器、对话区和发现点
- 确保节奏同时服务于游戏和故事

### Phase 4: 审查和一致性
委托给 **narrative-director**：
- 对照角色语音档案审查所有对话
- 验证新旧条目之间的传说一致性
- 确认叙事节奏与关卡设计对齐
- 检查所有谜题都有记录的"真相"

### Phase 5: 打磨（并行）
并行委托：
- **writer**：最终自审 — 验证没有台词超出对话框限制、所有文本使用字符串键（非原始字符串）、占位符变量名一致
- **localization-lead**：验证 i18n 合规性 — 检查字符串键命名约定、标记任何带有硬编码格式的字符串（这些格式在翻译后无法存活）、验证对于会扩展的语言（德语/芬兰语通常 +30%）的字符限制余量、确认文本中没有需要区域专属变体的文化假设
- **world-builder**：最终确定所有新传说条目的正典级别

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

所有文件写入（叙事文档、对话文件、传说条目）都委托给
通过 Task 生成的子 agent。每个子 agent 执行"May I write to [path]?"
协议。此编排器不直接写入文件。

## 输出

涵盖以下内容的摘要报告：叙事简报状态、创建/更新的传说条目、编写的对话行、关卡叙事集成点、一致性审查结果，以及任何未解决的矛盾。

裁决：**COMPLETE** — 叙事内容已交付。

如果管线因依赖关系未解决而停止（如传说矛盾或用户未解决的缺失前置条件）：

裁决：**BLOCKED** — [原因]

## 后续步骤

- 运行 `/design-review` 对叙事文档进行一致性验证。
- 对话最终确定后运行 `/localize extract` 提取新字符串供翻译。
- 运行 `/dev-story` 在游戏中实现对话触发器和叙事事件。
