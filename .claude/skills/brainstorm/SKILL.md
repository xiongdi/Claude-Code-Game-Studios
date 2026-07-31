---
name: brainstorm
description: "Guided game concept ideation — from zero idea to a structured game concept document. Uses professional studio ideation techniques, player psychology frameworks, and structured creative exploration."
argument-hint: "[genre or theme hint, or 'open'] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, WebSearch, Task, AskUserQuestion
model: sonnet
---

当此 skill 被调用时：

1. **解析参数**以获取可选的类型/主题提示（例如 `roguelike`、
   `space survival`、`cozy farming`）。如果 `open` 或无参数，从头开始。
   还要解析审查模式（一次，存储供本次运行的所有门生成使用）：
   1. 如果传入了 `--review [full|lean|solo]` → 使用该值
   2. 否则读取 `production/review-mode.txt` → 使用该值
   3. 否则 → 默认为 `lean`

   完整检查模式见 `.claude/docs/director-gates.md`。

2. **检查现有概念工作**：
   - 如果存在，读取 `design/gdd/game-concept.md`（恢复，不要重新开始）
   - 如果存在，读取 `design/gdd/game-pillars.md`（在已建立的支柱上构建）

3. **交互式地经历构思阶段**，在每个阶段向用户提问。
   不要静默生成所有内容 —— 目标是**协作探索**，其中 AI 充当创意推动者，而不是人类愿景的替代品。

   **在头脑风暴的关键决策点使用 `AskUserQuestion`**：
   - 受限的品味问题（类型偏好、范围、团队规模）
   - 展示选项后的概念选择（"哪 2-3 个概念引起共鸣？"）
   - 方向选择（"进一步开发、探索更多，还是原型？"）
   - 概念完善后的支柱排名
   首先在对话文本中写出完整的创意分析，然后使用
   `AskUserQuestion` 以简洁的标签捕获决策。

   要遵循的专业工作室头脑风暴原则：
   - 保留判断 —— 探索期间没有坏主意
   - 鼓励不寻常的想法 —— 开箱即用的思维激发更好的概念
   - 相互构建 —— "是的，而且..."回应，而不是"但是..."
   - 将约束作为创意燃料 —— 限制通常产生最好的想法
   - 为每个阶段设定时间限制 —— 保持势头，不要过早过度思考

---

### 阶段 1：创意发现

从了解这个人开始，而不是游戏。以对话方式询问这些问题
（不是作为清单）：

**情感锚点**：
- 游戏中有没有一个时刻真正打动你、让你兴奋，或者让你
  忘记了时间？是什么具体创造了那种感觉？
- 是否有一个幻想或权力之旅你一直想在游戏中体验但从未
  完全找到？

**品味档案**：
- 你花时间最多的 3 个游戏是什么？什么让你不断回来？
  *（将此作为纯文本询问 —— 用户必须能够自由输入具体游戏名称。
  不要将其放在带有预设选项的 AskUserQuestion 中。）*
- 有你喜欢的类型吗？有你避免的类型吗？为什么？
- 你更喜欢挑战你的游戏、放松你的游戏、讲述故事的游戏，
  还是让你表达自己的游戏？*（对此使用 `AskUserQuestion` —— 受限选择。）*

**实际约束**（在头脑风暴之前塑造沙盒）。
将这些捆绑成一个带有这些确切选项卡标签的多选项卡 `AskUserQuestion`：
- 选项卡 "Experience" —— "你最希望玩家拥有什么样的体验？"（Challenge & Mastery / Story & Discovery / Expression & Creativity / Relaxation & Flow）
- 选项卡 "Timeline" —— "你实际的开发时间线是什么？"（Weeks / Months / 1-2 years / Multi-year）
- 选项卡 "Dev level" —— "你在开发旅程中的哪个位置？"（First game / Shipped before / Professional background）

使用这些确切的选项卡名称 —— 不要重命名或复制它们。

**综合**答案成一个 **Creative Brief** —— 一个 3-5 句的
人的情感目标、品味档案和约束的摘要。
大声读回简报并确认它捕捉了他们的意图。

---

### 阶段 2：概念生成

以创意简报为基础，生成 **3 个不同的概念**，
每个都采用不同的创意方向。使用这些构思技巧：

**技巧 1：动词优先设计**
从核心玩家动词（build、fight、explore、solve、survive、
create、manage、discover）开始，并从那里向外扩展。动词就是游戏。

**技巧 2：混搭方法**
组合两个意想不到的元素：[类型 A] + [主题 B]。两者之间的
张力创造了独特的钩子。（例如，"farming sim + cosmic horror"、
"roguelike + dating sim"、"city builder + real-time combat"）

**技巧 3：体验优先设计（MDA 逆向）**
从期望的玩家情感开始（MDA 框架中的美学目标：
sensation、fantasy、narrative、challenge、fellowship、discovery、expression、
submission）并逆向推导出产生它的动态和机制。

对于每个概念，展示：
- **Working Title**
- **Elevator Pitch**（1-2 句话 —— 必须通过"10 秒测试"）
- **Core Verb**（最常见的玩家动作）
- **Core Fantasy**（情感承诺）
- **Unique Hook**（通过"and also"测试："Like X, AND ALSO Y"）
- **主要 MDA 美学**（哪种情感占主导？）
- **Estimated Scope**（small / medium / large）
- **Why It Could Work**（关于市场/受众契合度的 1 句话）
- **Biggest Risk**（关于最难回答的问题的 1 句话）

展示所有三个。然后使用 `AskUserQuestion` 捕获选择。

**关键**：这必须是一个普通列表调用 —— 没有选项卡，没有表单字段。使用这个确切的结构：

```
AskUserQuestion(
  prompt: "哪个概念引起你的共鸣？你可以选择一个、组合元素，或要求新的方向。",
  options: [
    "Concept 1 — [Title]",
    "Concept 2 — [Title]",
    "Concept 3 — [Title]",
    "Combine elements across concepts",
    "Generate fresh directions"
  ]
)
```

不要在这里使用 `tabs` 字段。`tabs` 表单仅用于多字段输入 —— 在此处使用它会导致"Invalid tool parameters"错误。这是一个普通的 `prompt` + `options` 调用。

永远不要强迫选择 —— 让他们沉淀一下。

---

### 阶段 3：核心循环设计

对于选定的概念，使用结构化提问来构建核心循环。
核心循环是游戏跳动的心脏 —— 如果它本身不好玩，
再多的内容或打磨也救不了游戏。

**30 秒循环**（时刻到时刻）：

将这些作为 `AskUserQuestion` 调用询问 —— 从选定的概念推导选项，不要硬编码：

1. **核心动作感觉** —— 提示："核心动作的主要感觉是什么？"生成 3-4 个适合概念类型和基调的选项，加上一个自由文本转义（`I'll describe it`）。

2. **关键设计维度** —— 识别这个特定概念最重要的设计变量（例如，世界反应性、节奏、玩家代理）并询问它。生成与概念匹配的选项。始终包含一个自由文本转义。

捕获答案后，分析：这个动作本身是否令人满意？是什么让它感觉良好？（音频反馈、视觉 juice、时机满足感、战术深度？）

**5 分钟循环**（短期目标）：
- 什么将时刻到时刻的游戏构建成循环？
- "再来一轮" / "再来一次"心理在哪里发挥作用？
- 玩家在这个层面做出什么选择？

**会话循环**（30-120 分钟）：
- 一个完整的会话是什么样的？
- 自然的停止点在哪里？
- 什么"钩子"让他们在不玩游戏时想到游戏？

**进程循环**（天/周）：
- 玩家如何成长？（力量？知识？选择？故事？）
- 长期目标是什么？游戏何时"结束"？

**玩家动机分析**（基于自我决定理论）：
- **Autonomy**：玩家有多少有意义的选择？
- **Competence**：玩家如何感受他们的技能在增长？
- **Relatedness**：玩家如何感受联系（与角色、其他玩家或世界）？

---

### 阶段 4：支柱和边界

游戏支柱被真正的 AAA 工作室（God of War、Hades、The Last of
Us）使用，让数百名团队成员做出都指向同一方向的决策。即使对于独立开发者，支柱也能防止范围蔓延并保持愿景清晰。

协作定义 **3-5 个支柱**：
- 每个支柱有一个**名称**和**一句话定义**
- 每个支柱有一个**设计测试**："如果我们在 X 和 Y 之间争论，
  这个支柱说我们选择 __"
- 支柱应该感觉它们在相互制造张力 —— 如果所有
  支柱指向同一个方向，它们没有做足够的工作

然后定义 **3+ 个反支柱**（这个游戏不是什么）：
- 反支柱防止最常见的范围蔓延形式："如果...不是很好吗..."的功能不服务于核心愿景
- 构建为："我们不会做 [事情]，因为它会损害 [支柱]"

**支柱确认**：展示完整的支柱集后，使用 `AskUserQuestion`：
- 提示："这些支柱对你的游戏感觉对吗？"
- 选项：`[A] Lock these in` / `[B] Rename or reframe one` / `[C] Swap a pillar out` / `[D] Something else`

如果用户选择 B、C 或 D，进行修改，然后再次使用 `AskUserQuestion`：
- 提示："支柱已更新。准备好锁定这些了吗？"
- 选项：`[A] Lock these in` / `[B] Revise another pillar` / `[C] Something else`

重复直到用户选择 [A] Lock these in。

**审查模式检查** —— 在派生 CD-PILLARS 和 AD-CONCEPT-VISUAL 之前应用：
- `solo` → 跳过两者。注意："CD-PILLARS skipped —— Solo mode。AD-CONCEPT-VISUAL skipped —— Solo mode。" 继续阶段 5。
- `lean` → 跳过两者（不是 PHASE-GATE）。注意："CD-PILLARS skipped —— Lean mode。AD-CONCEPT-VISUAL skipped —— Lean mode。" 继续阶段 5。
- `full` → 正常派生。

**在支柱和反支柱达成一致后，在继续阶段 5 之前，同时通过 Task 派生 BOTH `creative-director` AND `art-director`。同时发出两个 Task 调用 —— 不要等待一个完成再开始另一个。**

- **`creative-director`** —— 门 **CD-PILLARS**（`.claude/docs/director-gates.md`）
  传入：带有设计测试的完整支柱集、反支柱、核心幻想、独特钩子。

- **`art-director`** —— 门 **AD-CONCEPT-VISUAL**（`.claude/docs/director-gates.md`）
  传入：游戏概念电梯演讲、带有设计测试的完整支柱集、目标平台（如果已知）、用户提到的任何参考游戏或视觉标志。

收集两个裁决，然后使用两选项卡 `AskUserQuestion` 一起展示它们：
- 选项卡 **"Pillars"**：展示 creative-director 反馈。选项镜像标准 CD-PILLARS 处理 —— `Lock in as-is` / `Revise [specific pillar]` / `Discuss further`。
- 选项卡 **"Visual anchor"**：展示 art-director 的 2-3 个命名视觉方向选项。选项：每个命名方向（每个选项一个）+ `Combine elements across directions` + `Describe my own direction`。

用户选择的视觉方向（命名方向或他们的自定义描述）存储为 **Visual Identity Anchor** —— 它将被写入 game-concept 文档并成为艺术圣经的基础。

如果 creative-director 对支柱返回 CONCERNS 或 REJECT，在询问视觉选择之前解决支柱问题 —— 视觉方向应该从确认的支柱流出。

---

### 阶段 5：玩家类型验证

使用 Bartle 分类法和 Quantic Foundry 动机模型，验证
这个游戏实际上是为谁准备的：

- **主要玩家类型**：谁会爱这个游戏？（Achievers、Explorers、
  Socializers、Competitors、Creators、Storytellers）
- **次要吸引力**：还有谁可能喜欢它？
- **这不适合谁**：清楚谁不会喜欢这个游戏与知道谁会喜欢一样重要
- **市场验证**：是否有成功的游戏服务类似的
  玩家类型？我们可以从他们的受众规模中学到什么？

---

### 阶段 6：范围和可行性

将概念建立在现实中：

- **目标平台**：使用 `AskUserQuestion` —— "你针对哪些平台开发这个游戏？"
  选项：`PC (Steam / Epic)` / `Mobile (iOS / Android)` / `Console` / `Web / Browser` / `Multiple platforms`
  记录答案 —— 它直接影响引擎建议并将传递给 `/setup-engine`。
  如果相关，注意平台影响（例如，mobile 意味着 Unity 是强烈首选；console 意味着 Godot 有限制；web 意味着 Godot 导出干净）。

- **引擎经验**：使用 `AskUserQuestion` —— "你已经有使用的引擎了吗？"
  选项：`Godot` / `Unity` / `Unreal Engine 5` / `No preference —— help me decide`
  - 如果他们选择一个引擎 → 记录为他们的偏好并继续。不要质疑它。
  - 如果"No preference" → 告诉他们："此次会话后运行 `/setup-engine` —— 它将根据你的概念和平台目标引导你完成整个决策。" 不要在这里做推荐。
- **艺术管线**：艺术风格是什么，劳动密集度如何？
- **内容范围**：估计关卡/区域数量、物品数量、游戏小时数
- **MVP 定义**：测试"核心循环有趣吗？"的绝对最小构建是什么？
- **最大风险**：技术风险、设计风险、市场风险
- **范围层级**：完整愿景是什么 vs. 如果时间耗尽会发布什么？

**审查模式检查** —— 在派生 TD-FEASIBILITY 之前应用：
- `solo` → 跳过。注意："TD-FEASIBILITY skipped —— Solo mode。" 直接继续范围层级定义。
- `lean` → 跳过（不是 PHASE-GATE）。注意："TD-FEASIBILITY skipped —— Lean mode。" 直接继续范围层级定义。
- `full` → 正常派生。

**在识别最大技术风险后，在定义范围层级之前，使用门 TD-FEASIBILITY（`.claude/docs/director-gates.md`）通过 Task 派生 `technical-director`。**

传入：核心循环描述、平台目标、引擎选择（或"undecided"）、已识别的技术风险列表。

向用户展示评估。如果 HIGH RISK，提供在最终确定前重新审视范围。如果 CONCERNS，记录它们并继续。

**审查模式检查** —— 在派生 PR-SCOPE 之前应用：
- `solo` → 跳过。注意："PR-SCOPE skipped —— Solo mode。" 继续文档生成。
- `lean` → 跳过（不是 PHASE-GATE）。注意："PR-SCOPE skipped —— Lean mode。" 继续文档生成。
- `full` → 正常派生。

**在定义范围层级后，使用门 PR-SCOPE（`.claude/docs/director-gates.md`）通过 Task 派生 `producer`。**

传入：完整愿景范围、MVP 定义、时间线估计、团队规模。

向用户展示评估。如果 UNREALISTIC，提供在写入文档之前调整 MVP 定义或范围层级。

---

4. **生成游戏概念文档**，使用 `.claude/docs/templates/game-concept.md` 中的模板。从头脑风暴对话中填写**所有**部分，包括 MDA 分析、玩家动机档案和心流设计部分。

   **在游戏概念文档中包含 Visual Identity Anchor 部分**，包含：
   - 选定的视觉方向名称
   - 一行视觉规则
   - 带有设计测试的 2-3 个支持视觉原则
   - 颜色哲学摘要

   这个部分是艺术圣经的种子 —— 它捕捉了"一切必须
   移动"的决策，以免在会话之间被遗忘。

5. 使用 `AskUserQuestion` 进行写入批准：
- 提示："游戏概念已准备好。我可以写入 `design/gdd/game-concept.md` 吗？"
- 选项：`[A] Yes —— write it` / `[B] Not yet —— revise a section first`

如果 [B]：使用带有选项的 `AskUserQuestion` 询问要修改哪个部分：`Elevator Pitch` / `Core Fantasy & Unique Hook` / `Pillars` / `Core Loop` / `MVP Definition` / `Scope Tiers` / `Risks` / `Something else —— I'll describe`

修改后，将更新的部分显示为差异或清晰的前后对比，然后使用 `AskUserQuestion` —— "准备好写入更新的概念文档了吗？"
选项：`[A] Yes —— write it` / `[B] Revise another section`
重复直到用户选择 [A]。

如果是，使用 `.claude/docs/templates/game-concept.md` 中的模板生成文档，从头脑风暴对话中填写**所有**部分，并写入文件，根据需要创建目录。

**范围一致性规则**：Core Identity 表中的"Estimated Scope"字段必须与 Scope Tiers 部分的完整愿景时间线匹配 —— 不仅仅是说"Large (9+ months)"。将其写为"Large (X–Y months, solo)"或"Large (X–Y months, team of N)"，以便摘要表准确。

6. **建议下一步**（按此顺序 —— 这是专业工作室的
   预制作管道）。列出所有步骤 —— 不要缩写或截断：

**路径 A —— 设计优先**（如果概念定义良好则推荐）：
   1. "运行 `/setup-engine` 配置引擎并填充版本感知参考文档"
   2. "运行 `/art-bible` 创建视觉身份规范 —— 在编写 GDD 之前执行此操作。**艺术圣经在 Technical Setup 门前是必需的。** 它控制资产制作并塑造技术架构决策（渲染、VFX、UI 系统）。"
   3. "使用 `/design-review design/gdd/game-concept.md` 在继续下游之前验证概念的完整性"
   4. "与 `creative-director` 代理讨论愿景以进行支柱完善"
   5. "使用 `/map-systems` 将概念分解为独立系统 —— 映射依赖关系、分配优先级并创建系统索引"
   6. "使用 `/design-system` 编写每个系统的 GDD —— 为步骤 5 中识别的每个系统提供引导式的、逐部分的 GDD 编写"
   7. "使用 `/create-architecture` 规划技术架构 —— 生成主架构蓝图和所需 ADR 列表"
   8. "使用 `/architecture-decision (×N)` 记录关键架构决策 —— 为 `/create-architecture` 中所需 ADR 列表中的每个决策编写一个 ADR"
   9. "运行 `/architecture-review` —— 从你的 GDD 和 ADR 引导 TR registry 和 Requirements Traceability Matrix（在 Pre-Production 门前必需）"
   10. "使用 `/gate-check` 验证准备就绪以推进 —— 在投入生产前的阶段门"

**路径 B —— 原型优先**（如果核心机制未经证明或概念需要验证则使用）：
   1. "运行 `/setup-engine` 配置引擎"
   2. "运行 `/prototype [core-mechanic]` —— 在编写任何 GDD 之前验证核心想法是否有趣（1-3 天的可丢弃代码）"
   3. "如果原型 PROCEEDS：运行 `/art-bible`，然后继续使用路径 A 步骤 5-10，使用原型学习来指导你的 GDD"
   4. "如果原型 PIVOTS：返回 `/brainstorm` 并带着学习重塑概念"
   5. "完整设计和架构后，构建 `/vertical-slice` 在投入 sprint 之前验证生产准备就绪"

7. **输出摘要**，包含所选概念的电梯演讲、支柱、主要玩家类型、引擎建议、最大风险和文件路径。

裁决：**COMPLETE** —— 游戏概念已创建并移交下一步。

---

## 上下文窗口意识

这是一个多阶段 skill。如果上下文在任何一个阶段期间达到或超过 70%，
在继续之前将此通知附加到当前响应：

> **上下文正在接近限制（≥70%）。** 游戏概念文档已保存
> 到 `design/gdd/game-concept.md`。如果需要，打开一个新的 Claude Code 会话继续
> —— 进度不会丢失。

---

## 推荐的下一步

写入游戏概念后，按顺序遵循预制作管道：
1. `/setup-engine` —— 配置引擎并填充版本感知参考文档
2. `/art-bible` —— 在编写任何 GDD 之前建立视觉身份
3. `/map-systems` —— 将概念分解为带有依赖关系的独立系统
4. `/design-system [first-system]` —— 按依赖关系顺序编写每个系统的 GDD
5. `/create-architecture` —— 生成主架构蓝图
6. `/architecture-review` —— 引导 TR registry 和 Requirements Traceability Matrix
7. `/gate-check pre-production` —— 在投入生产前验证准备就绪
