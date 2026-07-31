---
name: team-level
description: "编排关卡设计团队：level-designer + narrative-director + world-builder + art-director + systems-designer + qa-tester，完成区域/关卡创建。"
argument-hint: "[level name or area to design] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
model: sonnet
---

当此 skill 被调用时：

**决策点：** 在每个步骤转换时，使用 `AskUserQuestion` 向用户
展示子 agent 的提案作为可选项。在对话中写入 agent 的
完整分析，然后用简洁标签捕获决策。
用户必须批准才能进入下一步。

## Phase 0: 解析 Review 模式

1. 如果传入了 `--review [mode]` 作为参数，使用该模式。
2. 否则读取 `production/review-mode.txt` — 使用那里写的内容。
3. 否则默认为 `lean`。

模式：
- `full` — 按所述生成所有 director 和 lead gates
- `lean` — 跳过 director gates，除非它们是 PHASE-GATE 类型（CD-PHASE-GATE, TD-PHASE-GATE, PR-PHASE-GATE, AD-PHASE-GATE）
- `solo` — 完全跳过所有 director gate 生成；在没有任何 agent gates 的情况下运行 skill

存储解析后的模式供所有后续阶段使用。

1. **读取参数**了解目标关卡或区域（如 `tutorial`、
   `forest dungeon`、`hub town`、`final boss arena`）。

2. **收集上下文**：
   - 读取 `design/gdd/game-concept.md` 中的游戏概念
   - 读取 `design/gdd/game-pillars.md` 中的游戏支柱
   - 读取 `design/levels/` 中的现有关卡文档
   - 读取 `design/narrative/` 中的相关叙事文档
   - 阅读该区域/派系的世界构建文档

## 如何委托

使用 Task 工具将每个团队成员生成为子 agent：
- `subagent_type: narrative-director` — 叙事目的、角色、情感弧线
- `subagent_type: world-builder` — 传说背景、环境叙事、世界规则
- `subagent_type: level-designer` — 空间布局、节奏、遭遇、导航
- `subagent_type: systems-designer` — 敌人组合、战利品表、难度平衡
- `subagent_type: art-director` — 视觉主题、调色板、灯光、资产需求
- `subagent_type: accessibility-specialist` — 导航清晰度、色盲安全、认知负荷
- `subagent_type: qa-tester` — 测试用例、边界测试、playtest 检查清单

始终在每个 agent 的提示中提供完整上下文（游戏概念、支柱、现有关卡文档、叙事文档）。

3. **按顺序编排关卡设计团队**：

### 步骤 1：叙事 + 视觉指导（narrative-director + world-builder + art-director，并行）

同时生成所有三个 agent — 在等待任何结果之前发出所有三个 Task 调用。

生成 `narrative-director` agent 来：
- 定义此区域的叙事目的（这里发生什么故事节拍？）
- 识别关键角色、对话触发器和传说元素
- 指定情感弧线（玩家进入、期间、离开时应该如何感受？）

生成 `world-builder` agent 来：
- 提供该区域的传说背景（历史、派系存在、生态）
- 定义环境叙事机会
- 指定任何影响该区域游戏的世界规则

生成 `art-director` agent 来：
- 为此区域建立视觉主题目标 — 这些是布局的输入，而非输出
- 定义该区域的色温和灯光氛围（它与相邻区域有何不同？）
- 指定形状语言方向（棱角分明的堡垒？有机洞穴？破败的宏伟？）
- 命名将引导玩家的主要视觉地标
- 如果存在，阅读 `design/art/art-bible.md` — 将所有指导锚定在已建立的 art bible 中

**步骤 1 中 art-director 的视觉目标必须作为明确约束传递给步骤 2 中的 level-designer**。布局决策在视觉指导范围内进行，而非在其之前。

**关卡**：使用 `AskUserQuestion` 展示所有三个步骤 1 输出（叙事简报、传说基础、视觉指导目标）并在继续到步骤 2 之前确认。

### 步骤 2：布局和遭遇设计（level-designer）
以完整的步骤 1 输出作为上下文生成 `level-designer` agent：
- 叙事简报（来自 narrative-director）
- 传说基础（来自 world-builder）
- **视觉指导目标（来自 art-director）** — 布局必须在这些目标范围内工作，而非与之矛盾

level-designer 应该：
- 设计空间布局（关键路径、可选路径、秘密） — 确保主要路线与步骤 1 中的视觉地标目标对齐
- 定义节奏曲线（紧张高峰、休息区、探索区） — 与 narrative-director 的情感弧线协调
- 放置有难度递进的遭遇
- 设计环境谜题或导航挑战
- 定义用于导航的兴趣点和地标 — 这些必须与 art-director 指定的视觉地标匹配
- 指定入口/出口点以及与相邻区域的连接

**相邻区域依赖检查**：布局生成后，检查 `design/levels/` 中 level-designer 引用的每个相邻区域。如果任何引用区域的 `.md` 文件不存在，展示缺口：
> "Level references [area-name] as an adjacent area but `design/levels/[area-name].md` does not exist."

使用 `AskUserQuestion`，选项为：
- (a) 使用占位符引用继续 — 在关卡文档中将连接标记为 UNRESOLVED 并在摘要报告的开放跨关卡依赖章节中列出
- (b) 暂停并首先运行 `/team-level [area-name]` 来建立该区域

不要为缺失的相邻区域编造内容。

**关卡**：使用 `AskUserQuestion` 展示步骤 2 布局（包括任何未解决的相邻区域依赖）并在继续到步骤 3 之前确认。

### 步骤 3：系统集成（systems-designer）
生成 `systems-designer` agent 来：
- 指定敌人组合和遭遇公式
- 定义战利品表和奖励放置
- 相对于预期玩家等级/装备平衡难度
- 设计任何区域专属机制或环境危害
- 指定资源分布（生命拾取、存档点、商店）

**关卡**：使用 `AskUserQuestion` 展示步骤 3 输出并在继续到步骤 4 之前确认。

### 步骤 4：制作概念 + 无障碍（art-director + accessibility-specialist，并行）

**注意**：art-director 的指导性通过（视觉主题、色彩目标、氛围）发生在步骤 1。此通过是特定于位置的制作概念 — 给定最终布局，每个特定空间看起来像什么？

以步骤 2 的最终布局生成 `art-director` agent：
- 为关键空间（入口、关键遭遇区、地标、出口）生成特定位置的概念规范
- 指定哪些美术资产是此区域独有的，哪些来自全局池共享
- 定义每个关键空间的视线和灯光设置（这些现在是布局知情的，而非指导性的）
- 指定该区域布局专属的 VFX 需求（天气体积、粒子、大气效果）
- 标记布局与步骤 1 目标产生视觉指导冲突的任何位置 — 将这些作为制作风险展示

并行生成 `accessibility-specialist` agent 来：
- 审查关卡布局的导航清晰度（玩家能否不依赖颜色自行定位？）
- 检查关键路径标识除了颜色外还使用形状/图标/声音提示
- 审查任何谜题机制的认知负荷 — 标记任何需要保持超过 3 个同时状态的内容
- 检查关键游戏区域对色盲玩家有足够的对比度
- 输出：无障碍问题列表，附严重程度（BLOCKING / RECOMMENDED / NICE TO HAVE）

在继续之前等待两个 agent 返回。

**关卡**：使用 `AskUserQuestion` 展示两个步骤 4 结果。如果 accessibility-specialist 返回了任何 BLOCKING 问题，突出显示它们并提供：
- (a) 返回 level-designer 和 art-director 重新设计被标记的元素，然后进入步骤 5
- (b) 记录为已知的无障碍缺口并继续到步骤 5，在最终报告中明确记录该问题

未经用户确认任何 BLOCKING 无障碍问题，不要继续到步骤 5。

### 步骤 5：QA 规划（qa-tester）
生成 `qa-tester` agent 来：
- 为关键路径编写测试用例
- 识别边界和边界情况（序列破坏、软锁）
- 为该区域创建 playtest 检查清单
- 定义关卡完成的验收标准

4. **编译关卡设计文档**，将所有团队输出组合为
   关卡设计模板格式。

收集所有子 agent 输出后，通过 Task 生成 `level-designer` 来编译和写入最终文档：
- 传递：所有子 agent 输出（逐字）、关卡简报、游戏支柱、相关 GDD 章节
- 要求 level-designer：编译为关卡设计文档格式，然后在写入前请求用户批准（"May I write the compiled level design to design/levels/[level-name].md?"）
- 编排器不直接为最终文档调用 Write。

5. **保存到** `design/levels/[level-name].md`（由 level-designer 子 agent 在用户批准后处理 — 见上文）。

6. **输出摘要**，包括：区域概览、遭遇数量、估算资产
   列表、叙事节拍、任何跨团队依赖或开放问题、开放
   跨关卡依赖（引用但尚未设计的相邻区域，每个
   标记为 UNRESOLVED），以及无障碍问题及其解决状态。

## 文件写入协议

所有文件写入（关卡设计文档、叙事文档、测试检查清单）都委托给
通过 Task 生成的子 agent。每个子 agent 执行"May I write to [path]?"
协议。此编排器不直接写入文件。

裁决：**COMPLETE** — 关卡设计文档已生成，所有团队输出已编译。
裁决：**BLOCKED** — 一个或多个 agent 阻塞；已生成部分报告，列出未解决的项目。

## 后续步骤

- 运行 `/design-review design/levels/[level-name].md` 验证完成的关卡设计文档。
- 设计批准后运行 `/dev-story` 实现关卡内容。
- 运行 `/qa-plan` 为此关卡生成 QA 测试计划。

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
