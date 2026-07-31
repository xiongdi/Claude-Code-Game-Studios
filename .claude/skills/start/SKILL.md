---
name: start
description: "首次使用引导 — 询问你当前的位置，然后引导你进入正确的工作流。不做任何假设。"
argument-hint: "[no arguments]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion
model: sonnet
---

# 引导式入门

此 skill 写入一个文件：`production/review-mode.txt`（review 模式配置在 Phase 3b 中设置）。

此 skill 是新用户的入口点。它不假设你有游戏想法、引擎偏好或任何先前经验。它先询问，然后引导你到正确的工作流。

---

## Phase 1: 检测项目状态

在询问任何内容之前，静默收集上下文以便定制指导。不要主动展示这些结果 — 它们为你的推荐提供信息，而非对话开场白。

检查：
- **引擎已配置？** 读取 `.claude/docs/technical-preferences.md`。如果 Engine 字段包含 `[待配置]`，则引擎未设置。
- **游戏概念存在？** 检查 `design/gdd/game-concept.md`。
- **源代码存在？** Glob 查找 `src/` 中的源文件（`*.gd`、`*.cs`、`*.cpp`、`*.h`、`*.rs`、`*.py`、`*.js`、`*.ts`）。
- **原型存在？** 检查 `prototypes/` 中的子目录。
- **设计文档存在？** 统计 `design/gdd/` 中的 markdown 文件。
- **制作产物？** 检查 `production/sprints/` 或 `production/milestones/` 中的文件。

内部存储这些发现以验证用户的自我评估并定制推荐。

---

## Phase 2: 询问用户所在位置

这是用户看到的第一件事。使用 `AskUserQuestion` 配合这些精确选项，让用户可以点击而非输入：

- **提示**："Welcome to Claude Code Game Studios! Before I suggest anything, I'd like to understand where you're starting from. Where are you at with your game idea right now?"
- **选项**：
  - `A) No idea yet` — 我完全没有游戏概念。我想探索并弄清楚要做什么。
  - `B) Vague idea` — 我心中有一个粗略的主题、感觉或类型（如"太空相关"或"舒适的农场游戏"），但还没有具体的东西。
  - `C) Clear concept` — 我知道核心想法 — 类型、基本机制，可能还有一句 pitch — 但尚未正式形成文档。
  - `D) Existing work` — 我已经有设计文档、原型、代码或重要的规划工作。我想组织或继续工作。

等待用户选择。在他们回复之前不要继续。

---

## Phase 3: 根据回答路由

#### 如果 A：尚无想法

用户在任何事情之前都需要创意探索。

1. 承认从零开始完全没问题
2. 简要解释 `/brainstorm` 的作用（使用专业框架的引导式构思 — MDA、玩家心理学、动词优先设计）。提及它有两种模式：`/brainstorm open` 用于完全开放的探索，或 `/brainstorm [hint]` 如果他们甚至有模糊的主题（如"太空"、"舒适"、"恐怖"）。
3. 推荐将 `/brainstorm open` 作为下一步运行，但如果想到什么也邀请他们使用提示
4. 展示推荐路径：
   **概念阶段：**
   - `/brainstorm open` — 发现你的游戏概念
   - `/setup-engine` — 配置引擎（brainstorm 会推荐一个）
   - `/prototype` — 一次性概念构建：在设计前验证核心想法是否有趣（1-3 天）
   - `/art-bible` — 定义视觉标识（使用 brainstorm 产生的 Visual Identity Anchor）
   - `/map-systems` — 将概念分解为系统
   - `/design-system` — 为每个 MVP 系统编写 GDD
   - `/review-all-gdds` — 跨系统一致性检查
   - `/gate-check` — 在架构工作前验证准备就绪
   **架构阶段：**
   - `/create-architecture` — 产出主架构蓝图和 Required ADR 列表
   - `/architecture-decision (×N)` — 记录关键技术决策，遵循 Required ADR 列表
   - `/create-control-manifest` — 将决策编译为可操作的规则表
   - `/architecture-review` — 验证架构覆盖率
   **预制作阶段：**
   - `/ux-design` — 为主要屏幕编写 UX 规范（主菜单、HUD、核心交互）
   - `/vertical-slice` — 制作质量的端到端构建，以验证完整游戏循环
   - `/playtest-report (×1+)` — 记录每个垂直切片 playtest 会话
   - `/create-epics` — 将系统映射到 epics
   - `/create-stories` — 将 epics 分解为可实现的 stories
   - `/sprint-plan` — 计划第一个 sprint
   **制作阶段：** → 用 `/dev-story` 接手 stories

#### 如果 B：模糊想法

1. 请他们分享他们模糊的想法 — 甚至几个词就足够了
2. 验证这个想法作为一个起点（不要评判或重定向）
3. 推荐运行 `/brainstorm [他们的提示]` 来发展它
4. 展示推荐路径：
   **概念阶段：**
   - `/brainstorm [hint]` — 将想法发展为完整概念
   - `/setup-engine` — 配置引擎
   - `/prototype` — 一次性概念构建：在设计前验证核心想法是否有趣（1-3 天）
   - `/art-bible` — 定义视觉标识（使用 brainstorm 产生的 Visual Identity Anchor）
   - `/map-systems` — 将概念分解为系统
   - `/design-system` — 为每个 MVP 系统编写 GDD
   - `/review-all-gdds` — 跨系统一致性检查
   - `/gate-check` — 在架构工作前验证准备就绪
   **架构阶段：**
   - `/create-architecture` — 产出主架构蓝图和 Required ADR 列表
   - `/architecture-decision (×N)` — 记录关键技术决策，遵循 Required ADR 列表
   - `/create-control-manifest` — 将决策编译为可操作的规则表
   - `/architecture-review` — 验证架构覆盖率
   **预制作阶段：**
   - `/ux-design` — 为主要屏幕编写 UX 规范（主菜单、HUD、核心交互）
   - `/vertical-slice` — 制作质量的端到端构建，以验证完整游戏循环
   - `/playtest-report (×1+)` — 记录每个垂直切片 playtest 会话
   - `/create-epics` — 将系统映射到 epics
   - `/create-stories` — 将 epics 分解为可实现的 stories
   - `/sprint-plan` — 计划第一个 sprint
   **制作阶段：** → 用 `/dev-story` 接手 stories

#### 如果 C：清晰概念

1. 请他们用一句话描述他们的概念 — 类型和核心机制。使用纯文本，而非 AskUserQuestion（这是开放式回答）。
2. 认可该概念，然后使用 `AskUserQuestion` 提供两条路径：
   - **提示**："How would you like to proceed?"
   - **选项**：
     - `Formalize it first` — 运行 `/brainstorm [concept]` 将其结构化为适当的游戏概念文档
     - `Jump straight in` — 现在就去 `/setup-engine`，之后手动编写 GDD
3. 展示推荐路径：
   **概念阶段：**
   - `/brainstorm` 或 `/setup-engine` —（他们在步骤 2 中的选择）
   - `/prototype` — 一次性概念构建：在设计前验证核心想法是否有趣（1-3 天）
   - `/art-bible` — 定义视觉标识（如果在 brainstorm 之后运行，或在概念文档存在之后）
   - `/design-review` — 验证概念文档
   - `/map-systems` — 将概念分解为单独的系统
   - `/design-system` — 为每个 MVP 系统编写 GDD
   - `/review-all-gdds` — 跨系统一致性检查
   - `/gate-check` — 在架构工作前验证准备就绪
   **架构阶段：**
   - `/create-architecture` — 产出主架构蓝图和 Required ADR 列表
   - `/architecture-decision (×N)` — 记录关键技术决策，遵循 Required ADR 列表
   - `/create-control-manifest` — 将决策编译为可操作的规则表
   - `/architecture-review` — 验证架构覆盖率
   **预制作阶段：**
   - `/ux-design` — 为主要屏幕编写 UX 规范（主菜单、HUD、核心交互）
   - `/vertical-slice` — 制作质量的端到端构建，以验证完整游戏循环
   - `/playtest-report (×1+)` — 记录每个垂直切片 playtest 会话
   - `/create-epics` — 将系统映射到 epics
   - `/create-stories` — 将 epics 分解为可实现的 stories
   - `/sprint-plan` — 计划第一个 sprint
   **制作阶段：** → 用 `/dev-story` 接手 stories

#### 如果 D：已有工作

1. 分享你在 Phase 1 中的发现：
   - "I can see you have [X source files / Y design docs / Z prototypes]..."
   - "Your engine is [configured as X / not yet configured]..."

2. **子情况 D1 — 早期阶段**（引擎未配置或仅存在游戏概念）：
   - 如果引擎未配置，首先推荐 `/setup-engine`
   - 然后 `/project-stage-detect` 进行缺口清单

   **子情况 D2 — GDD、ADR 或 stories 已存在：**
   - 解释："Having files isn't the same as the template's skills being able to use them. GDDs might be missing required sections. `/adopt` checks this specifically."
   - 推荐：
     1. `/project-stage-detect` — 了解当前阶段和完全缺失的内容
     2. `/adopt` — 审计现有产物是否采用正确的内部格式

3. 展示 D2 的推荐路径：
   - `/project-stage-detect` — 阶段检测 + 存在性缺口
   - `/adopt` — 格式合规审计 + 迁移计划
   - `/setup-engine` — 如果引擎未配置
   - `/design-system retrofit [path]` — 填补缺失的 GDD 章节
   - `/architecture-decision retrofit [path]` — 添加缺失的 ADR 章节
   - `/architecture-review` — 引导 TR 需求登记册
   - `/gate-check` — 验证下一阶段的准备就绪

---

## Phase 3c: 写入初始阶段文件

确认起始路径后（以及在询问 review 模式之前），将初始阶段写入 `production/stage.txt`。如果 `production/` 目录不存在则创建。

阶段映射：
- **路径 A、B 或 C（从零开始）**：写入 `Concept`
- **路径 D，现有项目，引擎未配置或仅存在游戏概念**：写入 `Concept`
- **路径 D，现有项目有 GDD 但无架构文档**：写入 `Systems Design`
- **路径 D，现有项目有完整架构（ADRs、架构文档）**：写入 `Technical Setup`

静默执行 — 这个单行文件不需要"可以写入吗？"。

说："I've set `production/stage.txt` to `[stage]` — this anchors your status line and stage detection."

---

## Phase 3b: 设置 Review 模式

检查 `production/review-mode.txt` 是否已存在。

**如果存在**：读取它并显示当前模式 — "Review mode is set to `[current]`." — 然后继续到 Phase 4。不要再问。

**如果不存在**：使用 `AskUserQuestion`：

- **提示**："One setup choice: how much design review would you want as you work through the workflow?"
- **选项**：
  - `Full` — Director 专家在每个关键工作流步骤进行审查。最适合团队、学习工作流，或当你希望对每个决策都有彻底反馈。
  - `Lean (recommended)` — Directors 仅在阶段关卡转换时（`/gate-check`）。跳过每个 skill 的审查。对 solo 开发和小团队的平衡方法。
  - `Solo` — 完全没有 Director 审查。最大速度。最适合 game jam、原型，或如果审查感觉像开销。

用户选择后立即将选择写入 `production/review-mode.txt`
— 不需要单独的"可以写入吗？"，因为写入是选择的直接
结果：
- `Full` → 写入 `full`
- `Lean (recommended)` → 写入 `lean`
- `Solo` → 写入 `solo`

如果 `production/` 目录不存在则创建。

---

## Phase 4: 继续前确认

展示推荐路径后，使用 `AskUserQuestion` 询问用户他们想先采取哪个步骤。绝不自动运行下一个 skill。

- **提示**："Would you like to start with [recommended first step]?"
- **选项**：
  - `Yes, let's start with [recommended first step]`
  - `I'd like to do something else first`

---

## Phase 5: 交接

当用户确认他们的下一步时，用一句简短的话回应："Type `[skill command]` to begin." 仅此而已。不要重新解释该 skill 或添加鼓励。`/start` 的工作已完成。

裁决：**COMPLETE** — 用户已定位并交接给下一步。

---

## 边界情况

- **用户选择 D 但项目为空**：温和地重定向 — "It looks like the project is a fresh template with no artifacts yet. Would Path A or B be a better fit?"
- **用户选择 A 但项目有代码**：提及你发现的 — "I noticed there's already code in `src/`. Did you mean to pick D (existing work)?"
- **用户是返回的（引擎已配置，概念存在）**：完全跳过入门 — "It looks like you're already set up! Your engine is [X] and you have a game concept at `design/gdd/game-concept.md`. Review mode: `[read from production/review-mode.txt, or 'lean (default)' if missing]`. Want to pick up where you left off? Try `/sprint-plan` or just tell me what you'd like to work on."
- **用户不符合任何选项**：让他们用自己的话描述情况并适应。

---

## 协作协议

1. **先询问** — 绝不假设用户的状态或意图
2. **展示选项** — 给出明确的路径，而非命令
3. **用户决定** — 他们选择方向
4. **不自动执行** — 推荐下一个 skill，不要未经询问就运行
5. **适应** — 如果用户的情况不符合模板，倾听并调整
