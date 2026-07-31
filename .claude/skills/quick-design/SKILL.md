---
name: quick-design
description: "Lightweight design spec for small changes — tuning adjustments, minor mechanics, balance tweaks. Skips full GDD authoring when a system GDD already exists or the change is too small to warrant one. Produces a Quick Design Spec that embeds directly into story files."
argument-hint: "[brief description of the change]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
model: sonnet
---

# Quick Design

这是不需要完整 GDD 的变更的**轻量级设计路径**。
通过 `/design-system` 进行完整的 GDD 编写是重量级路径。对于实现时间约 4 小时以下的工作使用此 skill — 调优调整、
小行为变更、对现有系统的添加，或太小而不需要完整文档的独立功能。

**输出：** `design/quick-specs/[名称]-[日期].md`

**何时运行：** 当变更对 `/design-system` 来说太小但
在没有书面理由的情况下实现又太有意义时。

---

## 1. 分类变更

首先，读取参数并确定此变更属于哪个类别：

- **Tuning** — 更改现有系统中的数值或平衡值，无
  行为变更（最轻量的路径）。示例："将跳跃高度从 5
  增加到 6 单位"、"将敌人巡逻速度降低 10%"。
- **Tweak** — 对现有系统的小行为变更，不引入新
  状态、分支或系统。示例："让冲刺在第 1 帧无敌"、
  "允许连招取消为翻滚"。
- **Addition** — 向现有系统添加小机制，可能引入
  1-2 个新状态或交互。示例："为格挡机制添加弹反窗口"、
  "为普通攻击添加蓄力变体"。
- **New Small System** — 足够小的独立功能，它没有
  现有 GDD 且实现时间约一周以下。示例："成就弹窗系统"、
  "简单的昼夜视觉循环"。

如果变更**不**适合这些类别 — 它引入了具有显著跨系统依赖的新系统，需要超过一周的实现，或从根本上改变了现有系统的核心规则 — 停止并改为重定向到 `/design-system`。

如果没有参数，请用户描述变更（纯文本提示），然后使用上述标准对其进行分类。

使用 `AskUserQuestion` 展示推断的分类：
- 提示："我将其分类为 **[推断类型]** — [简要原因]。正确吗？"
- 选项：
  - `[A] 是 — [推断类型] 正确`
  - `[B] Tuning — 仅更改数值或平衡值`
  - `[C] Tweak — 对现有系统的小行为变更`
  - `[D] Addition — 向现有系统添加小机制`
  - `[E] New Small System — 独立功能，一周以下工作量`
  - `[F] 这太大了 — 将我重定向到 /design-system`

如果选择 [F]：停止。判定：**REDIRECTED** — 对此变更使用 `/design-system`。
否则：继续所选类型。

---

## 2. 上下文扫描

在起草任何内容之前，阅读相关上下文：

- 搜索 `design/gdd/` 获取与此变更最相关的 GDD。阅读此变更会影响的部分。
- 检查 `design/gdd/systems-index.md` 是否存在。如果存在，阅读它以了解该系统在依赖图中的位置以及属于哪个层级。如果不存在，记录"未找到 systems index — 跳过依赖层级检查。"并继续。
- 检查 `design/quick-specs/` 中是否有触及此系统的先前 quick spec — 避免与它们矛盾。
- 如果这是 Tuning 变更，还要检查 `assets/data/` 中保存相关值的数据文件。

报告发现的内容："在 [路径] 找到 GDD。相关部分：[部分名称]。
未发现冲突的 quick spec。"（或记录发现的任何冲突。）

---

## 3. 起草 Quick Design Spec

使用适合变更类别的规范格式。

### 对于 Tuning 变更

生成单个表格：

```markdown
# Quick Design Spec: [标题]

**类型**: Tuning
**系统**: [系统名称]
**GDD 引用**: `design/gdd/[文件名].md` — Tuning Knobs 部分
**日期**: [今天]

## 变更

| 参数 | 旧值 | 新值 | 理由 |
|-----------|-----------|-----------|-----------|
| [参数]   | [旧]     | [新]     | [为什么]     |

## Tuning Knob 映射

映射到 GDD Tuning Knob: [旋钮名称及其文档记录的范围]。
新值在文档记录范围的 [内部 / 边缘 / 外部]。
[如果在外部：解释为什么应该扩展范围。]

## 验收标准

- [ ] [参数] 从 `assets/data/[文件]` 读取 [新值]
- [ ] 行为差异在 [具体上下文] 中可观察
- [ ] [相关行为] 无回归
```

### 对于 Tweak 和 Addition 变更

```markdown
# Quick Design Spec: [标题]

**类型**: [Tweak / Addition]
**系统**: [系统名称]
**GDD 引用**: `design/gdd/[文件名].md`
**日期**: [今天]

## 变更摘要

[1-2 句话描述变更了什么以及为什么。]

## 动机

[为什么需要此变更？它解决了什么玩家体验问题？
如适用，引用相关的 MDA 美学或玩家反馈。]

## 设计增量

当前 GDD 说（引用 `design/gdd/[文件名].md`，[部分]）：

> [相关规则或描述的准确引用]

此规范将其更改为：

[新规则或描述，以与 GDD Detailed Rules 部分相同的精度编写。
程序员应该能够仅从此文本实现。]

## 新规则 / 值

[替换内容的完整明确陈述。如果引入新状态，列出它们。如果引入新参数，定义它们的范围。]

## 受影响的系统

| 系统 | 影响 | 所需操作 |
|--------|--------|-----------------|
| [系统] | [如何受影响] | [更新 GDD / 更新数据文件 / 无操作] |

## 验收标准

- [ ] [具体、可测试的标准 1]
- [ ] [具体、可测试的标准 2]
- [ ] [具体、可测试的标准 3]
- [ ] 无回归：[此操作不得破坏的原始行为]

## 需要更新 GDD？

[是 / 否]
[如果是：哪个文件、哪个部分以及更新应该说什么。]
```

### 对于 New Small System 变更

使用精简的 GDD 结构。仅包含直接必要的部分 — 跳过 Player Fantasy、完整 Formulas 和 Edge Cases，除非系统特别需要它们。

```markdown
# Quick Design Spec: [标题]

**类型**: New Small System
**范围**: [1-2 句话描述此系统做什么和不做什么]
**日期**: [今天]
**估算实现**: [小时]

## 概述

[新团队成员可以理解的一段话。这个系统做什么，何时激活，产生什么？]

## 核心规则

[系统明确无歧义的规则。对顺序行为使用编号列表，对条件使用要点列表。足够精确，程序员可以实现而无需提问。]

## Tuning Knobs

| 旋钮 | 默认值 | 范围 | 类别 | 理由 |
|------|---------|-------|----------|-----------|
| [名称] | [值] | [最小-最大] | [感觉/曲线/门槛] | [为什么选择此默认值] |

所有值必须存在于 `assets/data/[适当文件].json` 中，不得硬编码。

## 验收标准

- [ ] [功能标准：做正确的事]
- [ ] [功能标准：处理边缘情况]
- [ ] [体验标准：感觉正确 — playtest 验证什么]
- [ ] [回归标准：不破坏相邻系统]

## Systems Index

此系统当前不在 `design/gdd/systems-index.md` 中。
[如果应该添加：建议哪个层级和优先级。]
[如果太小无法跟踪：声明"此系统低于 systems-index
跟踪阈值 — quick spec 足够。"]
```

---

## 4. 批准和归档

向用户完整展示草稿。然后使用 `AskUserQuestion`：
- 提示："这是 Quick Design Spec 草稿。您想如何继续？"
- 选项：
  - `[A] 批准 — 按所示写入`
  - `[B] 修订 — 我会描述要更改的内容`
  - `[C] 这变得太大了 — 改为重定向到 /design-system`

如果选择 [B]：收集请求的更改，修订草稿，并重新展示此小部件。
如果选择 [C]：停止。判定：**REDIRECTED** — 对此变更使用 `/design-system`。

如果选择 [A]：询问"我可以将此 Quick Design Spec 写入
`design/quick-specs/[kebab-case-标题]-[YYYY-MM-DD].md` 吗？"

文件名中使用今天的日期。标题应为变更的 kebab-case 描述
（例如，`jump-height-tuning-2026-03-10`、
`parry-window-addition-2026-03-10`）。

如果同意，创建 `design/quick-specs/` 目录（如果不存在），然后
写入文件。

如果需要更新 GDD（在规范中标记），在写入 quick spec 后单独询问：

"此规范修改了 [系统名称] 中的规则。我可以更新
`design/gdd/[文件名].md` — 特别是 [部分名称] 部分吗？"

在询问之前展示将要更改的确切文本（旧 vs 新）。不要
未经明确批准就进行 GDD 编辑。

---

## 5. 交接

写入文件后，输出：

```
Quick Design Spec 已写入: design/quick-specs/[文件名].md
类型: [Tuning / Tweak / Addition / New Small System]
系统: [系统名称]
GDD 更新: [需要 — 待批准 / 已应用 / 不需要]

后续步骤: 此规范已准备好在实现前进行 `/story-readiness` 验证。
在故事的 GDD Reference 字段中引用此规范。
```

### 管道说明

判定：**COMPLETE** — quick design spec 已写入并准备好实现。

Quick Design Spec **绕过** `/design-review` 和 `/review-all-gdds`，
按设计。它们适用于小的、低风险的、范围明确的工作，其中完整审查管道的成本超过变更本身的风险。

如果以下任何条件为真，则重定向到完整管道：
- 变更添加了属于 systems index 的新系统
- 变更显著改变了跨系统行为或系统与其他系统的契约
- 变更引入了影响游戏 MDA 美学平衡的新面向玩家的机制
- 实现可能超过一周的工作量

在这些情况下："此变更已超出 quick spec 范围。我推荐
使用 `/design-system` 为此编写完整的 GDD。"

---

## 推荐后续步骤

- 运行 `/story-readiness [故事路径]` 在实现开始前验证故事 — 在故事的 GDD Reference 字段中引用此规范
- 运行 `/dev-story [故事路径]` 在故事通过就绪检查后实现
- 如果变更比预期大，运行 `/design-system [系统名称]` 编写完整的 GDD
