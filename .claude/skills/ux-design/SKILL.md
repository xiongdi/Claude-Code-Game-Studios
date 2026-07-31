---
name: ux-design
description: "Guided, section-by-section UX spec authoring for a screen, flow, or HUD. Reads game concept, player journey, and relevant GDDs to provide context-aware design guidance. Produces ux-spec.md (per screen/flow) or hud-design.md using the studio templates."
argument-hint: "[screen/flow name] or 'hud' or 'patterns'"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Task
model: sonnet
agent: ux-designer
---

当此 skill 被调用时：

## 1. 解析参数并确定模式

根据参数存在三种编写模式：

| 参数 | 模式 | 输出文件 |
|----------|------|-------------|
| `hud` | HUD 设计 | `design/ux/hud.md` |
| `patterns` | 交互模式库 | `design/ux/interaction-patterns.md` |
| 其他任意值（例如 `main-menu`、`inventory`） | 屏幕或流程的 UX 规范 | `design/ux/[argument].md` |
| 无参数 | 询问用户 |（见下文） |

**如果未提供参数**，不要失败 — 改为询问。使用 `AskUserQuestion`：
- "What are we designing today?"
  - 选项："A specific screen or flow (I'll name it)"、"The game HUD"、"The interaction pattern library"、"I'm not sure — help me figure it out"

如果用户选择 "I'll name it" 或输入屏幕名称，将其标准化为 kebab-case 作为文件名（例如 "Main Menu" 变为 `main-menu`）。

---

## 2. 收集上下文（读取阶段）

在向用户提问之前，先阅读所有相关上下文。此 skill 的价值在于带着信息到达。

### 2a: 必需读取

- **游戏概念**：读取 `design/gdd/game-concept.md` — 如果缺失，警告：
  > "No game concept found. Run `/brainstorm` first to establish the game's
  > foundation before designing UX."
  > 如果用户要求，继续执行。

### 2b: 玩家旅程

如果 `design/player-journey.md` 存在，读取它。对于每个相关部分，提取：
- 此屏幕出现在旅程的哪个阶段？
- 玩家到达此屏幕时的情绪状态是什么？
- 此屏幕在旅程中满足什么玩家需求？
- 此屏幕交付的关键时刻（来自旅程地图）是什么？

如果玩家旅程文件不存在，记录缺口并继续：
> "No player journey map found at `design/player-journey.md`. Designing without it
> means we'll be making assumptions about player context. Consider running a player
> journey session after this spec is drafted."

还要添加到 UX 规范的 Open Questions 部分：
> "Player journey map not yet created. Template available at `.claude/docs/templates/player-journey.md`. Run `/ux-design` Phase 2b or create it manually to establish player context for this screen."

### 2c: GDD UI 需求

Glob `design/gdd/*.md` 并 grep 查找 `UI Requirements` 部分。读取任何 UI Requirements 部分通过名称或类别引用此屏幕的 GDD。

这些 GDD UI 需求是此规范的 **需求输入**。将它们收集为规范必须满足的约束列表。

如果设计 HUD，读取所有 GDD UI Requirements 部分 — HUD 聚合来自每个系统的需求。

### 2d: 现有 UX 规范

Glob `design/ux/*.md` 并记录哪些屏幕已有规范。对于将链接到或从当前屏幕链接的屏幕，读取其导航/流程部分以找到此规范必须匹配的入口和出口点。

### 2e: 交互模式库

如果 `design/ux/interaction-patterns.md` 存在，读取模式目录索引（模式名称及其单行描述的列表）。不要读取完整的模式详情 — 只读目录。这告诉你哪些模式已经存在，以便引用它们而不是重新发明。

### 2f: Art Bible

检查 `design/art/art-bible.md`。如果找到，读取视觉方向部分。UX 布局必须与已经做出的美学承诺保持一致。

### 2g: 无障碍需求

检查 `design/accessibility-requirements.md`。如果找到，读取它。规范必须满足其中承诺的无障碍层级。

### 2h: 输入方式（来自项目配置）

读取 `.claude/docs/technical-preferences.md` 并提取 `## Input & Platform` 部分。存储这些值以供整个 skill 使用 — 它们驱动 Interaction Map 并告知无障碍需求：

- **Input Methods** — 例如 Keyboard/Mouse、Gamepad、Touch、Mixed
- **Primary Input** — 此游戏的主导输入
- **Gamepad Support** — Full / Partial / None
- **Touch Support** — Full / Partial / None
- **Target Platforms** — 用于安全区域和宽高比决策

如果该部分未配置（`[TO BE CONFIGURED]`），询问一次：
> "Input methods aren't configured yet. What does this game target?"
> 选项："Keyboard/Mouse only"、"Gamepad only"、"Both (PC + Console)"、"Touch (mobile)"、"All of the above"
>
> (Run `/setup-engine` to save this permanently so you won't be asked again.)

存储答案以供本会话的其余部分使用。不要每个部分或每个屏幕都询问。

### 2i: 展示上下文摘要

在任何设计工作之前，向用户展示简要摘要：

> **Designing: [Screen/Flow Name]**
> - Mode: [UX Spec / HUD Design / Pattern Library]
> - Journey phase(s): [from player-journey.md, or "unknown — no journey map"]
> - GDD requirements feeding this spec: [count and names, or "none found"]
> - Related screens already specced: [list, or "none yet"]
> - Known patterns available: [count, or "no pattern library yet"]
> - Accessibility tier: [from requirements doc, or "not yet defined"]
> - Input methods: [from technical-preferences.md, or "asked above"]

然后询问："Anything else I should read before we start, or shall we proceed?"

---

## 2b. 改造模式检测

在创建骨架之前，检查目标输出文件是否已存在。

Glob `design/ux/[filename].md`（其中 `[filename]` 是阶段 1 中解析的输出路径）。

**如果文件存在 — 改造模式：**
- 完整读取文件
- 对于每个预期部分，检查正文是否有真实内容（多于 `[To be designed]` 占位符）或为空/占位符
- 向用户展示部分状态摘要：

> "Found existing UX spec at `design/ux/[filename].md`. Here's what's already done:
>
> | Section | Status |
> |---------|--------|
> | Overview & Context | [Complete / Empty / Placeholder] |
> | Player Journey Integration | ... |
> | Screen Layout & Information Architecture | ... |
> | Interaction Model | ... |
> | Feedback & State Communication | ... |
> | Accessibility | ... |
> | Edge Cases & Error States | ... |
> | Open Questions | ... |
>
> I'll work on the [N] incomplete sections only — existing content will not be overwritten."

- 跳过第 3 部分（骨架创建）— 文件已存在
- 在阶段 4（部分编写）中，仅处理状态为 Empty 或 Placeholder 的部分
- 使用 `Edit` 就地填充占位符，而不是创建新骨架

**如果文件不存在 — 全新编写模式：**
正常进入阶段 3（创建文件骨架）。

---

## 3. 创建文件骨架

用户确认后，**立即**创建输出文件，包含空的部分标题。这确保增量写入有目标，且工作中断后仍能保留。

询问："May I create the skeleton file at `design/ux/[filename].md`?"

---

### UX Spec 的骨架（屏幕或流程）

```markdown
# UX Spec: [Screen/Flow Name]

> **Status**: In Design
> **Author**: [user + ux-designer]
> **Last Updated**: [today's date]
> **Journey Phase(s)**: [from context]
> **Template**: UX Spec

---

## Purpose & Player Need

[To be designed]

---

## Player Context on Arrival

[To be designed]

---

## Navigation Position

[To be designed]

---

## Entry & Exit Points

[To be designed]

---

## Layout Specification

### Information Hierarchy

[To be designed]

### Layout Zones

[To be designed]

### Component Inventory

[To be designed]

### ASCII Wireframe

[To be designed]

---

## States & Variants

[To be designed]

---

## Interaction Map

[To be designed]

---

## Events Fired

[To be designed]

---

## Transitions & Animations

[To be designed]

---

## Data Requirements

[To be designed]

---

## Accessibility

[To be designed]

---

## Localization Considerations

[To be designed]

---

## Acceptance Criteria

[To be designed]

---

## Open Questions

[To be designed]
```

---

### HUD 设计的骨架

```markdown
# HUD Design

> **Status**: In Design
> **Author**: [user + ux-designer]
> **Last Updated**: [today's date]
> **Template**: HUD Design

---

## HUD Philosophy

[To be designed]

---

## Information Architecture

### Full Information Inventory

[To be designed]

### Categorization

[To be designed]

---

## Layout Zones

[To be designed]

---

## HUD Elements

[To be designed]

---

## Dynamic Behaviors

[To be designed]

---

## Platform & Input Variants

[To be designed]

---

## Accessibility

[To be designed]

---

## Open Questions

[To be designed]
```

---

### 交互模式库的骨架

```markdown
# Interaction Pattern Library

> **Status**: In Design
> **Author**: [user + ux-designer]
> **Last Updated**: [today's date]
> **Template**: Interaction Pattern Library

---

## Overview

[To be designed]

---

## Pattern Catalog

[To be designed]

---

## Patterns

[Individual pattern entries added here as they are defined]

---

## Gaps & Patterns Needed

[To be designed]

---

## Open Questions

[To be designed]
```

---

写入骨架后，更新 `production/session-state/active.md`，包含：
- Task: Designing [screen/flow name] UX spec
- Current section: Starting (skeleton created)
- File: design/ux/[filename].md

---

## 4. 逐部分编写

按顺序遍历每个部分。对于 **每个部分**，遵循此循环：

```
Context  ->  Questions  ->  Options  ->  Decision  ->  Draft  ->  Approval  ->  Write
```

1. **Context**：说明此部分需要包含什么，并从阶段 2 收集的上下文中呈现任何相关约束。
2. **Questions**：询问起草此部分需要什么。对受限选择使用 `AskUserQuestion`，对开放式探索使用对话式文本。
3. **Options**：在设计选择存在的地方，提出 2-4 种方法及其优缺点。在对话中解释推理，然后使用 `AskUserQuestion` 捕获决策。
4. **Decision**：用户选择一种方法或提供自定义方向。
5. **Draft**：在对话中编写部分正文供审查。明确标记临时假设。
6. **Approval**：使用 `AskUserQuestion`：
   - "Does this capture the [section name] correctly?"
   - 选项："Yes — write it to the file"、"Small changes needed (describe below)"、"Major rethink needed"
   在用户选择 "Yes" 之前不要进入第 7 步。
7. **Write**：使用 `AskUserQuestion`："May I write the [section name] section to `[filepath]`?"
   - 选项："Yes, write it"、"Wait — one more change"
   确认后，使用 `Edit` 将 `[To be designed]` 占位符替换为批准的正文。

写入每个部分后，更新 `production/session-state/active.md`。

---

### 部分指南：UX Spec 模式

#### 部分 A: Purpose & Player Need

此部分是基础。所有其他决策都源自它。

**要问的问题**：
- "What player goal does this screen serve? What is the player trying to DO here?"
- "What would go wrong if this screen didn't exist or was hard to use?"
- "Complete this sentence: 'The player arrives at this screen wanting to ___.' "

交叉引用阶段 2 中收集的玩家旅程上下文。陈述的目的必须与旅程阶段和情绪状态保持一致。

---

#### 部分 B: Player Context on Arrival

**要问的问题**：
- "When in the game does a player first encounter this screen?"
- "What were they just doing immediately before reaching this screen?"
- "What emotional state should the design assume? (calm, stressed, curious, time-pressured)"
- "Do players arrive at this screen voluntarily, or are they sent here by the game?"

如果玩家旅程文档存在，提供将其映射到旅程阶段。

---

#### 部分 B2: Navigation Position

此屏幕在游戏导航层次结构中的位置是什么？这是一段式的定向地图 — 不是完整的流程图。

**要问的问题**：
- "Is this screen accessed from the main menu, from pause, from within gameplay, or from another screen?"
- "Is it a top-level destination (always reachable) or a context-dependent one (only accessible in certain states)?"
- "Can the player reach this screen from more than one place in the game?"

展示为："This screen lives at: [root] → [parent] → [this screen]" 加上任何替代入口路径。

---

#### 部分 B3: Entry & Exit Points

映射玩家可以到达和离开此屏幕的每一种方式。

**要问的问题**：
- "What are all the ways a player can reach this screen?"（列出每个触发器：按钮按下、游戏事件、从另一个屏幕重定向等）
- "What can the player do to exit? What happens when they do?"（返回按钮、确认操作、超时、游戏事件）
- "Are there any exits that are one-way — where the player cannot return to this screen without starting over?"

展示为两个表格：

| Entry Source | Trigger | Player carries this context |
|---|---|---|
| [screen/event] | [how] | [state/data they arrive with] |

| Exit Destination | Trigger | Notes |
|---|---|---|
| [screen/event] | [how] | [any irreversible state changes] |

---

#### 部分 C: Layout Specification

这是最大且最具交互性的部分。按子部分进行：

**子部分 1 — Information Hierarchy**（在任何布局之前建立）：
- 要求用户列出此屏幕必须传达的每条信息。
- 然后要求他们对项目进行排序："What is the single most important thing a player needs to see first? What is second? What can be discovered rather than immediately visible?"
- 在移动到区域之前，展示结果层次结构供批准。

**子部分 2 — Layout Zones**：
- 基于信息层次结构，提出粗略的屏幕区域（header、content area、action bar、sidebar 等）。
- 提供 2-3 种区域安排及其各自的推理。参考从游戏概念中收集的平台和输入上下文。
- 使用 `AskUserQuestion` 捕获选择：
  - "Which zone arrangement fits best?"
  - 选项：[你刚刚提出的 2-3 种命名安排] + "None — build a custom arrangement"

**子部分 3 — Component Inventory**：
- 对于每个区域，列出它包含的 UI 组件。对于每个组件，注明：
  - 组件类型（button、list、card、stat display、input field 等）
  - 它显示的内容
  - 它是否可交互
  - 如果它使用库中的现有模式（通过模式名称引用）
  - 如果它引入了新模式（标记以供后续添加到库中）

**子部分 4 — ASCII Wireframe**：
- 提供基于区域布局和组件列表生成 ASCII 线框图。
- 使用 `AskUserQuestion`："Want an ASCII wireframe as part of this spec?"
  - 选项："Yes, include one"、"No, I'll attach a separate file"
- 如果是，首先在对话中生成线框图。在写入文件之前征求反馈。

---

#### 部分 D: States & Variants

引导用户思考 happy path 之外的情况。

**要问的问题**（逐个进行）：
- "What does this screen look like the very first time a player sees it, when there is no data yet? (empty state)"
- "What happens when something goes wrong — an error, a failed action, a missing resource? (error state)"
- "Is there ever a loading wait on this screen? If so, what does it show? (loading state)"
- "Are there any player progression states that change what this screen shows? For example, locked content, premium content, or tutorial-mode overlays?"
- "Does this screen behave differently on any supported platform? (platform variant)"

将收集的状态展示为表格供批准：

| State / Variant | Trigger | What Changes |
|-----------------|---------|--------------|
| Default | Normal load | — |
| Empty | No data available | [content area description] |
| [etc.] | [trigger] | [changes] |

---

#### 部分 E: Interaction Map

对于布局规范中识别的每个可交互组件，定义：
- 操作（tap、click、press、hold、scroll、drag）
- 触发它的平台输入（mouse click、gamepad A、keyboard Enter）
- 即时反馈（visual、audio、haptic）
- 结果（导航目标、状态变更、数据写入）

使用阶段 2h 中从 `technical-preferences.md` 加载的输入方式 — 不要再次询问用户。提前说明："Mapping interactions for: [Input Methods from tech-prefs]. Covering [Gamepad Support] gamepad support."

逐个处理组件，而不是一次性询问所有。
对于导航操作（前往另一个屏幕），验证目标是否与现有 UX 规范匹配或将其标记为规范依赖。

---

#### 部分 E2: Events Fired

对于 Interaction Map 中的每个玩家操作，记录游戏或分析系统应触发的对应事件 — 或者如果没有适用的事件，明确注明 "no event"。

**要问的问题**：
- "For each action, should the game fire an analytics event, trigger a game-state change, or both?"
- "Are there any actions that should NOT fire an event — and is that a deliberate choice?"

展示为与 Interaction Map 并列的表格：

| Player Action | Event Fired | Payload / Data |
|---|---|---|
| [action] | [EventName] or none | [data passed with event] |

标记任何修改持久游戏状态（存档数据、进度、经济）的操作 — 这些需要架构团队的明确关注。

---

#### 部分 E3: Transitions & Animations

指定屏幕如何进入和退出，以及它如何响应状态变化。

**要问的问题**：
- "How does this screen appear? (fade in, slide from right, instant pop, scale from button)"
- "How does it dismiss? (fade out, slide back, cut)"
- "Are there any in-screen state transitions that need animation? (loading spinner, success state, error flash)"
- "Is there any animation that could cause motion sickness — and does the game have a reduced-motion option?"

最低要求：
- 屏幕进入过渡
- 屏幕退出过渡
- 如果屏幕有多个状态，至少一个状态变更动画

---

#### 部分 F: Data Requirements

交叉引用阶段 2 中收集的 GDD UI Requirements 部分。

对于屏幕显示的每条信息，询问：
- "Where does this data come from? Which system owns it?"
- "Does this screen need to write data back, or is it read-only?"
- "Is any of this data time-sensitive or real-time? (health bars, cooldown timers)"

将 UI 需要拥有或管理游戏状态的任何情况标记为架构问题。UX 规范定义 UI 需要什么；它们不规定数据如何传递。这是一个架构决策。

将数据需求展示为表格：

| Data | Source System | Read / Write | Notes |
|------|--------------|--------------|-------|
| [item] | [system] | Read | — |
| [item] | [system] | Write | [concern if any] |

---

#### 部分 G: Accessibility

如果存在，交叉引用 `design/accessibility-requirements.md`。

遍历 ux-designer agent 对此屏幕的标准检查清单：
- 通过所有交互元素的纯键盘导航路径
- 手柄导航顺序（如果适用）
- 文本对比度和最小可读字体大小
- 颜色独立传达（没有仅通过颜色传达的信息）
- 任何非文本元素的屏幕阅读器考虑
- 任何需要减少动作替代方案的动作或动画

如果此项目尚未定义无障碍层级，在 UX 规范的 Open Questions 部分记录缺口：
> "Accessibility tier not yet defined — consider WCAG-AA as a baseline. Run `/gate-check` to see whether this blocks any phase gates."
然后继续到下一部分，不要停止。

---

#### 部分 H: Localization Considerations

记录影响此屏幕在文本翻译时行为的约束。

**要问的问题**：
- "Which text elements on this screen are the longest? What is the maximum character count that fits the layout?"
- "Are there any elements where text length is layout-critical — e.g., a button label that must stay on one line?"
- "Are there any elements that display numbers, dates, or currencies that need locale-specific formatting?"

注意：旨在标记任何 40% 文本扩展（从英语翻译到德语或法语时常见）会破坏布局的元素。将这些标记为本地化工程师的 HIGH PRIORITY。

---

#### 部分 I: Acceptance Criteria

编写至少 5 个具体的、可测试的标准，QA 测试人员无需阅读任何其他设计文档即可验证。这些成为 `/story-done` 的通过/失败条件。

**格式**：使用复选框。每个标准必须可由人类测试人员验证：

```
- [ ] Screen opens within [X]ms from [trigger]
- [ ] [Element] displays correctly at [minimum] and [maximum] values
- [ ] [Navigation action] correctly routes to [destination screen]
- [ ] Error state appears when [condition] and shows [specific message or icon]
- [ ] Keyboard/gamepad navigation reaches all interactive elements in logical order
- [ ] [Accessibility requirement] is met — e.g., "all interactive elements have focus indicators"
```

**最低要求**：
- 1 个性能标准（加载/打开时间）
- 1 个导航标准（至少验证一个入口或出口路径）
- 1 个错误/空状态标准
- 1 个无障碍标准（按承诺的层级）
- 1 个此屏幕核心目的特有的标准

使用 `AskUserQuestion` 确认：
- "Do these acceptance criteria cover what would make this screen 'done' for your QA process?"
- 选项："Yes — these are solid"、"Add one more criterion"、"Remove or rephrase one"

---

### 部分指南：HUD 设计模式

HUD 设计遵循与 UX 规范模式不同的顺序。从哲学开始；在信息架构完成之前不要碰布局。

#### 部分 A: HUD Philosophy

要求用户用 1-2 句话描述游戏与屏幕信息的关系。

提供框架示例来帮助：
- "Nearly HUD-free — atmosphere requires unobstructed immersion (e.g., Hollow Knight, Firewatch)"
- "Minimal but present — only critical information visible, everything else contextual (e.g., Dark Souls)"
- "Information-dense — all decision-relevant data always visible (e.g., Diablo IV, StarCraft II)"
- "Adaptive — HUD density responds to combat state, exploration mode, menus (e.g., God of War)"

此哲学成为每个后续 HUD 设计决策的设计约束。如果提议的元素与陈述的哲学冲突，呈现该冲突。

---

#### 部分 B: Information Architecture

在任何布局工作之前完成此部分。不要跳过。

**步骤 1 — 完整信息清单**：
从阶段 2 中收集的 GDD UI Requirements 部分提取所有信息。
展示完整列表："These are all the things your game systems say they need to communicate to the player on screen."

**步骤 2 — 分类**：
对于每个项目，要求用户对其进行分类：

| 类别 | 描述 |
|----------|-------------|
| **Must Show** | 始终可见，玩家需要它来做核心决策 |
| **Contextual** | 仅在相关时可见（在战斗中、在可交互对象附近等） |
| **On Demand** | 玩家必须主动请求它（toggle、按住按钮） |
| **Hidden** | 通过世界/音频传达，从不在屏幕上显示文本 |

使用 `AskUserQuestion` 以 3-4 个为一组逐步进行，而不是一次性全部进行。
这是 HUD 中最具影响力的设计决策 — 不要匆忙。

**冲突检查**：如果信息哲学（部分 A）说 "nearly HUD-free" 但 Must Show 列表越来越长，明确呈现冲突：
> "The current Must Show list has [N] items. That may conflict with the HUD-free philosophy. Options: reduce the Must Show list, revise the philosophy, or define a hybrid approach where HUD is absent in exploration and present in combat."

---

#### 部分 C: Layout Zones

仅在信息架构批准后，设计布局区域。

基于以下因素进行布局：
- 哪些项目是 Must Show（它们驱动永久区域决策）
- 游戏过程中玩家注意力的自然走向（动作游戏的中心屏幕，策略游戏的角落）
- 平台和宽高比目标

提供 2-3 种区域安排。包括基于 HUD 哲学和部分 B 中的分类的推理。

---

#### 部分 D: HUD Elements

对于布局中的每个元素，指定：
- 元素名称和类别（Must Show / Contextual / On Demand）
- 显示的内容
- 视觉形式（bar、number、icon、counter、map）
- 更新行为（real-time、event-driven、player-queried）
- 上下文触发器（如果不是始终可见）
- 动画行为（低值时脉冲？淡入？猛冲进入？）

逐个元素进行处理。如果状态显示、资源条或冷却指示器存在相关模式，引用交互模式库。

---

#### 部分 E、F、G：Dynamic Behaviors、Platform Variants、Accessibility

这些遵循与 UX 规范等价部分相同的结构。参见 UX Spec 部分指南中的 D（States/Variants）、E（Interactions）和 G（Accessibility）。

对于 HUD 特别强调：
- Dynamic Behaviors：什么导致 HUD 在游戏过程中改变密度？
- Platform Variants：移动/主机是否需要不同的元素大小或位置？

---

### 部分指南：交互模式库模式

模式库编写是增量和目录驱动的，不是线性的。

#### 阶段 1：编目现有模式

Glob `design/ux/*.md`（不包括 `interaction-patterns.md`）并读取每个规范的 Component Inventory 和 Interaction Map 部分。提取使用的每个交互模式。

展示提取的列表："Based on existing UX specs, these patterns are already in use in the game:"
- [Pattern name]: used in [screen], [screen]
- [etc.]

询问："Are there patterns you know exist but aren't in existing specs yet? List any additional ones now."

---

#### 阶段 2：形式化每个模式

对于每个模式（现有的或新的），记录：

```markdown
### [Pattern Name]

**Category**: Navigation / Input / Feedback / Data Display / Modal / Overlay / [other]
**Used In**: [list of screens]

**Description**: [One paragraph explaining what this pattern is and when to use it]

**Specification**:
- [Component behavior]
- [Input mapping]
- [Visual/audio feedback]
- [Accessibility requirements for this pattern]

**When to Use**: [Conditions where this pattern is appropriate]
**When NOT to Use**: [Conditions where another pattern is more appropriate]

**Reference**: [Screenshot path or ASCII example, if available]
```

以组为单位处理模式。使用 `AskUserQuestion`：
- "How do you want to work through these patterns?"
- 选项："Draft the first batch from existing specs (faster)"、"Define them one by one (more control)"、"Start with the most-used pattern first"

---

#### 阶段 3：识别缺口

在编目已知模式后，询问：
- "Are there screens or interactions planned that would need patterns not yet in this library?"
- "Are there any patterns in existing specs that feel inconsistent with each other and should be consolidated?"

在 Gaps 部分记录缺口以供后续跟进。

---

## 5. 交叉引用检查

在标记规范为准备好审查之前，运行这些检查：

**1. GDD 需求覆盖**：每个引用此屏幕的 GDD UI Requirement 在此规范中是否有对应元素？呈现任何缺口。

**2. 模式库一致性**：此规范中使用的所有交互模式是否都通过名称引用？如果在此规范会话期间发明了新模式，将其标记为添加到模式库：
使用 `AskUserQuestion`：
- "This spec uses [pattern name], which isn't in the pattern library yet. What should we do?"
- 选项："Add it to the pattern library now"、"Flag it as a gap and continue"、"Skip — this pattern is one-off"

**3. 导航一致性**：此规范中的入口/出口点是否与任何相关规范中的导航地图匹配？标记不匹配。

**4. 无障碍覆盖**：规范是否解决了 `design/accessibility-requirements.md` 中承诺的无障碍层级？如果没有，标记未解决的问题。

**5. 空状态**：每个数据依赖元素是否都有定义的空状态？标记任何没有的。

展示检查结果：
> **Cross-Reference Check: [Screen Name]**
> - GDD requirements: [N of M covered / all covered]
> - New patterns to add to library: [list or "none"]
> - Navigation mismatches: [list or "none"]
> - Accessibility gaps: [list or "none"]
> - Missing empty states: [list or "none"]

---

## 6. 交接

当所有部分都已批准并写入时：

### 6a: 更新会话状态

更新 `production/session-state/active.md`，包含：
- Task: [screen-name] UX spec
- Status: Complete (或 In Review)
- File: design/ux/[filename].md
- Sections: All written
- Next: [suggestion]

### 6b: 建议下一步

在展示选项之前，清楚说明：

> "This spec should be validated with `/ux-review` before it enters the implementation pipeline. The Pre-Production gate requires all key screen specs to have a review verdict."

然后使用 `AskUserQuestion`：
- "Run `/ux-review [filename]` now, or do something else first?"
  - 选项：
    - "Run `/ux-review` now — validate this spec"
    - "Design another screen first, then review all specs together"
    - "Update the interaction pattern library with new patterns from this spec"
    - "Stop here for this session"

如果用户选择 "Design another screen first"，添加一条注意："Reminder: run `/ux-review` on all completed specs before running `/gate-check pre-production`."

### 6c: 交叉链接相关规范

如果其他 UX 规范链接到此屏幕或从到此屏幕链接，注明哪些应该引用此规范。不要编辑这些文件而不询问 — 只列出它们的名字。

---

## 7. 恢复与继续

如果会话被中断（压缩、崩溃、新会话）：

1. 读取 `production/session-state/active.md` — 它记录当前屏幕和哪些部分已完成。
2. 读取 `design/ux/[filename].md` — 有真实内容的部分已完成；带有 `[To be designed]` 的部分仍需工作。
3. 从下一个未完成的部分继续 — 无需重新讨论已完成的部分。

这就是增量编写重要的原因：每个批准的部分在任何中断后都能保留。

---

## 8. 专家 Agent 路由

此 skill 使用 `ux-designer` 作为主要 agent（在 frontmatter 中设置）。对于特定子主题，可能需要额外的上下文或协调：

| 主题 | 协调对象 |
|-------|----------------|
| 视觉美学、颜色、布局感觉 | `art-director` — UX 规范定义区域；art 定义它们的外观 |
| 实现可行性（引擎约束） | `ui-programmer` — 在最终确定组件清单之前 |
| 游戏数据需求 | `game-designer` — 当数据所有权不明确时 |
| 叙事/传说中在 UI 中可见的内容 | `narrative-director` — 用于 flavor text、物品名称、传说面板 |
| 无障碍层级决策 | 由此会话处理 — 由 ux-designer 拥有 |

通过 Task 工具委托给另一个 agent 时：
- 提供：屏幕名称、游戏概念摘要、需要专家输入的具体问题
- agent 将分析返回给此会话
- 此会话向用户展示 agent 的输出
- 用户决定；此会话写入文件
- Agent 不直接写入文件 — 此会话拥有所有文件写入

---

## 协作协议

此 skill 在每一步都遵循协作设计原则：

1. 每个部分都遵循 **Question -> Options -> Decision -> Draft -> Approval**
2. 在每个决策点使用 `AskUserQuestion`（Explain -> Capture 模式）：
   - 阶段 2："Ready to start, or need more context?"
   - 阶段 3："May I create the skeleton?"
   - 阶段 4（每个部分）：设计问题、方法选项、草稿批准
   - 阶段 5："Run cross-reference check? What's next?"
3. 在骨架之前和每个部分写入之前使用 **"May I write to [filepath]?"**
4. **增量编写**：每个部分在批准后立即写入文件
5. **会话状态更新**：每次部分写入后

**美学遵从**：当布局或视觉选择归结为个人品味时，提出选项并询问。不要因为某个布局是"标准的"就选择它 — 始终确认。用户是创意总监。

**冲突呈现**：当 GDD 需求与可用屏幕空间冲突时，呈现冲突并提出解决选项。永远不要悄悄丢弃需求。永远不要悄悄扩展布局而不标记它。

**永远不要**自动生成完整规范并作为既成事实呈现。
**永远不要**在未经用户批准的情况下写入部分。
**永远不要**与现有已批准的 UX 规范矛盾而不标记冲突。
**始终**展示决策的来源（GDD 需求、玩家旅程、用户选择）。

结论：**COMPLETE** — UX 规范已编写并逐部分批准。

---

## 推荐的下一步

- 运行 `/ux-review [filename]` 在规范进入实现流水线之前验证它
- 运行 `/ux-design [next-screen]` 继续设计剩余的屏幕或流程
- 一旦所有关键屏幕都有批准的 UX 规范，运行 `/gate-check pre-production`
