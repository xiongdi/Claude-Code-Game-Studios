---
name: art-bible
description: "Guided, section-by-section Art Bible authoring. Creates the visual identity specification that gates all asset production. Run after /brainstorm is approved and before /map-systems or any GDD authoring begins."
argument-hint: "[--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Task, AskUserQuestion
model: sonnet
---

## 阶段 0：解析参数和上下文检查

解析审查模式（一次，存储供本次运行的所有门生成使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

完整检查模式见 `.claude/docs/director-gates.md`。

读取 `design/gdd/game-concept.md`。如果不存在，失败并显示：
> "未找到游戏概念。先运行 `/brainstorm` —— 艺术圣经在游戏概念获得批准后编写。"

从 game-concept.md 提取：
- 游戏标题（工作标题）
- 核心幻想和电梯演讲
- 游戏支柱（所有）
- **Visual Identity Anchor** 部分（如果存在）（来自 brainstorm 阶段 4 art-director 输出）
- 目标平台（如果已注明）

**Retrofit 模式检测**：Glob `design/art/art-bible.md`。如果文件存在：
- 完整读取它
- 对于 9 个部分中的每个，检查正文是否包含实际内容（多于 `[To be designed]` 占位符或类似内容）与是否为空/占位符
- 构建一个部分状态表：

```
Section | Status
--------|--------
1. Visual Identity Statement | [Complete / Empty / Placeholder]
2. Color Palette | ...
3. Lighting & Atmosphere | ...
4. Character Art Direction | ...
5. Environment & Level Art | ...
6. UI Visual Language | ...
7. VFX & Particle Style | ...
8. Asset Standards | ...
9. Style Prohibitions | ...
```

- 向用户展示此表：
  > "在 `design/art/art-bible.md` 找到现有艺术圣经。[N] 个部分已完成，[M] 个需要内容。我将仅处理不完整的部分 —— 现有内容不会被触及。"
- 仅处理状态为 Empty 或 Placeholder 的部分。不要重新编写已完成的部分。

如果文件不存在，这是一个全新的编写会话 —— 正常进行。

如果存在，读取 `.claude/docs/technical-preferences.md` —— 提取性能预算和引擎以获取资产标准约束。

---

## 阶段 1：框架

展示会话上下文并在编写任何内容之前询问两个问题：

使用带有两个选项卡的 `AskUserQuestion`：
- 选项卡 **"Scope"** —— "今天需要编写哪些部分？"
  选项：`Full bible — all 9 sections` / `Visual identity core (sections 1–4 only)` / `Asset standards only (section 8)` / `Resume — fill in missing sections`
- 选项卡 **"References"** —— "你有定义视觉方向的参考游戏、电影或艺术作品吗？"
  （自由文本 —— 让用户输入具体标题。不要在此预设选项。）

如果 game-concept.md 有 Visual Identity Anchor 部分，注明：
> "从 brainstorm 找到视觉锚点：'[anchor name] — [one-line rule]'。我将以此作为艺术圣经的基础。"

---

## 阶段 2：视觉身份基础（第 1-4 部分）

这四个部分定义核心视觉语言。**所有其他部分都源自它们。** 在移动到下一个之前，将每个部分编写并写入文件。

### 第 1 部分：Visual Identity Statement

**目标**：一条一行视觉规则加上 2-3 个解决视觉模糊性的支持原则。

如果 game-concept.md 中存在视觉锚点：展示它并询问：
- "直接基于此锚点构建？"
- "在扩展之前修改它？"
- "用新选项重新开始？"

**Agent 委托（强制）**：通过 Task 派生 `art-director`：
- 提供：游戏概念（电梯演讲、核心幻想）、完整支柱集、平台目标、阶段 1 框架中的任何参考游戏/艺术、视觉锚点（如果存在）
- 询问："为这个游戏起草 Visual Identity Statement。提供：(1) 一条一行视觉规则，可以解决任何视觉决策模糊性，(2) 2-3 个支持视觉原则，每个都带有一句话设计测试（'当 X 模糊时，此原则说选择 Y'）。将所有原则直接锚定在所述支柱中 —— 每个原则必须服务于特定支柱。"

向用户展示 art-director 的草稿。使用 `AskUserQuestion`：
- 选项：`[A] Lock this in` / `[B] Revise the one-liner` / `[C] Revise a supporting principle` / `[D] Describe my own direction`

立即将批准的部分写入文件。

### 第 2 部分：Mood & Atmosphere

**目标**：按游戏状态的情感目标 —— 足够具体，让灯光艺术家可以据此工作。

对于每个主要游戏状态（例如探索、战斗、胜利、失败、菜单 —— 适应该游戏的状态），定义：
- 主要情感/情绪目标
- 灯光特征（一天中的时间、色温、对比度水平）
- 氛围描述符（3-5 个形容词）
- 能量水平（frenetic / measured / contemplative / 等）

**Agent 委托**：通过 Task 派生 `art-director`，传入 Visual Identity Statement 和支柱集。询问："为这个游戏的每个主要游戏状态定义情绪和氛围目标。要具体 —— 'dark and foreboding' 不够。命名确切的情感目标、灯光特征（暖/冷、高/低对比度、一天中的时间方向）以及至少一个承载情绪的视觉元素。每个游戏状态必须在视觉上与其他状态明显不同。"

立即将批准的部分写入文件。

### 第 3 部分：Shape Language

**目标**：使这个游戏世界视觉上连贯和可区分的几何词汇。

涵盖：
- 角色剪影哲学（缩略图大小时可读性如何？每个原型的区分特征？）
- 环境几何（angular/curved/organic/geometric —— 哪个占主导，为什么？）
- UI 形状语法（UI 是否呼应世界美学，还是它是独特的 HUD 语言？）
- 英雄形状 vs. 支撑形状（什么吸引眼球，什么退后？）

**Agent 委托**：通过 Task 派生 `art-director`，传入 Visual Identity Statement 和情绪目标。询问："为这个游戏定义形状语言。将每个形状原则与视觉身份声明和特定游戏支柱联系起来。解释这些形状选择在情感上向玩家传达了什么。"

立即将批准的部分写入文件。

### 第 4 部分：Color System

**目标**：一个完整的、可生产的调色板系统，服务于美学和沟通需求。

涵盖：
- 主调色板（5-7 种颜色及其角色 —— 不仅仅是十六进制代码，而是每种颜色在这个世界中意味着什么）
- 语义颜色使用（红色传达什么？金色？蓝色？白色？建立颜色词汇）
- 每个生物群系或区域的色温规则（如果游戏有独特的区域）
- UI 调色板（可能与世界调色板不同 —— 明确定义分歧）
- 色盲安全：哪些语义颜色需要形状/图标/声音备份

**Agent 委托**：通过 Task 派生 `art-director`，传入 Visual Identity Statement 和情绪目标。询问："为这个游戏设计颜色系统。必须解释每个语义颜色分配 —— 为什么这种颜色在这个世界中意味着危险/安全/奖励？识别哪些颜色对可能会让色盲玩家失败，并指定需要什么备份提示。"

立即将批准的部分写入文件。

---

## 阶段 3：制作指南（第 5-8 部分）

这些部分将视觉身份转化为具体的制作规则。它们应该足够具体，让外包团队可以在没有额外简报的情况下遵循。

### 第 5 部分：Character Design Direction

**Agent 委托**：通过 Task 派生 `art-director`，传入第 1-4 部分。询问："为这个游戏定义角色设计方向。涵盖：玩家角色的视觉原型（如果有）、每种角色类型的区分特征规则（玩家如何一眼区分敌人/NPC/盟友？）、表情/姿势风格目标（stiff/expressive/realistic/exaggerated）和 LOD 哲学（在游戏相机距离保留多少细节？）。"

将批准的部分写入文件。

### 第 6 部分：Environment Design Language

**Agent 委托**：通过 Task 派生 `art-director`，传入第 1-4 部分。询问："为这个游戏定义环境设计语言。涵盖：建筑风格及其与世界文化/历史的关系、纹理哲学（painted vs. PBR vs. stylized —— 为什么为这个游戏做这个选择？）、道具密度规则（sparse/dense —— 什么驱动每种区域类型的选择？）和环境叙事指南（什么视觉细节应该在没有文字的情况下讲述故事？）。"

将批准的部分写入文件。

### 第 7 部分：UI/HUD Visual Direction

**Agent 委托**：并行派生：
- **`art-director`**：UI 的视觉风格 —— diegetic vs. screen-space HUD、排版方向（字体个性、字重、大小层次）、图标风格（flat/outlined/illustrated/photorealistic）、UI 元素的动画感觉
- **`ux-designer`**：UX 对齐检查 —— 视觉方向是否支持这个游戏所需的交互模式？标记艺术方向与可读性/无障碍需求之间的任何冲突。

收集两者。如果它们冲突（例如，art-director 想要精心制作的 diegetic UI，但 ux-designer 标记它会降低战斗可读性），明确展示冲突及双方立场。不要静默解决 —— 使用 `AskUserQuestion` 让用户决定。

将批准的部分写入文件。

### 第 8 部分：Asset Standards

**Agent 委托**：并行派生：
- **`art-director`**：文件格式偏好、命名约定方向、纹理分辨率层级、LOD 级别期望、导出设置哲学
- **`technical-artist`**：引擎特定的硬约束 —— 每个资产类别的多边形数量预算、纹理内存限制、材质插槽数量、导入器约束、`.claude/docs/technical-preferences.md` 中的性能预算的任何内容

如果任何艺术偏好与技术约束冲突（例如，art-director 想要 4K 纹理，但性能预算要求移动端使用 2K），明确解决冲突 —— 注意理想标准和受限标准，并解释权衡。资产标准中的模糊性是制作成本产生的地方。

将批准的部分写入文件。

---

## 阶段 4：参考方向（第 9 部分）

**目标**：一个精选的参考集，具体说明要从每个来源获取什么以及避免什么。

**Agent 委托**：通过 Task 派生 `art-director`，传入已完成的第 1-8 部分。询问："为这个游戏编译参考方向。提供 3-5 个参考来源（游戏、电影、艺术风格或特定艺术家）。对于每个：命名它，具体说明要从中提取的确切视觉元素（不是'一般美学' —— 特定的技术、颜色选择或构图规则），以及明确要避免或偏离的内容（以防止'试图复制 X'的解读）。参考应该是累加的 —— 没有两个参考应该指向完全相同的方向。"

将批准的部分写入文件。

---

## 阶段 5：Art Director 签字

**审查模式检查** —— 在派生 AD-ART-BIBLE 之前应用：
- `solo` → 跳过。注意："AD-ART-BIBLE skipped —— Solo mode." 继续阶段 6。
- `lean` → 跳过（不是 PHASE-GATE）。注意："AD-ART-BIBLE skipped —— Lean mode." 继续阶段 6。
- `full` → 正常派生。

完成所有部分（或阶段 1 中范围集）后，使用门 **AD-ART-BIBLE** 通过 Task 派生 `creative-director`（`.claude/docs/director-gates.md`）。

传入：艺术圣经文件路径、游戏支柱、视觉身份锚点。

按照 `director-gates.md` 中的标准规则处理裁决。将裁决记录在艺术圣经的状态头部：
`> **Art Director Sign-Off (AD-ART-BIBLE)**: APPROVED [date] / CONCERNS (accepted) [date] / REVISED [date]`

---

## 阶段 6：结束

展示下一步之前，检查项目状态：
- `design/gdd/systems-index.md` 是否存在？→ map-systems 已完成，跳过该选项
- `.claude/docs/technical-preferences.md` 是否包含已配置的引擎（不是 `[TO BE CONFIGURED]`）？→ setup-engine 已完成，跳过该选项
- `design/gdd/` 是否包含任何 `*.md` 文件？→ design-system 已运行，跳过该选项
- `design/gdd/gdd-cross-review-*.md` 是否存在？→ review-all-gdds 已完成
- GDD 是否存在（见上文）？→ 包含 /consistency-check 选项

使用 `AskUserQuestion` 进行下一步。仅包含基于上述状态检查真正下一步的选项：

**选项池 —— 仅包含尚未完成的：**
- `[_] Run /map-systems — decompose the concept into systems before writing GDDs`（如果 systems-index.md 存在则跳过）
- `[_] Run /setup-engine — configure the engine (asset standards may need revisiting after engine is set)`（如果引擎已配置则跳过）
- `[_] Run /design-system — start the first GDD`（如果存在任何 GDD 则跳过）
- `[_] Run /review-all-gdds — cross-GDD consistency check (required before Technical Setup gate)`（如果 gdd-cross-review-*.md 存在则跳过）
- `[_] Run /asset-spec — generate per-asset visual specs and AI generation prompts from approved GDDs`（如果存在 GDD 则包含）
- `[_] Run /consistency-check — scan existing GDDs against the art bible for visual direction conflicts`（如果存在 GDD 则包含）
- `[_] Run /create-architecture — author the master architecture document (next Technical Setup step)`
- `[_] Stop here`

仅为实际包含的选项分配字母 A、B、C…。将最符合逻辑的管道推进选项标记为 `(recommended)`。

> **始终包含** `/create-architecture` 和 Stop here 作为选项 —— 一旦艺术圣经完成，这些始终是有效的下一步。

---

## 协作协议

每个部分遵循：**Question → Options → Decision → Draft (from art-director agent) → Approval → Write to file**

- 永远不要在不首先派生相关 agent 的情况下起草部分
- 每个部分批准后立即写入文件 —— 不要批量处理
- 向用户展示所有 agent 分歧 —— 永远不要静默解决 art-director 和 technical-artist 之间的冲突
- 艺术圣经是一个约束文件：它以视觉连贯性换取对未来决策的限制。每个部分都应该感觉它在有效地缩小解决方案空间。

---

## 推荐的下一步

艺术圣经批准后：
- 运行 `/map-systems` 在编写 GDD 之前将概念分解为游戏系统
- 如果引擎尚未配置，运行 `/setup-engine`（资产标准可能需要在引擎选择后重新审视）
- 运行 `/design-system [first-system]` 开始编写每个系统的 GDD
- 一旦 GDD 存在，运行 `/consistency-check` 根据艺术圣经的视觉规则验证它们
- 运行 `/create-architecture` 生成主架构文档
