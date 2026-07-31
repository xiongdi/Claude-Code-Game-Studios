---
name: create-control-manifest
description: "After architecture is complete, produces a flat actionable rules sheet for programmers — what you must do, what you must never do, per system and per layer. Extracted from all Accepted ADRs, technical preferences, and engine reference docs. More immediately actionable than ADRs (which explain why)."
argument-hint: "[update — regenerate from current ADRs]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Task
model: sonnet
agent: technical-director
---

# Create Control Manifest

Control Manifest 是一份扁平化、可操作的程序员规则表。它回答"我该做什么？"和"我绝不能做什么？"——按架构层级组织，从所有 Accepted ADR、技术偏好和引擎参考文档中提取。ADR 解释*原因*，而 manifest 告诉你*做什么*。

**输出：** `docs/architecture/control-manifest.md`

**何时运行：** `/architecture-review` 通过且 ADR 处于 Accepted 状态后。每当有新 ADR 被接受或现有 ADR 被修订时重新运行。

---

## 1. 加载所有输入

### ADR
- Glob `docs/architecture/adr-*.md` 并读取每个文件
- 仅筛选 Accepted ADR（Status: Accepted）——跳过 Proposed、Deprecated、Superseded
- 记录每条规则来源的 ADR 编号和标题

### 技术偏好
- 读取 `.claude/docs/technical-preferences.md`
- 提取：命名约定、性能预算、已批准的库/插件、禁止的模式

### 引擎参考
- 读取 `docs/engine-reference/[engine]/VERSION.md` 获取引擎 + 版本
- 读取 `docs/engine-reference/[engine]/deprecated-apis.md`——这些成为禁止的 API 条目
- 如果存在，读取 `docs/engine-reference/[engine]/current-best-practices.md`

报告："已加载 [N] 个 Accepted ADR，引擎：[name + version]。"

---

## 2. 从每个 ADR 提取规则

对于每个 Accepted ADR，提取：

### 必需模式（来自 "Implementation Guidelines" 部分）
- 每个 "must"、"should"、"required to"、"always" 语句
- 每个强制要求的特定模式或方法

### 禁止的方法（来自 "Alternatives Considered" 部分）
- 每个被明确拒绝的替代方案——*为什么*被拒绝成为规则（"绝不要使用 X，因为 Y"）
- 任何明确指出的反模式

### 性能护栏（来自 "Performance Implications" 部分）
- 预算约束："此系统每帧最多 N 毫秒"
- 内存限制："此系统不得超过 N MB"

### 引擎 API 约束（来自 "Engine Compatibility" 部分）
- 需要验证的 cutoff 后 API
- 与默认 LLM 假设不同的已验证行为
- 在固定引擎版本中有不同行为的 API 字段或方法

### 层级分类
按系统所属的架构层级对每条规则进行分类：
- **Foundation**：场景管理、事件架构、save/load、引擎初始化
- **Core**：核心玩法循环、主要玩家系统、物理/碰撞
- **Feature**：辅助系统、辅助机制、AI
- **Presentation**：渲染、音频、UI、VFX、shader

如果 ADR 跨越多个层级，将规则复制到每个相关层级。

---

## 3. 添加全局规则

合并适用于所有层级的规则：

### 来自 technical-preferences.md：
- 命名约定（类、变量、信号/事件、文件、常量）
- 性能预算（目标帧率、帧预算、绘制调用限制、内存上限）

### 来自 deprecated-apis.md：
- 所有已弃用 API → 禁止 API 条目

### 来自 current-best-practices.md（如果可用）：
- 引擎推荐模式 → 必需条目

### 来自 technical-preferences.md 的禁止模式：
- 直接复制任何 "Forbidden Patterns" 条目

---

## 4. 写入前展示规则摘要

在写入 manifest 之前，向用户展示摘要：

```
## Control Manifest Preview
引擎：[name + version]
涵盖的 ADR：[列出 ADR 编号]
提取的规则总数：
  - Foundation 层：[N] 必需，[M] 禁止，[P] 护栏
  - Core 层：[N] 必需，[M] 禁止，[P] 护栏
  - Feature 层：...
  - Presentation 层：...
  - 全局：[N] 命名约定，[M] 禁止 API，[P] 已批准库
```

使用 `AskUserQuestion`：
- 提示："此规则摘要看起来完整吗？"
- 选项：
  - `[A] 是 — 看起来不错，运行主管审查并写入 manifest`
  - `[B] 添加规则 — 我在写入前有额外的规则要包含`
  - `[C] 移除规则 — 某些提取的规则应该删除`
  - `[D] 停在这里 — 我需要先审查 ADR`

---

## 4b. 主管门 — 技术审查

**审查模式检查** — 在生成 TD-MANIFEST 之前应用：
- `solo` → 跳过。注意："TD-MANIFEST 已跳过 — Solo 模式。" 进入第 5 阶段。
- `lean` → 跳过。注意："TD-MANIFEST 已跳过 — Lean 模式。" 进入第 5 阶段。
- `full` → 正常生成。

通过 Task 使用门 **TD-MANIFEST**（`.claude/docs/director-gates.md`）生成 `technical-director`。

传递：第 4 阶段的 Control Manifest Preview（每层规则计数、完整提取规则列表）、涵盖的 ADR 列表、引擎版本、以及来自 technical-preferences.md 或引擎参考文档的任何规则。

technical-director 审查：
- 所有强制 ADR 模式是否被准确捕获和陈述
- 禁止的方法是否完整且正确归属
- 没有添加缺乏来源 ADR 或偏好文档的规则
- 性能护栏是否与 ADR 约束一致

应用裁决：
- **APPROVE** → 进入第 5 阶段
- **CONCERNS** → 通过 `AskUserQuestion` 展示，选项：`修改标记的规则` / `接受并继续` / `进一步讨论`
- **REJECT** → 不写入 manifest；修复标记的规则并重新展示摘要

---

## 5. 写入 Control Manifest

使用 `AskUserQuestion`：
- 提示："我可以写入 Control Manifest 吗？"
- 选项：
  - `[A] 是 — 写入 docs/architecture/control-manifest.md`
  - `[B] 先展示完整草稿，然后再问一次`
  - `[C] 还不行 — 我想做更多更改`

格式：

```markdown
# Control Manifest

> **引擎**：[name + version]
> **最后更新**：[date]
> **Manifest 版本**：[date]
> **涵盖的 ADR**：[ADR-NNNN, ADR-MMMM, ...]
> **状态**：[Active — ADR 变更时用 `/create-control-manifest update` 重新生成]

`Manifest 版本` 是此 manifest 生成的日期。story 文件在创建时嵌入此日期。
`/story-readiness` 比较 story 的嵌入版本与此字段，以检测基于过期规则编写的 story。
始终与 `Last Updated` 匹配——它们是同一日期，服务于不同的消费者。

此 manifest 是从所有 Accepted ADR、技术偏好和引擎参考文档中提取的程序员快速参考。
关于每条规则背后的推理，请参见引用的 ADR。

---

## Foundation 层规则

*适用于：scene management、event architecture、save/load、引擎初始化*

### 必需模式
- **[rule]** — 来源：[ADR-NNNN]
- **[rule]** — 来源：[ADR-NNNN]

### 禁止的方法
- **绝不要 [anti-pattern]** — [简要原因] — 来源：[ADR-NNNN]

### 性能护栏
- **[system]**：最多 [N]ms/frame — 来源：[ADR-NNNN]

---

## Core 层规则

*适用于：核心玩法循环、主要玩家系统、物理、碰撞*

### 必需模式
...

### 禁止的方法
...

### 性能护栏
...

---

## Feature 层规则

*适用于：辅助机制、AI 系统、辅助功能*

### 必需模式
...

### 禁止的方法
...

---

## Presentation 层规则

*适用于：渲染、音频、UI、VFX、shader、动画*

### 必需模式
...

### 禁止的方法
...

---

## 全局规则（所有层级）

### 命名约定
| 元素 | 约定 | 示例 |
|---------|-----------|---------|
| 类 | [来自 technical-preferences] | [示例] |
| 变量 | [来自 technical-preferences] | [示例] |
| 信号/事件 | [来自 technical-preferences] | [示例] |
| 文件 | [来自 technical-preferences] | [示例] |
| 常量 | [来自 technical-preferences] | [示例] |

### 性能预算
| 目标 | 值 |
|--------|-------|
| 帧率 | [来自 technical-preferences] |
| 帧预算 | [来自 technical-preferences] |
| 绘制调用 | [来自 technical-preferences] |
| 内存上限 | [来自 technical-preferences] |

### 已批准的库 / 插件
- [library] — 批准用于 [purpose]

### 禁止 API（[engine version]）
这些 API 在 [engine + version] 中已弃用或未验证：
- `[api name]` — 自 [version] 起弃用 / cutoff 后未验证
- 来源：`docs/engine-reference/[engine]/deprecated-apis.md`

### 跨领域约束
- [无论层级如何，到处适用的约束]
```

---

## 6. 建议后续步骤

写入 manifest 后：

- 如果 epic/story 尚不存在："运行 `/create-epics layer: foundation` 然后 `/create-stories [epic-slug]`——程序员现在可以在编写 story 实现备注时使用此 manifest。"
- 如果是重新生成（manifest 已存在）："已更新。建议通知团队规则变更——特别是任何新的禁止条目。"

---

## 协作协议

1. **静默加载** — 在展示任何内容之前读取所有输入
2. **先展示摘要** — 让用户在写入前看到范围
3. **写入前询问** — 在创建或覆盖 manifest 之前始终确认。写入时：裁决：**COMPLETE** — control manifest 已写入。拒绝时：裁决：**BLOCKED** — 用户拒绝写入。
4. **每条规则都要有来源** — 绝不添加不追溯到 ADR、技术偏好或引擎参考文档的规则
5. **不解释** — 按 ADR 中陈述的方式提取规则；不要以改变含义的方式意译
