# 示例会话：/adopt — Brownfield 项目入门

**背景：** 开发者在发现此 Agent 系统之前已经开发游戏3个月。他们有代码、一些设计文档和粗略笔记——但没有任何适合技能使用的正确格式。

**起始状态：**
- `src/gameplay/` — 约4000行 Godot 4.6 GDScript
- `design/` — 3个带有粗略设计笔记的 markdown 文件（不是 GDD 模板格式）
- 无 `design/gdd/` 目录
- 无 `docs/architecture/` 目录
- 无 `production/` 目录
- 无 `design/gdd/systems-index.md`
- CLAUDE.md 存在（引擎已配置：Godot 4.6）

**会话时长：** 约30分钟
**轮次：** 8
**结果：** 采用计划已写，最紧急空白（缺失系统索引）立即修复

---

## 第1轮：用户调用 /adopt

**用户：**
> /adopt

## 第2轮：Agent 检测项目状态

**Agent（technical-director，通过 /adopt，上下文：fork）：**
> 运行 brownfield 审计...
>
> **阶段1：项目状态检测**
>
> | 工件类别 | 发现 | 备注 |
> |------------------|-------|-------|
> | 引擎配置 | ✅ | CLAUDE.md 中的 Godot 4.6 |
> | 源代码 | ✅ | `src/gameplay/` 中约4000行 |
> | 设计文档 | ⚠️ | `design/` 中有3个文件，不是 GDD 格式 |
> | 架构文档 | ❌ | 缺少 `docs/architecture/` |
> | 系统索引 | ❌ | 缺少 `design/gdd/systems-index.md` |
> | 生产追踪 | ❌ | 缺少 `production/` 目录 |
> | Sprint/Story 文件 | ❌ | 无 |
>
> **阶段2：格式审计（现有文件）**
>
> 检查3个设计文件的内部结构...
>
> *`design/combat-notes.md`：*
> - 有非正式笔记，无 GDD 模板章节
> - 包含：一些规则描述、伤害公式、粗略的平衡数字
> - GDD 模板缺失：概述、玩家幻想、边缘情况、依赖、调优旋钮、验收标准（8章中缺6章）
>
> *`design/crafting-ideas.md`：*
> - 头脑风暴式笔记，无结构
> - 无公式，无验收标准
> - 代表早期概念思考——可作为 GDD 输入有用，尚未成为 GDD
>
> *`design/inventory.md`：*
> - 最接近 GDD——有概述和规则部分
> - 缺失：玩家幻想、公式、边缘情况、调优旋钮、验收标准（8章中缺5章）
>
> **阶段3：空白分类**
>
> | 空白 | 严重性 | 原因 |
> |-----|----------|--------|
> | 无系统索引 | BLOCKING | `/design-system`、`/create-stories`、`/gate-check` 都需要它 |
> | GDD 文件不是模板格式 | HIGH | 技能无法从格式错误的 GDD 引用 TR-ID |
> | 无架构文档 | HIGH | 无法运行 `/architecture-review` 或 `/create-control-manifest` |
> | 无生产追踪 | MEDIUM | Sprint 和 story 技能不工作，但生产阶段前不需要 |
> | `design/crafting-ideas.md` — 预 GDD 内容 | LOW | 作为输入有用；在阻塞任何内容前需要完整 GDD 创作 |

## 第3轮：Agent 呈现迁移计划

**Agent：**
> **阶段4：迁移计划**
>
> 7个有序步骤将你的项目完全对齐。永远不覆盖你现有内容——每步仅填补空白。
>
> | # | 步骤 | 技能 | 优先级 | 备注 |
> |---|------|-------|----------|-------|
> | 1 | 创建 `design/gdd/systems-index.md` | `/map-systems` 或 `/adopt` 内联 | BLOCKING | 解锁所有下游技能 |
> | 2 | 改造 `design/inventory.md` → GDD 格式 | `/design-system retrofit design/inventory.md` | HIGH | 添加5个缺失章节，保留你现有内容 |
> | 3 | 改造 `design/combat-notes.md` → GDD 格式 | `/design-system retrofit design/combat-notes.md` | HIGH | 添加6个缺失章节 |
> | 4 | 转换 `design/crafting-ideas.md` → 完整 GDD | `/design-system crafting`（新创作，使用 ideas 作为输入） | MEDIUM | 不是改造——太不正式无法改造 |
> | 5 | 从现有代码创建架构 ADR | `/reverse-document` + `/architecture-decision` | HIGH | 捕获代码中已做的决策 |
> | 6 | 创建主架构文档 | `/create-architecture` | HIGH | 在 ADR 之后 |
> | 7 | 设置生产追踪 | `/sprint-plan new` | MEDIUM | 在架构阶段之后 |
>
> **重要：** 第1步必须在任何其他步骤之前发生。其他可以并行或根据你的优先级重新排序。
>
> 可以将此计划写入 `docs/adoption-plan-2026-03-12.md` 吗？

**用户：**
> 是。写吧。

**Agent 写入 adoption plan。**

## 第4轮：Agent 提供修复最紧急空白的选项

**Agent：**
> BLOCKING 空白是缺失的 `design/gdd/systems-index.md`。
>
> 我现在可以通过读取你现有代码和设计文档来创建一个，推断你构建了哪些系统。这比从头运行 `/map-systems` 更快——我将基于已存在的内容填充它，然后你确认或调整。
>
> 你想现在这样做吗？

**用户：**
> 是，做吧。

## 第5轮：Agent 读取代码库并起草系统索引

**Agent：**
> 读取 `src/gameplay/` 以推断系统...
>
> 在代码中找到这些系统边界：
>
> - `src/gameplay/combat/` — 伤害计算、命中检测、状态效果
> - `src/gameplay/inventory/` — 物品管理、重量、堆叠
> - `src/gameplay/crafting/` — 配方查询、结果生成（引用 inventory）
> - `src/gameplay/movement/` — CharacterBody2D、地形修饰符（被 combat 引用用于定位）
> - `src/gameplay/ui/` — HUD、物品栏屏幕、对话框
> - `src/gameplay/world/` — 瓦片地图、地形类型、块加载
>
> **起草系统索引：**
>
> | 系统 | 类别 | 优先级 | 依赖 | GDD 状态 |
> |--------|----------|----------|--------------|------------|
> | Movement | Foundation | MVP | None | ❌ 缺失 |
> | World/Terrain | Foundation | MVP | Movement | ❌ 缺失 |
> | Combat | Core | MVP | Movement | ⚠️ 需要改造 |
> | Inventory | Core | MVP | None | ⚠️ 需要改造 |
> | Crafting | Feature | MVP | Inventory | ❌ 仅预 GDD |
> | UI / HUD | Presentation | MVP | Combat, Inventory, Movement | ❌ 缺失 |
>
> 看起来对吗？我遗漏或错误分类了什么？

**用户：**
> 对。添加一个 Stamina 系统——它在 combat/ 中但应该是自己的系统。

**Agent：**
> 添加 Stamina 作为 Core / MVP 系统：
>
> | Stamina | Core | MVP | Movement, Combat | ❌ 缺失 |
>
> 更新依赖：Movement 依赖 Stamina（移动消耗它）。
>
> 可以将此写入 `design/gdd/systems-index.md` 吗？

## 第6轮：用户批准

**用户：**
> 是

**Agent 写入 `design/gdd/systems-index.md`。**

## 第7轮：Agent 显示现在解锁了什么

**Agent：**
> `design/gdd/systems-index.md` 已创建。
>
> **BLOCKING 空白已解决。** 以下技能现在可以运行：
>
> - ✅ `/design-system [name]` — 任何系统的引导式 GDD 创作
> - ✅ `/design-system retrofit [path]` — 在现有文档中填补缺失章节
> - ✅ `/gate-check` — 现在有系统索引可验证
> - ✅ `/help` — 现在可以显示正确的阶段工作流程
>
> **推荐的下一步（来自采用计划）：**
>
> 1. 运行 `/design-system retrofit design/inventory.md` — 你最完整的现有文档，最容易的胜利
> 2. 运行 `/design-system retrofit design/combat-notes.md`
> 3. 运行 `/design-system crafting` — 使用你的 ideas 文件作为输入创作 crafting GDD
>
> 完成这3个后，你将有7个系统中5个的完整 GDD。然后运行 `/review-all-gdds` 以在架构工作之前捕获跨系统一致性问题。

## 第8轮：用户确认计划

**用户：**
> 完美。我从 inventory 改造开始。

**Agent：**
> 运行：
> ```
> /design-system retrofit design/inventory.md
> ```
>
> 技能将检测你现有的概述和规则章节，显示缺失的8章中的哪些，并在间隙上仅运行章节循环。你的现有内容不会被触及。

---

## 此示例演示什么

- **格式审计，而非存在审计**：`/adopt` 不仅检查文件是否存在——还检查其内部结构是否匹配技能期望。名为 `gdd.md` 但无模板章节的文件被标记为空白。
- **迁移，而非替换**：现有内容永远不会被覆盖。计划仅填补空白。
- **BLOCKING 空白首先浮现**：缺失的系统索引被识别为 BLOCKING，因为它阻止最多技能运行。
- **紧急空白的内联创建**：Agent 不仅报告空白，还提供在同一会话中立即修复的选项。
- **从代码推断**：系统索引从代码库结构引导，而非从头编写——brownfield 代码已包含答案。
- **改造 vs. 新创作**：`/design-system retrofit` 处理现有但不完整的文档；完整的 `/design-system` 处理全新文档。采用计划区分每个文件适用哪个。
- **上下文：fork**：`/adopt` 在 forked 上下文中运行，以避免大型文件读取污染主会话。