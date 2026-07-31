---
name: reverse-document
description: "从现有实现生成设计或架构文档。从代码/原型反向推导，创建缺失的规划文档。"
argument-hint: "<type> <path> (例如 'design src/gameplay/combat' 或 'architecture src/core')"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash
model: sonnet
# 只读诊断技能 — 无需委派给专家 Agent
---

# 反向文档化

此技能分析现有实现（代码、原型、系统）并生成相应的设计或架构文档。在以下情况下使用：
- 你在没有先写设计文档的情况下构建了功能
- 你继承了没有文档的代码库
- 你制作了机制原型并需要将其正式化
- 你需要记录现有代码背后的"为什么"

---

## 工作流

## Phase 1: 解析参数

**格式**：`/reverse-document <type> <path>`

**类型选项**：
- `design` → 生成游戏设计文档（GDD 章节）
- `architecture` → 生成架构决策记录（ADR）
- `concept` → 从原型生成概念文档

**路径**：要分析的目录或文件
- `src/gameplay/combat/` → 所有战斗相关代码
- `src/core/event-system.cpp` → 特定文件
- `prototypes/stealth-mech/` → 原型目录

**示例**：
```bash
/reverse-document design src/gameplay/magic-system
/reverse-document architecture src/core/entity-component
/reverse-document concept prototypes/vehicle-combat
```

## Phase 2: 分析实现

**阅读并理解代码/原型**：

**对于设计文档（GDD）：**
- 识别机制、规则、公式
- 提取游戏数值（伤害、冷却、范围）
- 查找状态机、能力系统、进度系统
- 检测代码中处理的边缘情况
- 映射依赖关系（哪些系统交互？）

**对于架构文档（ADR）：**
- 识别模式（ECS、单例、观察者等）
- 理解技术决策（线程、序列化等）
- 映射依赖关系和耦合
- 评估性能特征
- 查找约束和权衡

**对于概念文档（原型分析）：**
- 识别核心机制
- 提取涌现式游戏模式
- 记录哪些有效 vs 哪些无效
- 查找技术可行性洞察
- 记录玩家幻想/手感

## Phase 3: 提出澄清性问题

**不要**只描述代码。**要**询问意图：

**设计问题**：
- "我看到一个 [资源] 系统在 [活动] 期间消耗。这是为了：
  - 节奏控制（防止滥用）？
  - 资源管理（策略深度）？
  - 还是其他目的？"
- "[机制] 似乎是核心。这是核心支柱还是辅助功能？"
- "[数值] 随 [因子] 指数增长。是刻意设计还是需要重新平衡？"

**架构问题**：
- "你使用了 service locator 模式。选择它是为了：
  - 可测试性（mock 依赖）？
  - 解耦（减少硬引用）？
  - 还是从现有代码继承的？"
- "我看到使用手动内存管理而非智能指针。是性能要求还是历史遗留？"

**概念问题**：
- "原型强调潜行而非战斗。这是预期的支柱吗？"
- "玩家似乎利用钩爪来加速。是功能还是 bug？"

## Phase 4: 展示发现

在起草之前，展示你的发现：

```
我已分析 [path]/。以下是我的发现：

已实现机制：
- [机制-a] 具有 [属性]（例如时间窗口、冷却）
- [机制-b]（例如两个状态之间的交互）
- [资源] 系统（在 [动作] 时消耗，在 [条件] 时恢复）
- [状态] 系统（积累，触发 [效果]）

发现的公式：
- [输出] = [使用发现变量的公式]
- [次要输出] = [公式]

意图不明确区域：
1. [资源] 系统 — 节奏控制还是资源管理？
2. [机制] — 核心支柱还是辅助功能？
3. [数值] 缩放 — 刻意设计还是需要调整？

在我起草设计文档之前，能否澄清这些问题？
```

等待用户澄清意图后再起草。

## Phase 5: 使用模板起草文档

根据类型，使用相应模板：

| 类型 | 模板 | 输出路径 |
|------|----------|-------------|
| `design` | `templates/design-doc-from-implementation.md` | `design/gdd/[system-name].md` |
| `architecture` | `templates/architecture-doc-from-code.md` | `docs/architecture/[decision-name].md` |
| `concept` | `templates/concept-doc-from-prototype.md` | `prototypes/[name]/CONCEPT.md` 或 `design/concepts/[name].md` |

**草稿结构**：
- 捕获**存在的内容**（机制、模式、实现）
- 记录**存在的原因**（与用户澄清的意图）
- 识别**缺失的内容**（未处理的边缘情况、设计空白）
- 标记**后续工作**（平衡调整、缺失功能）

## Phase 6: 展示草稿并请求审批

**协作协议**：
```
我已根据你的代码和澄清起草了 [system-name] 设计文档。

[展示关键章节：概览、机制、公式、设计意图]

我添加的内容：
- 根据你的澄清，将 [机制] 记录为"[意图]"
- 添加了代码中不存在的边缘情况（例如，如果 [资源] 在 [动作] 中途归零会怎样？）
- 标记了平衡问题：[缩放类型] 在 [边界条件] 处缩放

标记为不完整的章节：
- "[系统] 与 [其他系统] 的交互"（尚未完全实现）
- "[变体或功能]"（目前仅实现了 [子集]）

可以写入 design/gdd/[system-name].md 吗？
```

等待审批。用户可能在写入前要求修改。

## Phase 7: 写入带元数据的文档

审批通过后，使用特殊标记写入文件：

```markdown
---
status: reverse-documented
source: [path/]
date: [today]
verified-by: [User name]
---

# [System Name] 设计

> **注意**：本文档是从现有实现反向工程生成的。
> 它捕获了当前行为和澄清的设计意图。部分章节可能
> 不完整，因为实现是部分的或意图不明确。

[文档其余部分...]
```

## Phase 8: 标记后续工作

写入后，建议下一步：

```
✅ 已写入 design/gdd/combat-system.md

建议后续工作：
1. 对战斗公式运行 /balance-check（指数缩放问题）
2. 为体力系统架构决策创建 ADR
3. 实现缺失的边缘情况：
   - 连招中途体力耗尽行为
   - 闪避期间的受击硬直状态
4. 在武器种类实现后扩展设计文档

你想现在处理其中任何一项吗？
```

---

## 模板选择逻辑

| 如果分析... | 使用模板... | 因为... |
|----------------|-----------------|------------|
| `src/gameplay/*` | design-doc-from-implementation.md | 游戏机制 → GDD |
| `src/core/*`、`src/ai/*` | architecture-doc-from-code.md | 核心系统 → ADR |
| `prototypes/*` | concept-doc-from-prototype.md | 实验 → 概念文档 |
| `src/networking/*` | architecture-doc-from-code.md | 技术系统 → ADR |
| `src/ui/*` | design-doc-from-implementation.md | UI/UX → 设计规格 |

---

## 示例会话：反向文档化一个系统

```
User: /reverse-document design src/gameplay/[system]

Agent: 我将分析你的 [system] 实现以创建设计文档。
       [读取代码，发现机制、公式]

Agent: 我已分析 src/gameplay/[system]/。以下是我的发现：
       [展示机制、公式、不明确区域]

       在起草之前，能否澄清：
       1. [资源] 系统意图？
       2. [机制] 是核心支柱吗？
       3. [数值] 缩放 — 刻意的还是需要调整？

User: 1. [资源] 是为了节奏控制，防止 [不良行为]
      2. [机制] 是核心 — 我想要 [设计意图]
      3. 是的，指数增长太过分了，应该是线性的

Agent: 完美，这澄清了愿景。让我起草设计文档。
       [展示带有澄清意图的草稿]

       可以写入 design/gdd/[system-name].md 吗？

User: 是的，写入。

Agent: ✅ 已写入 design/gdd/[system-name].md
       ✅ 标记为 [REVERSE-DOCUMENTED]
       ✅ 标记 [数值] 缩放需要重新平衡

       下一步：
       - 将 [公式] 更新为 [修正后的缩放]
       - 运行 /balance-check 验证 [曲线]
       - 在 game-pillars.md 中将 [机制] 记录为核心支柱
```

---

## 协作协议

此技能遵循协作设计原则：

1. **先分析**：阅读代码，理解实现
2. **询问意图**：问"为什么"，而不仅仅是"什么"
3. **展示发现**：展示发现，突出不明确区域
4. **用户澄清**：区分意图和偶然
5. **起草文档**：基于现实 + 意图创建文档
6. **展示草稿**：展示关键章节，解释添加内容
7. **获取审批**："可以写入 [filepath] 吗？" 审批后：裁决：**COMPLETE** — 文档已生成。拒绝后：裁决：**BLOCKED** — 用户拒绝写入。
8. **标记后续工作**：建议相关工作，不自动执行

**永远不要假设意图。在记录"为什么"之前始终先询问。**
