---
name: map-systems
description: "Decompose a game concept into individual systems, map dependencies, prioritize design order, and create the systems index."
argument-hint: "[next | system-name] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, TodoWrite, Task
model: sonnet
---

当此技能被调用时：

## 解析参数

两种模式：

- **无参数**：`/map-systems` — 运行完整的分解工作流（第 1-5 阶段）
  以创建或更新 systems index。
- **`next`**：`/map-systems next` — 从索引中选择最高优先级的未设计系统
  并移交给 `/design-system`（第 6 阶段）。

还解析审查模式（一次，存储供本次运行的所有门生成使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

参见 `.claude/docs/director-gates.md` 了解完整的检查模式。

---

## 第 1 阶段：读取概念（必需上下文）

读取游戏概念和任何现有的设计工作。这为系统分解提供原材料。

**必需：**
- 读取 `design/gdd/game-concept.md` — **如果缺失则以明确消息失败**：
  > "在 `design/gdd/game-concept.md` 未找到游戏概念。先运行 `/brainstorm`
  > 创建一个，然后回来将其分解为系统。"

**可选（如果存在则读取）：**
- 读取 `design/gdd/game-pillars.md` — 支柱约束优先级和范围
- 读取 `design/gdd/systems-index.md` — 如果存在，从它停止的地方**恢复**
  （更新，不要从头重新创建）
- Glob `design/gdd/*.md` — 检查哪些系统 GDD 已存在

**如果 systems index 已存在：**
- 读取它并向用户展示当前状态
- 使用 `AskUserQuestion` 询问：
  "systems index 已存在，包含 [N] 个系统（[M] 个已设计，[K] 个未开始）。
  你想做什么？"
  - 选项："用新系统更新索引"、"设计下一个未设计的系统"、"审查和修订优先级"

---

## 第 2 阶段：系统枚举（协作）

提取并识别游戏需要的所有系统。这是技能的创意核心——它需要人类判断，因为概念文档很少明确枚举每个系统。

### 步骤 2a：提取显式系统

扫描游戏概念中直接提到的系统和机制：
- 核心机制部分（最明确）
- 核心循环部分（暗示每个循环层级由什么系统驱动）
- 技术考虑部分（网络、程序生成等）
- MVP 定义部分（必需功能 = 必需系统）

### 步骤 2b：识别隐式系统

对于每个显式系统，识别它暗示的**隐藏系统**。游戏总是需要比概念文档提到的更多系统。使用此推理模式：

- "背包"暗示：物品数据库、装备槽、重量/容量规则、
  背包 UI、物品序列化用于 save/load
- "战斗"暗示：伤害计算、生命系统、击中检测、状态效果、
  敌人 AI、战斗 UI（血条、伤害数字）、死亡/重生
- "开放世界"暗示：流式/分块、LOD 系统、快速旅行、地图/小地图、
  兴趣点跟踪、世界状态持久化
- "多人"暗示：网络层、大厅/匹配、状态同步、反作弊、
  网络 UI（ping、玩家列表）
- "制作"暗示：配方数据库、材料收集、制作 UI、
  成功/失败机制、配方发现/学习
- "对话"暗示：对话树系统、对话 UI、选择跟踪、NPC
  状态管理、本地化钩子
- "进度"暗示：XP 系统、升级机制、技能树、解锁
  跟踪、进度 UI、进度存档数据

在对话文本中解释为什么需要每个隐式系统（含示例）。

### 步骤 2c：用户审查

按类别组织展示枚举。对于每个系统，展示：
- 名称
- 类别
- 简要描述（1 句话）
- 它是显式的（来自概念）还是隐式的（推断的）

然后使用 `AskUserQuestion` 捕获反馈：
- "此列表中是否缺少系统？"
- "是否应该合并或拆分其中任何一个？"
- "是否有列出的系统这个游戏不需要？"

迭代直到用户批准枚举。

---

## 第 3 阶段：依赖关系映射（协作）

对于每个系统，确定它依赖于什么。如果一个系统在没有另一个系统存在的情况下无法运行，它就"依赖"于那个系统。

### 步骤 3a：映射依赖关系

对于每个系统，列出它的依赖关系。使用这些依赖关系启发式：
- **输入/输出依赖关系**：系统 A 产生系统 B 需要的数据
- **结构依赖关系**：系统 A 提供系统 B 插入的框架
- **UI 依赖关系**：每个游戏系统都有一个对应的 UI 系统
  依赖于它（但 UI 在游戏系统设计之后设计）

### 步骤 3b：按依赖关系顺序排序

将系统排列成层级：
1. **Foundation**：零依赖关系的系统（首先设计和构建）
2. **Core**：仅依赖 Foundation 系统的系统
3. **Feature**：依赖 Core 系统的系统
4. **Presentation**：包装游戏系统的 UI 和反馈系统
5. **Polish**：元系统、教程、分析、无障碍

### 步骤 3c：检测循环依赖

检查依赖图中的循环。如果找到：
- 向用户突出显示它们
- 提出解决方案（接口抽象、同时设计、通过定义两个系统之间的合同打破循环）

### 步骤 3d：向用户展示

将依赖关系映射展示为分层列表。突出显示：
- 任何循环依赖关系
- 任何"瓶颈"系统（许多其他系统依赖它们——这些是高风险的）
- 任何没有依赖项的系统（叶子节点——风险较低，可以晚设计）

使用 `AskUserQuestion` 询问："此依赖关系排序看起来对吗？我是否遗漏了任何依赖关系或应该删除的？"

**审查模式检查** — 在生成 TD-SYSTEM-BOUNDARY 之前应用：
- `solo` → 跳过。注意："TD-SYSTEM-BOUNDARY 已跳过 — Solo 模式。" 进入优先级分配。
- `lean` → 跳过（不是 PHASE-GATE）。注意："TD-SYSTEM-BOUNDARY 已跳过 — Lean 模式。" 进入优先级分配。
- `full` → 正常生成。

**在依赖关系映射获得批准后，在继续到优先级分配之前，通过 Task 使用门 TD-SYSTEM-BOUNDARY（`.claude/docs/director-gates.md`）生成 `technical-director`。**

传递：依赖关系映射摘要、层级分配、瓶颈系统列表、任何循环依赖关系解决方案。

展示评估。如果 REJECT，在移动到优先级分配之前与用户修改系统边界。如果 CONCERNS，在 systems index 中内联注明它们并继续。

---

## 第 4 阶段：优先级分配（协作）

根据系统需要的里程碑将每个系统分配到优先级层级。

### 步骤 4a：基于概念自动分配

使用这些启发式进行初始分配：
- **MVP**：概念中"Required for MVP"部分提到的系统，加上它们的
  Foundation 层依赖关系
- **Vertical Slice**：在一个区域内完成体验所需的系统
- **Alpha**：所有剩余的游戏系统
- **Full Vision**：Polish、元和锦上添花的系统

### 步骤 4b：用户审查

以表格形式展示优先级分配。对于每个层级，解释为什么系统被放在那里。

使用 `AskUserQuestion` 询问："这些优先级分配是否与你的愿景匹配？
哪些系统应该更高或更低优先级？"

在对话中解释推理："我将 [system] 放在 MVP 中，因为核心循环
需要它——没有 [system]，30 秒循环无法运行。"

**"Why" 列指导**：在解释为什么每个系统被放在优先级层级中时，将技术必要性与玩家体验推理混合。不要使用纯粹的技术理由如"战斗需要伤害数学"——在相关的地方连接到玩家体验。好的"Why" 条目示例：
- "核心循环所需——没有它，放置决策没有后果（支柱 2：放置是谜题）"
- "Ballista 的穿刺身份在此确立——这个属性定义是使它感觉与弓箭手不同的原因"
- "所有经济决策的基础——玩家必须理解升级成本才能做出有意义的放置选择"

当系统直接塑造玩家体验时，纯技术必要性（"X 依赖于 Y"）单独是不够的。

**审查模式检查** — 在生成 PR-SCOPE 之前应用：
- `solo` → 跳过。注意："PR-SCOPE 已跳过 — Solo 模式。" 进入写入 systems index。
- `lean` → 跳过（不是 PHASE-GATE）。注意："PR-SCOPE 已跳过 — Lean 模式。" 进入写入 systems index。
- `full` → 正常生成。

**在优先级获得批准后，通过 Task 使用门 PR-SCOPE（`.claude/docs/director-gates.md`）生成 `producer`，然后写入索引。**

传递：每个里程碑层级的系统总数、每层级的估计实现量（系统数量 × 平均复杂度）、团队规模、声明的项目时间线。

展示评估。如果 UNREALISTIC，在写入索引之前提供修改优先级层级分配。如果 CONCERNS，注明它们并继续。

### 步骤 4c：确定设计顺序

组合依赖关系排序 + 优先级层级以产生最终设计顺序：
1. MVP Foundation 系统优先
2. MVP Core 系统其次
3. MVP Feature 系统第三
4. Vertical Slice Foundation/Core 系统
5. ...依此类推

这是团队应该编写 GDD 的顺序。

---

## 第 5 阶段：创建 Systems Index（写入）

### 步骤 5a：起草文档

使用 `.claude/docs/templates/systems-index.md` 中的模板，用第 2-4 阶段的所有数据填充 systems index：
- 填写枚举表
- 填写依赖关系映射
- 填写推荐的设计顺序
- 填写高风险系统
- 填写进度跟踪器（所有系统最初为"未开始"，除非 GDD 已存在）

### 步骤 5b：批准

展示文档摘要：
- 按类别的系统总数
- MVP 系统数量
- 设计顺序中的前 3 个系统
- 任何高风险项目

询问："我可以将 systems index 写入 `design/gdd/systems-index.md` 吗？"

等待批准。仅在"是"后写入文件。

**审查模式检查** — 在生成 CD-SYSTEMS 之前应用：
- `solo` → 跳过。注意："CD-SYSTEMS 已跳过 — Solo 模式。" 进入第 7 阶段后续步骤。
- `lean` → 跳过（不是 PHASE-GATE）。注意："CD-SYSTEMS 已跳过 — Lean 模式。" 进入第 7 阶段后续步骤。
- `full` → 正常生成。

**在 systems index 写入后，通过 Task 使用门 CD-SYSTEMS（`.claude/docs/director-gates.md`）生成 `creative-director`。**

传递：systems index 路径、游戏支柱和核心幻想（来自 `design/gdd/game-concept.md`）、MVP 优先级层级系统列表。

展示评估。如果 REJECT，在 GDD 编写开始之前与用户修改系统集。如果 CONCERNS，将它们记录在 systems index 中相关层级部分顶部的 `> **Creative Director Note**` 中。

### 步骤 5c：更新会话状态

写入后，如果不存在，创建 `production/session-state/active.md`，然后用以下内容更新它：
- 任务：系统分解
- 状态：Systems index 已创建
- 文件：design/gdd/systems-index.md
- 下一步：设计单个系统 GDD

**裁决：COMPLETE** — systems index 已写入 `design/gdd/systems-index.md`。
如果用户拒绝：**裁决：BLOCKED** — 用户未批准写入。

---

## 第 6 阶段：设计单个系统（移交给 /design-system）

当以下情况时进入此阶段：
- 用户说"是的"要在创建索引后设计系统
- 用户调用 `/map-systems [system-name]`
- 用户调用 `/map-systems next`

### 步骤 6a：选择系统

- 如果提供了系统名称，在 systems index 中查找它
- 如果使用了 `next`，选择最高优先级的未设计系统（按设计顺序）
- 如果用户刚刚完成索引，询问：
  "你现在想开始设计单个系统设计吗？设计顺序中的第一个系统是 [name]。还是你想停在这里稍后再回来？"

使用 `AskUserQuestion` 询问："现在开始设计 [system-name]，选择不同的系统，还是停在这里？"

### 步骤 6b：移交给 /design-system

一旦选择了系统，调用 `/design-system [system-name]` 技能。

`/design-system` 技能处理完整的 GDD 编写过程：
- 从游戏概念、systems index 和依赖 GDD 收集上下文
- 立即创建文件骨架
- 一次遍历所有 8 个必需部分（协作、增量）
- 交叉引用现有文档以防止矛盾
- 路由到专家代理获取领域专业知识
- 一旦批准就立即将每个部分写入文件
- 完成时运行 `/design-review`
- 更新 systems index

**不要在此重复 /design-system 工作流。** 此技能拥有 systems
*index*；`/design-system` 拥有单个系统 *GDD*。

### 步骤 6c：循环或停止

`/design-system` 完成后，使用 `AskUserQuestion`：
- "继续下一个系统（[next system name]）？"
- "选择不同的系统？"
- "本次会话停在这里？"

如果继续，返回步骤 6a。

---

## 第 7 阶段：建议后续步骤

创建 systems index 后（或设计了一些系统后），使用 `AskUserQuestion` 展示下一步操作：

- "Systems index 已写入。你想下一步做什么？"
  - [A] 开始设计 GDD — 运行 `/design-system [first-system-in-order]`
  - [B] 运行 `/gate-check systems-design` — 自动触发 CD-SYSTEMS 和 TD-SYSTEM-BOUNDARY 门，对系统集进行正式的主管签字确认
  - [C] 本次会话停在这里

**门检查选项（[B]）值得突出显示**：运行 `/gate-check systems-design` 触发 CD-SYSTEMS 和 TD-SYSTEM-BOUNDARY 门，在它们被锁定在许多文档中之前捕获范围问题、缺失的系统和边界问题。它是可选的但推荐用于新项目。

任何单个 GDD 完成后：
- "在新鲜会话中运行 `/design-review design/gdd/[system].md` 验证质量"
- "当所有 MVP GDD 完成时运行 `/gate-check systems-design`"

---

## 协作协议

此技能在每个阶段都遵循协作设计原则：

1. **问题 -> 选项 -> 决策 -> 草稿 -> 批准** 在每个步骤
2. **AskUserQuestion** 在每个决策点（解释 -> 捕获模式）：
   - 第 2 阶段："缺少系统？合并或拆分？"
   - 第 3 阶段："依赖关系排序正确？"
   - 第 4 阶段："优先级分配是否与你的愿景匹配？"
   - 第 5 阶段："我可以写入 systems index 吗？"
   - 第 6 阶段："开始设计、选择不同的，还是停止？" 然后移交给 `/design-system`
3. **"我可以写入 [filepath] 吗？"** 在每次文件写入之前
4. **增量写入**：在每个系统设计后更新 systems index
5. **移交**：单个 GDD 编写由 `/design-system` 拥有，它处理
   增量部分写入、交叉引用、设计审查和索引更新
6. **会话状态更新**：在每个里程碑（索引创建、系统设计、优先级更改）后
   写入 `production/session-state/active.md`

**永远不要**自动生成完整的 systems 列表并在没有审查的情况下写入。
**永远不要**在用户确认之前开始设计系统。
**始终**展示枚举、依赖关系和优先级供用户验证。

## 上下文窗口意识

如果上下文在任何时候达到或超过 70%，附加此通知：

> **上下文接近限制（≥70%）。** Systems index 已保存到
> `design/gdd/systems-index.md`。打开一个新的 Claude Code 会话继续
> 设计单个 GDD — 运行 `/map-systems next` 从你离开的地方继续。

---

## 推荐的后续步骤

- 运行 `/design-system [first-system-in-order]` 编写第一个 GDD（使用索引中的设计顺序）
- 运行 `/map-systems next` 始终自动选择最高优先级的未设计系统
- 每个 GDD 编写后在新鲜会话中运行 `/design-review design/gdd/[system].md`
- 当所有 MVP GDD 都已编写和审查时运行 `/gate-check pre-production`
