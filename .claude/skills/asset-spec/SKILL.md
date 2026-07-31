---
name: asset-spec
description: "Generate per-asset visual specifications and AI generation prompts from GDDs, level docs, or character profiles. Produces structured spec files and updates the master asset manifest. Run after art bible and GDD/level design are approved, before production begins."
argument-hint: "[system:<name> | level:<name> | character:<name>] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Task, AskUserQuestion
model: sonnet
---

如果未提供参数，检查 `design/assets/entity-inventory.md` 是否存在：
- 如果存在：读取它，找到第一个状态为 "Needed" 但没有 spec 文件的实体或屏幕，并使用 `AskUserQuestion`：
  - 提示："下一个未指定规格的项是 **[name]**。为其生成规格？"
  - 选项：`[A] Yes — spec [name]` / `[B] Pick a different item` / `[C] Stop here`
- 如果没有实体清单：检查 `design/assets/asset-manifest.md`。如果清单存在，同上但从清单读取。
- 如果两者都不存在：**启动 Entity & Screen Inventory 流程**（下面的阶段 0b）而不是失败。

---

## 阶段 0b：Entity & Screen Inventory（当无参数且无现有清单时运行）

此流程生成 `design/assets/entity-inventory.md` —— 游戏在视觉上需要的一切的主列表。在资产规格工作开始前运行一次。

### 步骤 1 —— 从文档收集
并行读取所有可用的源材料：
- `design/gdd/systems-index.md` —— 提取列出的每个系统
- `design/gdd/` 中的所有 GDD —— 提取：Visual/Audio Requirements 部分、提到的 UI 元素、VFX 事件、任何命名的实体（角色、敌人、建筑、物品）
- `design/art/art-bible.md` —— 提取：任何命名的视觉类别、资产类型预期
- `design/narrative/` —— 扫描任何角色或世界实体文档（如果存在）（可选 —— 非必需）

### 步骤 2 —— 构建提议的清单
将找到的所有内容组织成类别：

```
Characters / Protagonists
Enemies / Creatures
Buildings / Structures
Environment / Terrain
Items / Props
VFX / Particles
UI Screens（按名称列出每个屏幕）
HUD Elements
Audio（SFX、音乐 —— 仅描述，无生成提示）
Other
```

对于每个项目，注明找到它的源文档。

### 步骤 3 —— 展示并协作
在对话中向用户展示完整的提议清单。然后使用 `AskUserQuestion`：
- 提示："我在你的 GDD 和艺术圣经中找到了 **[N] 个视觉实体和 [N] 个 UI 屏幕**。审查列表 —— 缺少什么，什么不需要？"
- 选项：
  - `[A] 看起来不错 — 保存此清单`
  - `[B] 添加我要描述的项目`
  - `[C] 移除不适用项目`
  - `[D] 同时添加和移除 —— 让我编辑`

如果 [B] 或 [D]：要求用户描述其他项目。接受简短描述（"a medieval keep, used as a level background"）或详细描述 —— 都可以。协作处理直到用户满意。

如果 [C] 或 [D]：询问要删除哪些项目以及原因。从列表中删除它们。

### 步骤 4 —— 写入清单
用户批准后，询问："我可以将实体清单写入 `design/assets/entity-inventory.md` 吗？"

写入文件：

```markdown
# Visual Entity & Screen Inventory

> Generated: [date]
> Sources: [读取的源文档列表]

## Entities

| # | Name | Type | Description | Source | Status |
|---|------|------|-------------|--------|--------|
| 1 | [name] | Character / Enemy / Building / Environment / Item / Other | [简短描述] | [源文档] | Needed |

## UI Screens

| # | Screen Name | Description | Source | Status |
|---|-------------|-------------|--------|--------|
| 1 | Main Menu | [描述] | [源] | Needed |

## HUD Elements

| # | Element | Description | Source | Status |
|---|---------|-------------|--------|--------|

## Audio

| # | Name | Type (SFX / Music / Ambient) | Description | Source | Status |
|---|------|------------------------------|-------------|--------|--------|
```

写入后，告诉用户：
> "实体清单已保存。下一步：
> - 为清单中的每个 UI 屏幕运行 `/ux-design [screen name]`
> - 运行 `/asset-spec entity:[name]` 为每个视觉实体指定规格
> - 或再次运行 `/asset-spec` 逐项处理清单"

---

## 阶段 0：解析参数

提取：
- **目标类型**：`system`、`level` 或 `character`
- **目标名称**：冒号后的名称（规范化为 kebab-case）
- **审查模式**：如果存在 `--review [full|lean|solo]`

**模式行为：**
- `full`（默认）：并行派生 `art-director` 和 `technical-artist`
- `lean`：仅派生 `art-director` —— 更快，跳过技术约束传递
- `solo`：不派生 agent —— 主会话仅从艺术圣经规则编写规格。用于简单资产类别或速度比深度更重要时。

---

## 阶段 1：收集上下文

在询问用户任何内容之前**读取所有源材料**。

### 必需读取：
- **艺术圣经**：读取 `design/art/art-bible.md` —— 如果缺失则失败：
  > "未找到艺术圣经。先运行 `/art-bible` —— 资产规格锚定在艺术圣经的视觉规则和资产标准上。"
  提取：Visual Identity Statement、Color System（语义颜色）、Shape Language、Asset Standards（第 8 部分 —— 尺寸、格式、多边形预算、纹理分辨率层级）。

- **技术偏好**：读取 `.claude/docs/technical-preferences.md` —— 提取性能预算和命名约定。

### 源文档读取（按目标类型）：
- **system**：读取 `design/gdd/[target-name].md`。提取 **Visual/Audio Requirements** 部分。如果不存在或读取 `[To be designed]`：
  > "`design/gdd/[target-name].md` 的 Visual/Audio 部分为空。要么运行 `/design-system [target-name]` 完成 GDD，要么手动描述视觉需求。"
  使用 `AskUserQuestion`：`[A] Describe needs manually` / `[B] Stop — complete the GDD first`
- **level**：读取 `design/levels/[target-name].md`。提取艺术需求、资产列表、VFX 需求和步骤 4 中 art-director 的制作概念规格。
- **character** 或 **entity**：读取 `design/narrative/characters/[target-name].md` 或在 `design/narrative/` 和 `design/assets/entity-inventory.md` 中搜索匹配条目。提取视觉描述、角色和任何指定的区分特征。
  - **如果不存在源文档**：不要失败。而是使用 `AskUserQuestion`：
    - 提示："未找到 **[name]** 的档案。简要描述它 —— 一两句话就够了。"
    - 选项：`[A] Describe it now` / `[B] Skip this entity` / `[C] Stop here`
    - 如果 [A]：用户的描述成为源。简短答案产生简洁规格；详细答案产生详细规格。接受用户提供的任何详细程度并据此工作。

### 可选读取：
- **现有清单**：如果存在，读取 `design/assets/asset-manifest.md` —— 提取已为此目标指定规格的资产以避免重复。
- **相关规格**：Glob `design/assets/specs/*.md` —— 扫描可以共享的资产（例如，为一个系统指定规格的通用 UI 元素可能也适用于此处）。

### 展示上下文摘要：
> **Asset Spec: [Target Type] — [Target Name]**
> - 源文档：[path] —— 识别出 [N] 种资产类型
> - 艺术圣经：找到 —— Asset Standards 在第 8 部分
> - 此目标的现有规格：[N 个已指定规格 / 无]
> - 在其他规格中找到的共享资产：[列表或"无"]

---

## 阶段 2：资产识别

从源文档中提取提到的每种资产类型 —— 显式和隐含的。

**对于系统**：查找 VFX 事件、精灵引用、UI 元素、音频触发器、粒子效果、图标需求和任何"视觉反馈"语言。

**对于关卡**：查找独特的环境道具、大气 VFX、灯光设置、环境音频、天空盒/背景和任何区域特定的材质。

**对于角色**：查找精灵表（idle、walk、attack、death）、肖像/头像、附着在能力上的 VFX、UI 表示（图标、血条皮肤）。

将资产分组为类别：
- **Sprite / 2D Art** —— 角色精灵、UI 图标、tile 表
- **VFX / Particles** —— 打击效果、环境粒子、屏幕效果
- **Environment** —— 道具、tile、背景、天空盒
- **UI** —— HUD 元素、菜单艺术、字体（如果自定义）
- **Audio** —— SFX、音乐曲目、环境循环 *（注意：音频规格仅为描述 —— 无生成提示）*
- **3D Assets** —— 网格、材质（如果适用于引擎）

向用户展示完整的识别列表。使用 `AskUserQuestion`：
- 提示："我为 **[target]** 在 [N] 个类别中识别出 [N] 个资产。在指定规格前审查："
- 首先在对话文本中显示分组列表
- 选项：`[A] 继续 —— 为所有这些指定规格` / `[B] 移除一些资产` / `[C] 添加我遗漏的资产` / `[D] 调整类别`

未经用户确认资产列表，不要继续到阶段 3。

---

## 阶段 3：规格生成

基于审查模式派生专家 agent。**同时发出所有 Task 调用 —— 不要等待一个完成再开始下一个。**

### 完整模式 —— 并行派生：

**`art-director`** 通过 Task：
- 提供：阶段 2 的完整资产列表、艺术圣经 Visual Identity Statement、Color System、Shape Language、源文档的视觉需求以及艺术圣经第 9 部分中提到的任何参考游戏/艺术
- 询问："对于此列表中的每个资产，生成：(1) 2-3 句视觉描述，锚定在艺术圣经的形状语言和颜色系统上 —— 足够具体，让两个不同的艺术家能产生一致的结果；(2) 准备用于 AI 图像工具的生成提示（Midjourney/Stable Diffusion 风格 —— 包括风格关键词、构图、调色板锚点、负面提示）；(3) 哪些艺术圣经规则直接管辖此资产（按部分引用）。对于音频资产，描述声音特征而不是生成提示。"

**`technical-artist`** 通过 Task：
- 提供：完整资产列表、艺术圣经 Asset Standards（第 8 部分）、technical-preferences.md 性能预算、引擎名称和版本
- 询问："对于此列表中的每个资产，指定：(1) 精确尺寸或多边形数量（匹配艺术圣经 Asset Standards 层级 —— 不要发明新尺寸）；(2) 文件格式和导出设置；(3) 命名约定（来自 technical-preferences.md）；(4) 此资产类型必须尊重的任何引擎特定约束；(5) 如果适用，LOD 需求。标记艺术圣经的首选标准与引擎约束冲突的任何资产类型。"

### 精简模式 —— 仅派生 art-director（跳过 technical-artist）。

### 独奏模式 —— 两者都跳过。仅从艺术圣经规则推导规格，注意技术约束未经验证。

**在阶段 4 之前收集两个响应。** 如果 art-director 和 technical-artist 之间存在任何冲突（例如，art-director 指定 4K 纹理，但 technical-artist 标记引擎预算需要 512px），明确展示它 —— 不要静默解决。

---

## 阶段 4：编译和审查

将 agent 输出组合成每个资产的草稿规格。使用此格式在对话文本中展示所有规格：

```
## ASSET-[NNN] — [Asset Name]

| Field | Value |
|-------|-------|
| Category | [Sprite / VFX / Environment / UI / Audio / 3D] |
| Dimensions | [例如 256×256px, 4-frame sprite sheet] |
| Format | [PNG / SVG / WAV / 等] |
| Naming | [例如 vfx_frost_hit_01.png] |
| Polycount | [如果 3D —— 例如 <800 tris] |
| Texture Res | [例如 512px — matches Art Bible §8 Tier 2] |

**Visual Description:**
[2-3 句话。足够具体，让两个艺术家能产生一致的结果。]

**Art Bible Anchors:**
- §3 Shape Language: [应用的相关规则]
- §4 Color System: [颜色角色 —— 例如 "uses Threat Blue per semantic color rules"]

**Generation Prompt:**
[准备使用的提示。包括：风格关键词、构图笔记、调色板锚点、灯光方向、负面提示。]

**Status:** Needed
```

展示所有规格后，使用 `AskUserQuestion`：
- 提示："**[target]** 的资产规格 —— [N] 个资产。审查完成？"
- 选项：`[A] 全部批准 —— 写入文件` / `[B] 修改特定资产` / `[C] 用不同方向重新生成`

如果 [B]：询问哪个资产以及更改什么。在线内联修改并重新呈现。不要为小的文本修改重新派生 agent —— 仅当视觉方向本身需要改变时才重新派生。

如果 [C]：询问要改变什么方向。用更新的简报重新派生相关 agent。

---

## 阶段 5：写入规格文件

批准后，询问："我可以将规格写入 `design/assets/specs/[target-name]-assets.md` 吗？"

写入文件：

```markdown
# Asset Specs — [Target Type]: [Target Name]

> **Source**: [源 GDD/关卡/角色文档的路径]
> **Art Bible**: design/art/art-bible.md
> **Generated**: [date]
> **Status**: [N] assets specced / [N] approved / [N] in production / [N] done

[ASSET-NNN 格式的所有资产规格]
```

然后更新 `design/assets/asset-manifest.md`。如果不存在，创建它：

```markdown
# Asset Manifest

> Last updated: [date]

## Progress Summary

| Total | Needed | In Progress | Done | Approved |
|-------|--------|-------------|------|----------|
| [N] | [N] | [N] | [N] | [N] |

## Assets by Context

### [Target Type]: [Target Name]
| Asset ID | Name | Category | Status | Spec File |
|----------|------|----------|--------|-----------|
| ASSET-001 | [name] | [category] | Needed | design/assets/specs/[target]-assets.md |
```

如果清单已存在，追加新上下文块并更新 Progress Summary 计数。

询问："我可以更新 `design/assets/asset-manifest.md` 吗？"

---

## 阶段 6：结束

使用 `AskUserQuestion`：
- 提示："**[target]** 的资产规格完成。下一步？"
- 选项：
  - `[A] Spec another system — /asset-spec system:[next-system]`
  - `[B] Spec a level — /asset-spec level:[level-name]`
  - `[C] Spec a character — /asset-spec character:[character-name]`
  - `[D] Run /asset-audit —— 根据规格验证交付的资产`
  - `[E] Stop here`

---

## 资产 ID 分配

资产 ID 在整个项目中按顺序分配 —— 不是按上下文。在分配 ID 之前读取清单以找到当前最高数字：

```
Grep pattern="ASSET-" path="design/assets/asset-manifest.md"
```

从 `ASSET-[highest + 1]` 开始新资产。这确保 ID 在整个项目中稳定且唯一。

如果尚不存在清单，从 `ASSET-001` 开始。

---

## 共享资产协议

在指定资产规格之前，检查等效资产是否已存在于另一个上下文的规格中：

- 通用 UI 元素（血条、分数显示）通常跨系统共享
- 通用环境道具可能出现在多个关卡中
- 角色 VFX（打击火花、死亡效果）可能重用带有颜色变体的基础规格

如果找到匹配项：引用现有 ASSET-ID 而不是创建重复项。在清单的 referenced-by 列中注明共享用法。

> "ASSET-012（Generic Hit Spark）已为 Combat 系统指定规格。为 Tower Defense 重用 —— 将 tower-defense 添加到 referenced-by。"

---

## 错误恢复协议

如果任何派生的 agent 返回 BLOCKED 或无法完成：

1. 立即展示："[AgentName]: BLOCKED —— [reason]"
2. 在 `lean` 模式或如果 `technical-artist` 阻塞：仅使用 art-director 输出继续 —— 注意技术约束未经验证
3. 在 `solo` 模式或如果 `art-director` 阻塞：从艺术圣经规则推导描述 —— 标记为 "Art director not consulted —— verify against art bible before production"
4. 始终生成部分规格 —— 永远不要因为一个 agent 阻塞而丢弃工作

---

## 协作协议

每个阶段遵循：**Identify → Confirm → Generate → Review → Approve → Write**

- 永远不要在没有首先与用户确认资产列表的情况下指定资产规格
- 始终将规格锚定在艺术圣经上 —— 与艺术圣经矛盾的规格是错误的
- 展示所有 agent 分歧 —— 不要静默选择一方
- 仅在明确批准后写入规格文件
- 写入规格后立即更新清单

---

## 推荐的下一步

- 运行 `/asset-spec [next-context]` 继续为剩余系统、关卡或角色指定规格
- 运行 `/asset-audit` 根据编写的规格验证交付的资产并识别缺口或不匹配
