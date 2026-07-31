---
name: design-system
description: "Guided, section-by-section GDD authoring for a single game system. Gathers context from existing docs, walks through each required section collaboratively, cross-references dependencies, and writes incrementally to file."
argument-hint: "<system-name> [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Task, AskUserQuestion, TodoWrite
model: sonnet
---

当此技能被调用时：

## 1. 解析参数并验证

解析审查模式（一次，存储供本次运行的所有门生成使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

参见 `.claude/docs/director-gates.md` 了解完整的检查模式。

系统名称或 retrofit 路径是**必需的**。如果缺失：

1. 检查 `design/gdd/systems-index.md` 是否存在。
2. 如果存在：读取它，找到状态为 "Not Started" 或等效状态的最高优先级系统，并使用 `AskUserQuestion`：
   - 提示："设计顺序中的下一个系统是 **[system-name]**（[priority] | [layer]）。开始设计它？"
   - 选项：`[A] 是 — 设计 [system-name]` / `[B] 选择不同的系统` / `[C] 停在这里`
   - 如果 [A]：继续该系统名称。如果 [B]：询问要设计哪个系统（纯文本）。如果 [C]：退出。
3. 如果不存在 systems index，失败并显示：
   > "用法：`/design-system <system-name>` — 例如 `/design-system movement`
   > 或填充现有 GDD 中的空白：`/design-system retrofit design/gdd/[system-name].md`
   > 未找到 systems index。先运行 `/map-systems` 映射你的系统并获取设计顺序。"

**检测 retrofit 模式：**
如果参数以 `retrofit` 开头或参数是 `design/gdd/` 中现有 `.md` 文件的文件路径，进入 **retrofit 模式**：

1. 读取现有 GDD 文件。
2. 识别存在 8 个必需部分中的哪些（扫描部分标题）。
   必需部分：Overview、Player Fantasy、Detailed Design/Rules、Formulas、
   Edge Cases、Dependencies、Tuning Knobs、Acceptance Criteria。
3. 识别哪些部分仅包含占位符文本（`[To be designed]` 或等效——空白、单行或明显不完整）。
4. 在做任何事情之前向用户展示：
   ```
   ## Retrofit: [系统名称]
   文件：design/gdd/[filename].md

   已写入的部分（不会触及）：
   ✓ [部分名称]
   ✓ [部分名称]

   缺失或不完整的部分（将被编写）：
   ✗ [部分名称] — 缺失
   ✗ [部分名称] — 仅有占位符
   ```
5. 询问："我可以填充 [N] 个缺失的部分吗？我不会修改任何现有内容。"
6. 如果同意：正常进入**第 2 阶段（收集上下文）**，但在**第 3 阶段**跳过创建骨架（文件已存在），在**第 4 阶段**跳过已完成的部分。仅对缺失/不完整的部分运行部分循环。
7. **永远不要覆盖现有部分内容。** 使用 Edit 工具仅替换 `[To be designed]` 占位符或空的部分主体。

如果不在 retrofit 模式中，将系统名称规范化为 kebab-case 用于文件名（例如，"combat system" 变为 `combat-system`）。

---

## 2. 收集上下文（读取阶段）

在向用户询问**任何事情之前**读取所有相关上下文。这是此技能相对于临时设计的主要优势——它带着信息到达。

### 2a：必需读取

- **游戏概念**：读取 `design/gdd/game-concept.md` — 如果缺失则失败：
  > "未找到游戏概念。先运行 `/brainstorm`。"
- **Systems index**：读取 `design/gdd/systems-index.md` — 如果缺失则失败：
  > "未找到 systems index。先运行 `/map-systems` 映射你的系统。"
- **目标系统**：在索引中查找系统。如果未列出，警告：
  > "[system-name] 不在 systems index 中。你想添加它，还是将其设计为索引外系统？"
- **实体注册表**：如果存在，读取 `design/registry/entities.yaml`。
  提取此系统引用或相关的所有条目（grep
  `referenced_by.*[system-name]` 和 `source.*[system-name]`）。将这些作为**已知事实**保留在上下文中——其他 GDD 已建立的值，此 GDD 不能与之矛盾。
- **Reflexion 日志**：如果存在，读取 `docs/consistency-failures.md`。
  提取 Domain 与此系统类别匹配的条目。这些是反复出现的冲突模式——在第 2d 阶段上下文摘要中将其展示在"过去的失败模式"下，让用户知道此领域以前哪里出过错。

### 2b：依赖关系读取

从 systems index 中，识别：
- **上游依赖**：此系统依赖的系统。如果它们的 GDD 存在，读取它们（这些包含此系统必须尊重的决策）。
- **下游依赖项**：依赖此系统的系统。如果它们的 GDD 存在，读取它们（这些包含此系统必须满足的期望）。

对于每个存在的依赖 GDD，提取并保留在上下文中：
- 关键接口（系统之间流动什么数据）
- 引用此系统输出的公式
- 假设此系统行为的边缘情况
- 输入此系统的调优旋钮

### 2c：可选读取

- **游戏支柱**：如果存在，读取 `design/gdd/game-pillars.md`
- **现有 GDD**：如果存在，读取 `design/gdd/[system-name].md`（恢复，不要从头开始）
- **相关 GDD**：Glob `design/gdd/*.md` 并读取任何主题相关的（例如，如果设计一个在范围上与另一个重叠的系统，即使它不是正式依赖关系，也读取相关的 GDD）

### 2d：展示上下文摘要

开始设计工作之前，向用户展示简要摘要：

> **设计：[系统名称]**
> - 优先级：[来自索引] | 层级：[来自索引]
> - 依赖于：[列表，注明哪些有 GDD 与哪些未设计]
> - 被依赖：[列表，注明哪些有 GDD 与哪些未设计]
> - 要尊重的现有决策：[来自依赖 GDD 的关键约束]
> - 支柱对齐：[此系统主要服务哪个支柱]
> - **已知的跨系统事实（来自注册表）：**
>   - [entity_name]：[attribute]=[value]，[attribute]=[value]（由 [source GDD] 拥有）
>   - [item_name]：[attribute]=[value]，[attribute]=[value]（由 [source GDD] 拥有）
>   - [formula_name]：variables=[list]，output=[min–max]（由 [source GDD] 拥有）
>   - [constant_name]：[value] [unit]（由 [source GDD] 拥有）
>   *（这些值已锁定——如果此 GDD 需要不同的值，在写入之前展示冲突。不要默默使用不同的数字。）*
>
> 如果没有相关的注册表条目：省略"已知的跨系统事实"部分。

如果任何上游依赖未设计，警告：
> "[dependency] 还没有 GDD。我们需要对其接口做出假设。考虑先设计它，或者我们可以定义预期的合同并将其标记为临时。"

### 2e：技术可行性预检

在要求用户开始设计之前，加载引擎上下文并展示将塑造设计的任何约束或知识空白。

**步骤 1 — 确定此系统的引擎领域：**
将系统的类别（来自 systems-index.md）映射到引擎领域：

| 系统类别 | 引擎领域 |
|----------------|--------------|
| 战斗、物理、碰撞 | Physics |
| 渲染、视觉效果、shader | Rendering |
| UI、HUD、菜单 | UI |
| 音频、声音、音乐 | Audio |
| AI、寻路、行为树 | Navigation / Scripting |
| 动画、IK、绑定 | Animation |
| 网络、多人、同步 | Networking |
| 输入、控制、键位绑定 | Input |
| 存档/读档、持久化、数据 | Core |
| 对话、任务、叙事 | Scripting |

**步骤 2 — 读取引擎上下文（如果可用）：**
- 读取 `.claude/docs/technical-preferences.md` 以识别引擎和版本
- 如果引擎已配置，读取 `docs/engine-reference/[engine]/VERSION.md`
- 如果存在，读取 `docs/engine-reference/[engine]/modules/[domain].md`
- 读取 `docs/engine-reference/[engine]/breaking-changes.md` 中与领域相关的条目
- Glob `docs/architecture/adr-*.md` 并读取领域匹配的任何 ADR
  （检查 Engine Compatibility 表中的 "Domain" 字段）

**步骤 3 — 展示可行性简报：**

如果引擎参考文档存在，在设计开始前展示：

```
## 技术可行性简报：[系统名称]
引擎：[name + version]
领域：[domain]

### 已知的引擎能力（针对 [version] 验证）
- [与此系统相关的能力]
- [能力 2]

### 将塑造此设计的引擎约束
- [来自 engine-reference 或现有 ADR 的约束]

### 知识空白（在承诺这些之前验证）
- [此设计可能依赖的 cutoff 后功能——标记 HIGH/MEDIUM 风险]

### 约束此系统的现有 ADR
- ADR-XXXX：[决策摘要] — 意味着 [对此 GDD 的影响]
  （或 "尚无"）
```

如果引擎参考文档不存在（引擎尚未配置），显示简短备注：
> "尚未配置引擎——跳过技术可行性检查。如果还没有，在移动到架构之前运行 `/setup-engine`。"

**步骤 4 — 继续前询问：**

使用 `AskUserQuestion`：
- "在我们开始之前有任何约束要添加，还是按这些记录的继续？"
  - 选项："按这些记录的继续"、"先添加一个约束"、"我需要检查引擎文档——暂停在这里"

---

使用 `AskUserQuestion`：
- "准备好开始设计 [system-name] 了吗？"
  - 选项："是的，开始"、"先展示更多上下文"、"先设计一个依赖"

---

## 3. 创建文件骨架

用户确认后，**立即**创建带有空部分标题的 GDD 文件。这确保增量写入有目标。

使用 `.claude/docs/templates/game-design-document.md` 中的模板结构：

```markdown
# [系统名称]

> **状态**：In Design
> **作者**：[user + agents]
> **最后更新**：[today's date]
> **实现支柱**：[来自上下文]

## Overview

[To be designed]

## Player Fantasy

[To be designed]

## Detailed Design

### Core Rules

[To be designed]

### States and Transitions

[To be designed]

### Interactions with Other Systems

[To be designed]

## Formulas

[To be designed]

## Edge Cases

[To be designed]

## Dependencies

[To be designed]

## Tuning Knobs

[To be designed]

## Visual/Audio Requirements

[To be designed]

## UI Requirements

[To be designed]

## Acceptance Criteria

[To be designed]

## Open Questions

[To be designed]
```

询问："我可以在 `design/gdd/[system-name].md` 创建骨架文件吗？"

如果用户拒绝：停止并显示以下消息：
> "裁决：**BLOCKED** — 骨架创建被拒绝。没有骨架文件设计会话无法进行，因为所有后续阶段都将其作为基础。准备好创建文件时重新运行 `/design-system [system]`。"
不要进入 A 部分。

写入后，更新 `production/session-state/active.md`：
- 使用 Glob 检查文件是否存在。
- 如果**不存在**：使用 **Write** 工具创建它。永远不要尝试在可能不存在的文件上使用 Edit。
- 如果**已存在**：使用 **Edit** 工具更新相关字段。

文件内容：
- 任务：设计 [system-name] GDD
- 当前部分：开始（骨架已创建）
- 文件：design/gdd/[system-name].md

---

## 4. 逐部分设计

按顺序遍历每个部分。对于**每个部分**，遵循此循环：

### 部分循环

```
上下文  ->  问题  ->  选项  ->  决策  ->  草稿  ->  批准  ->  写入
```

1. **上下文**：说明此部分需要包含什么，并展示来自依赖 GDD 的任何相关决策，这些决策约束它。

2. **问题**：询问此部分的特定澄清问题。对受约束的问题使用 `AskUserQuestion`，对开放式探索使用对话文本。

3. **选项**：在部分涉及设计选择（不仅仅是文档）的地方，展示 2-4 种方法及优缺点。在对话文本中解释推理，然后使用 `AskUserQuestion` 捕获决策。

4. **决策**：用户选择方法或提供自定义方向。

5. **草稿**：在对话文本中编写部分内容供审查。标记任何关于未设计依赖的临时假设。

6. **批准**：草稿后立即——在同一响应中——使用 `AskUserQuestion`。**永远不要用纯文本。永远不要跳过此步骤。**
   - 提示："批准 [部分名称] 部分？"
   - 选项：`[A] 批准 — 写入文件` / `[B] 修改 — 描述要修复什么` / `[C] 重新开始`

   **草稿和批准 widget 必须在一个响应中一起出现。
   如果草稿出现时没有 widget，用户将面对空白提示，没有前进路径——这是协议违规。**

7. **写入**：使用 Edit 工具将占位符替换为批准的内容。
   **关键**：始终在 `old_string` 中包含部分标题以确保唯一性——永远不要单独匹配 `[To be designed]`，因为多个部分使用相同的占位符，Edit 工具需要唯一匹配。使用此模式：
   ```
   old_string: "## [部分名称]\n\n[To be designed]"
   new_string: "## [部分名称]\n\n[批准的内容]"
   ```
   确认写入。

8. **注册表冲突检查**（仅 C 和 D 部分——Detailed Design 和 Formulas）：
   写入后，扫描部分内容中出现在注册表中的实体名称、物品名称、公式名称和数字常量。对于每个匹配：
   - 将刚写入的值与注册表条目进行比较。
   - 如果它们不同：**立即展示冲突**，在开始下一个部分之前。不要默默继续。
     > "注册表冲突：[name] 在 [source GDD] 中注册为 [registry_value]。
     > 此部分刚写入 [new_value]。哪个是正确的？"
   - 如果是新的（不在注册表中）：将其标记为注册表注册的候选（将在第 5 阶段处理）。

写入每个部分后，使用已完成的部分名称更新 `production/session-state/active.md`。使用 Glob 检查文件是否存在——如果不存在使用 Write 创建，如果存在使用 Edit 更新。

### 特定部分指南

每个部分都有独特的设计考虑，可能受益于专家代理：

---

### A 部分：Overview

**目标**：陌生人可以阅读并理解的一段话。

**在构建 widget 之前推导推荐选项**：从 systems index（已在第 2 阶段的上下文中）读取系统的类别和层级，然后确定每个标签的推荐选项：
- **Framing 标签**：Foundation/Infrastructure 层级 → 推荐 `[A]`。面向玩家的类别（Combat、UI、Dialogue、Character、Animation、Visual Effects、Audio）→ 推荐 `[C] Both`。
- **ADR ref 标签**：Glob `docs/architecture/adr-*.md` 并 grep 任何 ADR 的 GDD Requirements 部分中的系统名称。如果找到匹配的 ADR → 推荐 `[A] Yes — cite the ADR`。如果未找到 → 推荐 `[B] No`。
- **Fantasy 标签**：Foundation/Infrastructure 层级 → 推荐 `[B] No`。所有其他类别 → 推荐 `[A] Yes`。

在每个标签中的适当选项文本后附加 `(Recommended)`。

**框架问题（起草前询问）**：使用 `AskUserQuestion` 配合多标签 widget：
- 标签 "Framing" — "overview 应如何构建此系统？" 选项：`[A] 作为数据/基础设施层（技术框架）` / `[B] 通过其面向玩家的效果（设计框架）` / `[C] 两者 — 描述数据层及其玩家影响`
- 标签 "ADR ref" — "overview 是否应引用此系统的现有 ADR？" 选项：`[A] 是 — 引用 ADR 了解实现细节` / `[B] 否 — 将 GDD 保持在纯设计级别`
- 标签 "Fantasy" — "此系统是否有值得说明的玩家幻想？" 选项：`[A] 是 — 玩家直接感受到` / `[B] 否 — 纯基础设施，玩家感受它启用的东西`

使用用户的答案来塑造草稿。不要自己回答这些问题并自动起草。

**要问的问题**：
- 这个系统用一句话是什么？
- 玩家如何与它交互？（主动/被动/自动）
- 这个系统为什么存在——没有它会失去什么？

**交叉引用**：检查描述与 systems index 描述它的方式是否一致。标记差异。

**设计与实现边界**：Overview 问题必须保持在行为级别——系统*做什么*，而不是*如何构建*。如果在 Overview 期间出现实现问题（例如，"这应该使用 Autoload 单例还是信号总线？"），将其标记为"→ 变成 ADR"并继续。实现模式属于 `/architecture-decision`，不属于 GDD。GDD 描述行为；ADR 描述用于实现它的技术方法。

---

### B 部分：Player Fantasy

**目标**——情感目标——玩家应该*感受*什么。

**在构建 widget 之前推导推荐选项**：从第 2 阶段上下文读取系统的类别和层级：
- 面向玩家的类别（Combat、UI、Dialogue、Character、Audio、Level/World）→ 推荐 `[A] Direct`
- Foundation/Infrastructure 层级 → 推荐 `[B] Indirect`
- 混合类别（Camera/input、Economy、具有可见玩家效果的 AI）→ 推荐 `[C] Both`

在适当的选项文本后附加 `(Recommended)`。

**框架问题（起草前询问）**：使用 `AskUserQuestion`：
- 提示："这是玩家直接参与的系统，还是他们间接体验的基础设施？"
- 选项：`[A] 直接 — 玩家积极使用或感受此系统` / `[B] 间接 — 玩家体验效果，而不是系统` / `[C] 两者 — 有直接交互层和下面的基础设施`

使用答案适当地构建 Player Fantasy 部分。不要假设答案。

**要问的问题**：
- 这服务于什么情感或力量幻想？
- 哪些参考游戏抓住了这种感觉？具体是什么创造了它？
- 这是一个"你喜欢的参与的系统"还是"你注意不到的基础设施"？

**交叉引用**：必须与游戏支柱一致。如果系统服务于支柱，引用相关的支柱文本。

**审查模式检查**（生成前应用）：
- `solo` → 跳过此代理生成。没有专家起草部分。添加备注："`creative-director` 未咨询 — Solo 模式。在生产前手动审查。"
- `lean` → 跳过，除非这是具有高实现风险的部分（仅 D 和 H 部分）。对于其他部分，没有代理起草。
- `full` → 如下所述生成。

**代理委托（强制性）**：在给出框架答案之后但在起草之前，通过 Task 生成 `creative-director`：
- 提供：系统名称、框架答案（直接/间接/两者）、游戏支柱、用户提到的任何参考游戏、游戏概念摘要
- 询问："塑造此系统的 Player Fantasy。它应该服务于什么情感或力量幻想？我们应该锚定什么玩家时刻？什么 tone 和语言适合游戏已建立的感觉？要具体——给我 2-3 个候选框架。"
- 收集 creative-director 的框架并与草稿一起展示给用户。

**在未先咨询 `creative-director` 之前不要起草 B 部分。** 框架答案告诉我们它*是什么类型*的幻想；creative-director 塑造*如何描述*它——tone、语言、要锚定的特定玩家时刻。

---

### C 部分：Detailed Design（Core Rules、States、Interactions）

**目标**：程序员可以在没有问题的情况下实现的明确规范。

这通常是最大的部分。将其分解为子部分：

1. **Core Rules**：基本机制。对顺序过程使用编号规则，对属性使用项目符号。
2. **States and Transitions**：如果系统有状态，映射每个状态和每个有效转换。使用表。
3. **Interactions with Other Systems**：对于每个依赖（上游和下游），指定什么数据流入、什么流出、谁拥有接口。

**要问的问题**：
- 带我逐步了解此系统的典型使用
- 玩家面临的决策点是什么？
- 玩家*不能*做什么？（约束与能力同样重要）

**审查模式检查**（生成前应用）：
- `solo` → 跳过此代理生成。没有专家起草部分。添加备注："专家代理未咨询 — Solo 模式。在生产前手动审查。"
- `lean` → 跳过，除非这是具有高实现风险的部分（仅 D 和 H 部分）。对于其他部分，没有代理起草。
- `full` → 如下所述生成。

**代理委托（强制性）**：在起草 C 部分之前，通过 Task 并行生成专家代理：
- 在路由表中查找系统类别（此技能的第 6 节）
- 生成该类别列出的 Primary Agent 和 Supporting Agent
- 为每个代理提供：系统名称、游戏概念摘要、pillar 集、依赖 GDD 摘录、正在处理的特定部分
- 在起草之前收集它们的发现
- 通过 `AskUserQuestion` 向用户展示代理之间的任何分歧
- 仅在收到专家输入后起草

**在未先咨询适当的专家之前不要起草 C 部分。** 审查规则和机制的 `systems-designer` 将捕获主会话无法发现的设计空白。

**交叉引用**：对于列出的每个交互，验证它与依赖 GDD 指定的内容是否匹配。如果依赖定义了一个值或公式，而此系统期望不同的东西，标记冲突。

---

### D 部分：Formulas

**目标**：每个数学公式，定义变量，指定范围，注明边缘情况。

**完成引导——始终以此确切结构开始每个公式：**

```
[formula_name] 公式定义为：

`[formula_name] = [expression]`

**变量：**
| 变量 | 符号 | 类型 | 范围 | 描述 |
|----------|--------|------|-------|-------------|
| [name] | [sym] | float/int | [min–max] | [它代表什么] |

**输出范围：** [min] 到 [max] 在正常游戏中；[极端情况下的行为]
**示例：** [使用真实数字的完整示例]
```

不要写 `[Formula TBD]` 或仅用散文描述公式而不使用变量表。没有定义变量的公式无法在没有猜测的情况下实现。

**要问的问题**：
- 此系统执行的核心计算是什么？
- 缩放应该是线性、对数还是阶梯式？
- 早期/中期/后期游戏的输出范围应该是什么？

**审查模式检查**（生成前应用）：
- `solo` → 跳过此代理生成。没有专家起草部分。添加备注："`systems-designer` 未咨询 — Solo 模式。在生产前手动审查。"
- `lean` → 跳过，除非这是具有高实现风险的部分（仅 D 和 H 部分）。对于其他部分，没有代理起草。
- `full` → 如下所述生成。

**代理委托（强制性）**：在提出任何公式或平衡值之前，通过 Task 并行生成专家代理：
- **始终生成 `systems-designer`**：提供 C 部分的 Core Rules、用户的调优目标、来自依赖 GDD 的平衡上下文。要求他们提出含变量表和输出范围的公式。
- **对于经济/成本系统，还要生成 `economy-designer`**：提供放置成本、升级成本意图和进度目标。要求他们验证成本曲线和比率。
- 通过 `AskUserQuestion` 向用户展示专家的提案供审查
- 用户决定；主会话写入文件
- **没有专家输入不要发明公式值或平衡数字。** 没有平衡设计专业知识的用户无法评估原始数字——他们需要专家的推理。

**交叉引用**：如果依赖 GDD 定义了一个其输出输入此系统的公式，明确引用它。不要重新发明——连接。

---

### E 部分：Edge Cases

**目标**：明确处理异常情况，使它们不会变成 bug。

**完成引导——将每个边缘情况格式化为：**
- **如果 [condition]**：[exact outcome]。[如果不明显，说明理由]

示例（调整术语以适应游戏的领域）：
- **如果 [resource] 达到 0 而 [protective condition] 处于活动状态**：保持最小值直到条件结束，然后应用后果。
- **如果两个 [triggers/events] 同时触发**：按 [defined priority order] 解决；平局使用 [defined tiebreak rule]。

不要写诸如"适当处理"之类的模糊条目——每个必须命名确切条件和确切解决方案。没有解决方案的边缘情况是一个开放的设计问题，不是规范。

**要问的问题**：
- 零时会发生什么？最大值时？超出范围值时？
- 当两个规则同时适用时会发生什么？
- 如果玩家发现意外交互会怎样？（识别退化策略）

**审查模式检查**（生成前应用）：
- `solo` → 跳过此代理生成。没有专家起草部分。添加备注："`systems-designer` 未咨询 — Solo 模式。在生产前手动审查。"
- `lean` → 跳过，除非这是具有高实现风险的部分（仅 D 和 H 部分）。对于其他部分，没有代理起草。
- `full` → 如下所述生成。

**代理委托（强制性）**：在最终确定边缘情况之前，通过 Task 生成 `systems-designer`：提供：已完成的 C 和 D 部分，并要求他们识别主会话可能从公式和规则空间中遗漏的边缘情况。对于叙事系统，还要生成 `narrative-director`。展示他们的发现并询问用户要包含哪些。

**交叉引用**：对照依赖 GDD 检查边缘情况。如果依赖定义了此系统可能违反的下限、上限或解决规则，标记它。

---

### F 部分：Dependencies

**目标**：映射每个系统连接及其方向和性质。

此部分从上下文收集阶段部分填充。展示来自 systems index 的已知依赖关系并询问：
- 我是否遗漏了任何依赖关系？
- 对于每个依赖关系，具体的数据接口是什么？
- 哪些依赖关系是硬性的（系统没有它就无法运行）与软性的（由它增强但没有它也能运行）？

**交叉引用**：此部分必须是双向一致的。如果此系统列出"依赖于 Combat"，那么 Combat GDD 应该列出"被 [this system] 依赖"。标记任何单向依赖关系以进行更正。

---

### G 部分：Tuning Knobs

**目标**：每个设计师可调整的值，具有安全范围和极端行为。

**要问的问题**：
- 设计师应该能够在不更改代码的情况下调整哪些值？
- 对于每个旋钮，如果设置太高会破坏什么？太低？
- 哪些旋钮彼此交互？（更改 A 使 B 无关紧要）

**代理委托**：如果公式复杂，委托给 `systems-designer` 从公式变量中推导调优旋钮。

**交叉引用**：如果依赖 GDD 列出了影响此系统的调优旋钮，在此引用它们。不要创建重复的旋钮——指向真相的来源。

---

### H 部分：Acceptance Criteria

**目标**：证明系统按设计工作的可测试条件。

**完成引导——将每个标准格式化为 Given-When-Then：**
- **GIVEN** [初始状态]，**WHEN** [动作或触发]，**THEN** [可测量的结果]

示例（调整术语以适应游戏的领域）：
- **GIVEN** [初始状态]，**WHEN** [玩家动作或系统触发]，**THEN** [具体的可测量的结果]。
- **GIVEN** [约束处于活动状态]，**WHEN** [玩家尝试动作]，**THEN** [显示反馈和动作结果]。

包括至少：C 部分中每个核心规则一个标准，D 部分中每个公式一个标准。不要写"系统按设计工作"——每个标准必须可由 QA tester 独立验证，无需阅读 GDD。

**审查模式检查**（生成前应用）：
- `solo` → 跳过此代理生成。没有专家起草部分。添加备注："`qa-lead` 未咨询 — Solo 模式。在生产前手动审查。"
- `lean` → 跳过，除非这是具有高实现风险的部分（仅 D 和 H 部分）。对于其他部分，没有代理起草。
- `full` → 如下所述生成。

**代理委托（强制性）**：在最终确定验收标准之前，通过 Task 生成 `qa-lead`：提供：已完成的 GDD 部分 C、D、E，并要求他们验证标准是否可独立测试并涵盖所有核心规则和公式。向用户展示任何空白或不可测试的标准。

**要问的问题**：
- 证明这有效的最小测试集是什么？
- 此系统获得什么性能预算？（帧时间、内存）
- QA tester 首先会检查什么？

**交叉引用**：包括验证跨系统交互工作的标准，而不仅仅是此系统单独工作。

---

### 可选部分：Visual/Audio、UI Requirements、Open Questions

这些部分包含在模板中。Visual/Audio 对于视觉系统类别是**必需的**——不是可选的。在询问之前确定要求级别：

**Visual/Audio 对于这些系统类别是必需的（强制性——不要提供跳过）：**
- 战斗、伤害、生命
- UI 系统（HUD、菜单）
- 动画、角色移动
- 视觉效果、粒子、shader
- 角色系统
- 对话、任务、lore
- 关卡/世界系统

对于必需系统：在起草此部分之前，通过 Task 生成 `art-director`。提供：系统名称、游戏概念、游戏支柱、art bible 部分 1-4（如果存在）。要求他们指定：(1) 此系统事件的 VFX 和视觉反馈要求，(2) 任何动画或视觉风格约束，(3) 哪些 art bible 原则最直接适用于此系统。展示他们的输出；不要将此部分留为 `[To be designed]` 对于视觉系统。

对于**所有其他系统类别**（Foundation/Infrastructure、Economy、AI/pathfinding、Camera/input），在必需部分后提供可选部分：

使用 `AskUserQuestion`：
- "8 个必需部分已完成。你还要定义 Visual/Audio 要求、UI 要求，还是捕获未解决问题？"
  - 选项："是的，三个都"、"仅未解决问题"、"跳过——我稍后会添加"

对于 **Visual/Audio**（非必需系统）：如果需要详细信息，与 `art-director` 和 `audio-director` 协调。通常在 GDD 阶段简短备注就足够了。

> **Asset Spec 标志**：在 Visual/Audio 部分写入真实内容后，输出此通知：
> "📌 **Asset Spec** — Visual/Audio 要求已定义。art bible 批准后，运行 `/asset-spec system:[system-name]` 从此部分生成每个资产的视觉描述、尺寸和生成提示。"

对于 **UI Requirements**：对于复杂的 UI 系统与 `ux-designer` 协调。
写入此部分后，检查它是否包含真实内容（不仅仅是 `[To be designed]` 或说明此系统没有 UI 的备注）。如果确实有真实的 UI 要求，立即输出此标志：

> **📌 UX 标志 — [系统名称]**：此系统有 UI 要求。在第 4 阶段
> （Pre-Production），在写 epic 之前，为每个屏幕或 HUD 元素运行 `/ux-design`，
> 此系统贡献的**之前**。引用 UI 的 story 应该引用 `design/ux/[screen].md`，
> 而不是直接引用 GDD。
>
> 如果你更新了它，在 systems index 中为此系统注明。

对于 **Open Questions**：捕获设计期间出现的任何未完全解决的问题。每个问题应该有一个所有者和目标解决日期。

---

## 5. 设计后验证

所有部分写入后：

### 5a：自检

从文件（不是从对话记忆——文件是真相的来源）读回完整的 GDD。验证：
- 所有 8 个必需部分都有真实内容（不是占位符）
- 公式引用定义的变量
- 边缘情况有解决方案
- 依赖关系与接口一起列出
- 验收标准是可测试的

### 5a-bis：Creative Director 支柱审查

**审查模式检查** — 在生成 CD-GDD-ALIGN 之前应用：
- `solo` → 跳过。注意："CD-GDD-ALIGN 已跳过 — Solo 模式。" 进入第 5b 步。
- `lean` → 跳过（不是 PHASE-GATE）。注意："CD-GDD-ALIGN 已跳过 — Lean 模式。" 进入第 5b 步。
- `full` → 正常生成。

在最终确定 GDD 之前，通过 Task 使用门 **CD-GDD-ALIGN**（`.claude/docs/director-gates.md`）生成 `creative-director`。

传递：已完成的 GDD 文件路径、游戏支柱（来自 `design/gdd/game-concept.md` 或 `design/gdd/game-pillars.md`）、MDA 美学目标。

按照 `director-gates.md` 中的标准规则处理裁决。解决后，在 GDD Status 头部记录裁决：
`> **Creative Director Review (CD-GDD-ALIGN)**：APPROVED [date] / CONCERNS (accepted) [date] / REVISED [date]`

---

### 5b：更新实体注册表

扫描已完成的 GDD 中应注册的跨系统事实：
- 命名实体（敌人、NPC、boss）含属性或掉落
- 命名物品含价值、重量或类别
- 命名公式含定义的变量和输出范围
- 命名常量在多个地方按值引用

对于每个候选，检查它是否已存在于 `design/registry/entities.yaml` 中：
```
Grep pattern="  - name: [candidate_name]" path="design/registry/entities.yaml"
```

展示摘要：
```
此 GDD 的注册表候选：
  新的（尚未注册）：
    - [entity_name] [entity]：[attribute]=[value]，[attribute]=[value]
    - [item_name] [item]：[attribute]=[value]，[attribute]=[value]
    - [formula_name] [formula]：variables=[list]，output=[min–max]
  已注册（referenced_by 将更新）：
    - [constant_name] [constant]：value=[N] ← 与注册表匹配 ✅
```

询问："我可以将这些 [N] 个新条目更新到 `design/registry/entities.yaml` 中，并更新现有条目的 `referenced_by` 吗？"

如果同意：追加新条目并更新 `referenced_by` 数组。永远不要修改现有的 `value` / 属性字段，除非首先将其展示为冲突。

### 5c：提供设计审查

展示完成摘要：

> **GDD 完成：[系统名称]**
> - 已写入部分：[列表]
> - 临时假设：[列出任何关于未设计依赖的假设]
> - 发现的跨系统冲突：[列表或 "无"]

> **要验证此 GDD，打开一个新的 Claude Code 会话并运行：**
> `/design-review design/gdd/[system-name].md`
>
> **永远不要在 `/design-system` 的同一会话中运行 `/design-review`。** 审查代理必须独立于编写上下文。在这里运行它会继承完整的设计历史，使独立批评不可能。

**永远不要提供内联运行 `/design-review`。** 始终引导用户到一个新窗口。

### 5d：更新 Systems Index

GDD 完成后（以及可选地审查后）：

- 读取 systems index
- 更新目标系统的行：
  - 如果运行了 design-review 且裁决为 APPROVED：状态 → "Approved"
  - 如果运行了 design-review 且裁决为 NEEDS REVISION：状态 → "In Review"
  - 如果跳过了 design-review：状态 → "Designed"（待审查）
  - 如果用户选择了"我先自己审查"：状态 → "Designed"
  - 设计文档：链接到 `design/gdd/[system-name].md`
- 更新进度跟踪器计数

询问："我可以更新 `design/gdd/systems-index.md` 中的 systems index 吗？"

### 5e：更新会话状态

更新 `production/session-state/active.md`，包含：
- 任务：[system-name] GDD
- 状态：Complete（或如果运行了 design-review 则为 In Review）
- 文件：design/gdd/[system-name].md
- 部分：全部 8 个已写入
- 下一步：[从设计顺序建议下一个系统]

### 5f：建议后续步骤

使用 `AskUserQuestion`：
- "下一步？"
  - 选项：
    - "运行 `/consistency-check` — 验证此 GDD 的值不与现有 GDD 冲突（推荐在下一个系统设计前）"
    - "设计下一个系统（[next-in-order]）"——如果仍有未设计的系统
    - "修复审查发现"——如果 design-review 标记了问题
    - "本次会话停在这里"
    - "运行 `/gate-check`"——如果足够的 MVP 系统已设计

---

## 6. 专家代理路由

此技能委托给专家代理获取领域专业知识。主会话编排整体流程；代理提供专家内容。

| 系统类别 | 主要代理 | 辅助代理 |
|----------------|---------------|---------------------|
| **Foundation/Infrastructure**（事件总线、save/load、场景管理、服务定位器） | `systems-designer` | `gameplay-programmer`（可行性）、`engine-programmer`（引擎集成） |
| 战斗、伤害、生命 | `game-designer` | `systems-designer`（公式）、`ai-programmer`（敌人 AI）、`art-director`（击中反馈视觉方向、VFX 意图） |
| 经济、战利品、制作 | `economy-designer` | `systems-designer`（曲线）、`game-designer`（循环） |
| 进度、XP、技能 | `game-designer` | `systems-designer`（曲线）、`economy-designer`（sink） |
| 对话、任务、lore | `game-designer` | `narrative-director`（故事）、`writer`（内容）、`art-director`（角色视觉档案、电影色调） |
| UI 系统（HUD、菜单） | `game-designer` | `ux-designer`（流程）、`ui-programmer`（可行性）、`art-director`（视觉风格方向）、`technical-artist`（渲染/shader 约束） |
| 音频系统 | `game-designer` | `audio-director`（方向）、`sound-designer`（规范） |
| AI、寻路、行为 | `game-designer` | `ai-programmer`（实现）、`systems-designer`（评分） |
| 关卡/世界系统 | `game-designer` | `level-designer`（空间）、`world-builder`（lore） |
| 相机、输入、控制 | `game-designer` | `ux-designer`（手感）、`gameplay-programmer`（可行性） |
| 动画、角色移动 | `game-designer` | `art-director`（动画风格、姿势语言）、`technical-artist`（绑定/混合约束）、`gameplay-programmer`（手感） |
| 视觉效果、粒子、shader | `game-designer` | `art-director`（VFX 视觉方向）、`technical-artist`（性能预算、shader 复杂度）、`systems-designer`（触发/状态集成） |
| 角色系统（属性、原型） | `game-designer` | `art-director`（角色视觉原型）、`narrative-director`（角色弧对齐）、`systems-designer`（属性公式） |

**通过 Task 工具委托时**：
- 提供：系统名称、游戏概念摘要、依赖 GDD 摘录、正在处理的特定部分以及需要专家输入的问题
- 代理将分析/提案返回给主会话
- 主会话通过 `AskUserQuestion` 向用户展示代理的输出
- 用户决定；主会话写入文件
- 代理不直接写入文件——主会话拥有所有文件写入

---

## 7. 恢复与恢复

如果会话中断（压缩、崩溃、新会话）：

1. 读取 `production/session-state/active.md`——它记录当前系统和哪些部分已完成
2. 读取 `design/gdd/[system-name].md`——有真实内容的部分已完成；有 `[To be designed]` 的部分仍需要工作
3. 从下一个不完整的部分恢复——无需重新讨论已完成的部分

这就是增量写入重要的原因：每个批准的部分在任何中断后都存活。

---

## 协作协议

此技能在每一步都遵循协作设计原则：

1. **问题 -> 选项 -> 决策 -> 草稿 -> 批准** 对于每个部分
2. **AskUserQuestion** 在每个决策点（解释 -> 捕获模式）：
   - 第 2 阶段："准备好开始，还是需要更多上下文？"
   - 第 3 阶段："我可以创建骨架吗？"
   - 第 4 阶段（每个部分）：设计问题、方法选项、草稿批准
   - 第 5 阶段："运行设计审查？更新 systems index？下一步？"
3. **"我可以写入 [filepath] 吗？"** 在骨架之前和每个部分写入之前
4. **增量写入**：每个部分在批准后立即写入文件
5. **会话状态更新**：每次部分写入后
6. **交叉引用**：每个部分检查现有 GDD 的冲突
7. **专家路由**：复杂部分获得专家代理输入，展示给用户决定——永远不要默默写入

**永远不要**自动生成完整的 GDD 并将其作为既成事实展示。
**永远不要**在用户批准之前写入部分。
**永远不要**与现有批准的 GDD 矛盾而不标记冲突。
**始终**展示决策来自哪里（依赖 GDD、支柱、用户选择）。

## 上下文窗口意识

这是一个长时间运行的技能。写入每个部分后，检查状态行是否显示上下文达到或超过 70%。如果是，在响应中附加此通知：

> **上下文接近限制（≥70%）。** 你的进度已保存——所有批准的部分已写入 `design/gdd/[system-name].md`。准备好继续时，打开一个新的 Claude Code 会话并运行 `/design-system [system-name]`——它将检测哪些部分已完成并从下一个恢复。

---

## 推荐的后续步骤

- 在**新会话**中运行 `/design-review design/gdd/[system-name].md` 独立验证已完成的 GDD
- 运行 `/consistency-check` 验证此 GDD 的值不与其他 GDD 冲突
- 运行 `/map-systems next` 移动到下一个最高优先级的未设计系统
- 当所有 MVP GDD 都已编写和审查时运行 `/gate-check pre-production`
