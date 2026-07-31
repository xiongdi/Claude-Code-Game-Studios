---
name: design-review
description: "Reviews a game design document for completeness, internal consistency, implementability, and adherence to project design standards. Run this before handing a design document to programmers."
argument-hint: "[path-to-design-doc] [--depth full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Task, AskUserQuestion
model: sonnet
---

## 第 0 阶段：解析参数

如果存在，提取 `--depth [full|lean|solo]`。未给出标志时默认为 `full`。

**注意**：`--depth` 控制此技能的*分析深度*（生成多少专家代理）。它与 `production/review-mode.txt` 中的全局审查模式无关，后者控制主管门生成。这是两个不同的概念——`--depth` 是关于此技能分析文档的彻底程度。

- **`full`**：完整审查——所有阶段 + 专家代理委托（第 3b 阶段）
- **`lean`**：所有阶段，无专家代理——更快的单会话分析
- **`solo`**：仅第 1-4 阶段，无委托，无第 5 阶段后续提示——从另一个技能内调用时使用

---

## 第 1 阶段：加载文档

完整读取目标设计文档。读取 CLAUDE.md 以了解项目上下文和标准。读取目标文档引用或暗示的相关设计文档（检查 `design/gdd/` 中的相关系统）。

**依赖图验证：** 对于 Dependencies 部分中列出的每个系统，使用 Glob 检查其 GDD 文件是否存在于 `design/gdd/` 中。标记任何尚不存在的——这些是下游作者会遇到的中断引用。

**Lore/narrative 对齐：** 如果 `design/gdd/game-concept.md` 或 `design/narrative/` 中的任何文件存在，读取它。注意此 GDD 中与已建立的 world rules、tone 或 design pillars 相矛盾的任何机械选择。在阶段 3b 中将此上下文传递给 `game-designer`。

**先前审查检查：** 检查 `design/gdd/reviews/[doc-name]-review-log.md` 是否存在。如果存在，读取最近的条目——注意给出了什么裁决以及列出了哪些阻塞项目。本次会话是重新审查；跟踪先前项目是否已解决。

---

## 第 2 阶段：完整性检查

对照 Design Document Standard 清单评估：

- [ ] 有 Overview 部分（一段式摘要）
- [ ] 有 Player Fantasy 部分（预期感受）
- [ ] 有 Detailed Rules 部分（明确的机制）
- [ ] 有 Formulas 部分（所有数学定义含变量）
- [ ] 有 Edge Cases 部分（处理的异常情况）
- [ ] 有 Dependencies 部分（列出的其他系统）
- [ ] 有 Tuning Knobs 部分（识别的可配置值）
- [ ] 有 Acceptance Criteria 部分（可测试的成功条件）

---

## 第 3 阶段：一致性和可实现性

**内部一致性：**
- 公式产生的值是否与描述的行为匹配？
- 边缘情况是否与主规则矛盾？
- 依赖关系是否是双向的（另一个系统知道这个系统吗）？

**可实现性：**
- 规则是否足够精确，程序员可以在不猜测的情况下实现？
- 是否有任何"挥手"部分，细节缺失？
- 是否考虑了性能影响？

**跨系统一致性：**
- 这是否与任何现有机制冲突？
- 这是否与其他系统产生意外交互？
- 这是否与游戏已建立的 tone 和 pillars 一致？

---

## 第 3b 阶段：对抗性专家审查（仅 full 模式）

**在 `lean` 或 `solo` 模式下跳过此阶段。**

**此阶段在 full 模式下是强制性的。** 不要跳过它。

**在生成任何代理之前**，打印此通知：
> "完整审查：正在并行生成专家代理。这通常需要 8-15 分钟。使用 `--review lean` 进行更快的单会话分析。"

### 步骤 1 — 识别 GDD 涉及的所有领域

读取 GDD 并识别存在的每个领域。GDD 可以同时涉及多个领域——要彻底。常见信号：

| 如果 GDD 包含... | 生成这些代理 |
|------------------------|-------------------|
| 成本、价格、掉落、奖励、经济 | `economy-designer` |
| 战斗属性、伤害、生命、DPS | `game-designer`、`systems-designer` |
| AI 行为、寻路、瞄准 | `ai-programmer` |
| 关卡布局、生成、波次结构 | `level-designer` |
| 玩家进度、XP、解锁 | `economy-designer`、`game-designer` |
| UI、HUD、菜单、面向玩家的显示 | `ux-designer`、`ui-programmer` |
| 对话、任务、故事、lore | `narrative-director` |
| 动画、手感、时机、juice | `gameplay-programmer` |
| 多人、同步、复制 | `network-programmer` |
| 音频提示、音乐触发 | `audio-director` |
| 性能、绘制调用、内存 | `performance-analyst` |
| 引擎特定模式或 API | 主要引擎专家（来自 `.claude/docs/technical-preferences.md`） |
| 验收标准、测试覆盖 | `qa-lead` |
| 数据模式、资源结构 | `systems-designer` |
| 任何游戏系统 | `game-designer`（始终）

为描述游戏机制或面向玩家规则的所有 GDD 生成 `game-designer`。
为包含公式或系统交互规则的所有 GDD 生成 `systems-designer`。
这些是最常见的基线——但对于纯 UI 规范、音频规范或 lore 文档不是必需的。使用上面的领域表来确定哪些专家真正相关。

### 步骤 2 — 并行生成所有相关专家

**关键：此技能中的 Task 生成一个 SUBAGENT——一个独立的 Claude Code 会话，有自己的上下文窗口。它不是任务跟踪。不要在内部模拟专家视角。不要自己推理领域视图。你必须发出实际的 Task 调用。模拟审查不是专家审查。**

同时发出所有 Task 调用。不要一次生成一个。

**对抗性地提示每个专家：**
> "这是 [system] 的 GDD 和到目前为止主要审查的结构性发现。你的工作不是验证这个设计——你的工作是发现问题。从你的领域专业知识挑战设计选择。什么错了、什么不完整、什么可能引起问题、什么完全缺失？要具体和批判。与主要审查的分歧是受欢迎的。"

**按代理类型的额外指示：**

- **`game-designer`**：将你的审查锚定在此 GDD 的 B 部分所述的 Player Fantasy 上。这个设计是否真正交付了那个幻想？玩家会感受到预期的体验吗？标记任何服务于可实现性但破坏所述感受的规则。

- **`systems-designer`**：对于 GDD 中的每个公式，代入边界值（最小和最大合理输入）。报告是否有任何输出变得退化——负值、除以零、无穷大或在极端情况下无意义的结果。

- **`qa-lead`**：审查每个验收标准。标记任何不可独立测试的——诸如"感觉平衡"、"工作正常"、"表现良好"之类的短语不是 AC。为任何未通过此测试的建议具体重写。

### 步骤 3 — 高级主管审查

所有专家响应后，生成 `creative-director` 作为**高级审查员**：
- 提供：GDD、所有专家发现、它们之间的任何分歧
- 询问："综合这些发现。最重要的问题是什么？你同意专家吗？你对这个设计的总体裁决是什么？"
- creative-director 的综合成为第 4 阶段中的**最终裁决**。

### 步骤 4 — 展示分歧

如果专家彼此不同意或与 creative-director 不同意，不要默默选择一个视图。在第 4 阶段明确展示分歧，以便用户可以裁决。

用其来源标记每个发现：`[game-designer]`、`[economy-designer]`、`[creative-director]` 等。

---

## 第 4 阶段：输出审查

```
## Design Review: [文档标题]
咨询的专家：[列出生成的代理]
重新审查：[是——先前裁决为 X，日期 YYYY-MM-DD / 否——首次审查]

### 完整性：[X/8 部分存在]
[列出缺失的部分]

### 依赖图
[列出每个声明的依赖关系及其 GDD 文件是否存在于磁盘上]
- ✓ enemy-definition-data.md — 存在
- ✗ loot-system.md — 未找到（文件尚不存在）

### 实现前必需
[编号列表——仅阻塞项目。每个项目标记来源代理。]

### 建议修订
[编号列表——重要但不阻塞。标记来源。]

### 专家分歧
[代理彼此不同意或与主要审查不同意的任何情况。
展示双方——不要默默解决。]

### 锦上添花
[小改进，低优先级。]

### 高级裁决 [creative-director]
[Creative director 的综合和总体评估。]

### 范围信号
基于以下估计实现范围：依赖关系数量、公式数量、触及的系统以及是否需要新 ADR。
- **S** — 单一系统、无公式、无新 ADR、<3 个依赖关系
- **M** — 中等复杂度、1-2 个公式、3-6 个依赖关系
- **L** — 多系统集成、3+ 个公式、可能需要新 ADR
- **XL** — 跨领域问题、5+ 个依赖关系、可能需要多个新 ADR
清晰标记："粗略范围信号：M（producer 应在 sprint 计划前验证）"

### 裁决：[APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED]
```

此技能是只读的——第 4 阶段不写入文件。

---

## 第 5 阶段：后续步骤

对所有结束交互使用 `AskUserQuestion`。永远不要用纯文本。

**第一个 widget — 下一步做什么：**

如果 APPROVED（首次通过，无需修订），直接进入 systems-index widget、review-log widget，然后进入最终关闭 widget。不要显示单独的"做什么" widget——最终关闭 widget 涵盖后续步骤。

如果 NEEDS REVISION 或 MAJOR REVISION NEEDED，选项：
- `[A] 现在修订 GDD — 一起解决阻塞项目`
- `[B] 停在这里 — 在单独的会话中修订`
- `[C] 按原样接受并继续（仅当所有项目都是建议性时）`

**如果用户选择 [A] — 现在修订：**

处理所有阻塞项目，仅当你无法仅从 GDD 和现有文档中解决问题时才询问设计决策。在进行任何编辑之前，将所有设计决策问题分组到一个多标签 `AskUserQuestion` 中——不要在修订过程中单独打断每个阻塞项目。

所有修订完成后，展示摘要表（阻塞项目 → 应用的修复）并使用 `AskUserQuestion` 进行**修订后关闭 widget**：

- 提示："修订完成——解决了 [N] 个阻塞项目。下一步？"
- 注意当前上下文使用：如果上下文高于 ~50%，添加："（推荐：重新审查前 /clear — 此会话已使用 X% 上下文。完整重新审查运行 5 个代理，需要干净的上下文。）"
- 选项：
  - `[A] 在新会话中重新审查 — /clear 后运行 /design-review [doc-path]`
  - `[B] 接受修订并标记为 Approved — 更新 systems index，跳过重新审查`
  - `[C] 转到下一个系统 — /design-system [next-system]（设计顺序中的 #N）`
  - `[D] 停在这里`

永远不要用纯文本结束修订流。始终用这个 widget 关闭。

**第二个 widget — 跟踪记录（合并，用于 APPROVED 路径）：**

当裁决为 APPROVED 时，使用单个 `AskUserQuestion` 配合 `multiSelect: true` 批量处理两个跟踪更新：
- 提示："裁决：APPROVED。我现在可以更新跟踪记录。选择你想让我完成的任何一项："
- 选项：
  - `将 systems-index.md 中 [system] 的状态更新为 'Approved'`
  - `将批准条目追加到 design/gdd/reviews/[doc-name]-review-log.md`

如果选择了 review-log 选项，追加与以下相同的格式。在执行两个选定的操作后显示最终关闭 widget。

当裁决为 NEEDS REVISION 或 MAJOR REVISION NEEDED 时，像以前一样使用单独的 widget：

使用第二个 `AskUserQuestion`：
- 提示："我可以更新 `design/gdd/systems-index.md` 将 [system] 标记为 [In Review / Approved] 吗？"
- 选项：`[A] 是 — 更新它` / `[B] 否 — 保持原样`

使用第三个 `AskUserQuestion`：
- 提示："我可以将此审查摘要追加到 `design/gdd/reviews/[doc-name]-review-log.md` 吗？这创建了一个修订历史，以便未来的重新审查可以跟踪更改。"
- 选项：`[A] 是 — 追加到审查日志` / `[B] 否 — 跳过`

如果同意，以此格式追加条目：
```
## 审查 — [YYYY-MM-DD] — 裁决：[APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED]
范围信号：[S/M/L/XL]
专家：[列表]
阻塞项目：[数量] | 建议：[数量]
摘要：[来自 creative-director 裁决的关键发现的 2-3 句摘要]
先前裁决已解决：[是 / 否 / 首次审查]
```

---

**最终关闭 widget — 所有文件写入完成后始终显示：**

回答 systems-index 和 review-log widget 后，检查项目状态并显示一个最终 `AskUserQuestion`：

在构建选项之前，读取：
- `design/gdd/systems-index.md` — 查找任何状态为 In Review 或 NEEDS REVISION 的系统（除了刚刚审查的那个）
- 计算 `design/gdd/` 中的 `.md` 文件（不包括 game-concept.md、systems-index.md）以确定是否值得提供 `/review-all-gdds`（≥2 个 GDD）
- 查找设计顺序中状态为 Not Started 的下一个系统

动态构建选项列表——仅包含真正下一步的选项：
- `[_] 运行 /design-review [other-gdd-path] — [system name] 仍为 [In Review / NEEDS REVISION]`（如果另一个 GDD 需要审查则包含）
- `[_] 运行 /consistency-check — 验证此 GDD 的值不与现有 GDD 冲突`（如果存在 ≥1 个其他 GDD 则始终包含）
- `[_] 运行 /review-all-gdds — 跨所有设计系统的整体设计理论审查`（如果存在 ≥2 个 GDD 则包含）
- `[_] 运行 /design-system [next-system] — 设计顺序中的下一个`（始终包含，命名实际系统）
- `[_] 停在这里`

仅对包含的选项分配字母 A、B、C…。将最推进管道的选项标记为 `(recommended)`。

文件写入后永远不要用纯文本结束技能。始终用这个 widget 关闭。
