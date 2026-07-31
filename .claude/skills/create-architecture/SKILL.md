---
name: create-architecture
description: "Guided, section-by-section authoring of the master architecture document for the game. Reads all GDDs, the systems index, existing ADRs, and the engine reference library to produce a complete architecture blueprint before any code is written. Engine-version-aware: flags knowledge gaps and validates decisions against the pinned engine version."
argument-hint: "[focus-area: full | layers | data-flow | api-boundaries | adr-audit] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Bash, AskUserQuestion, Task
model: sonnet
agent: technical-director
---

# Create Architecture

此 skill 生成 `docs/architecture/architecture.md` —— 将已批准的所有 GDD 转化为具体技术蓝图的主架构文档。它位于设计和实施之间，必须在 sprint 计划开始之前存在。

**与 `/architecture-decision` 的区别**：ADR 记录单个点决策。此 skill 创建赋予 ADR 上下文的整个系统蓝图。

解析审查模式（一次，存储供本次运行的所有门生成使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

完整检查模式见 `.claude/docs/director-gates.md`。

**参数模式：**
- **无参数 / `full`**：完整引导式遍历 —— 所有部分，从头到尾
- **`layers`**：仅关注系统层图
- **`data-flow`**：仅关注模块之间的数据流
- **`api-boundaries`**：仅关注 API 边界定义
- **`adr-audit`**：仅审计现有 ADR 的引擎兼容性缺口

---

## 阶段 0：加载所有上下文

在做任何其他事情之前，按此顺序加载完整的项目上下文：

### 0a. 引擎上下文（关键）

完整读取引擎参考库：

1. `docs/engine-reference/[engine]/VERSION.md`
   → 提取：引擎名称、版本、LLM 截止日期、post-cutoff 风险级别
2. `docs/engine-reference/[engine]/breaking-changes.md`
   → 提取：所有 HIGH 和 MEDIUM 风险变更
3. `docs/engine-reference/[engine]/deprecated-apis.md`
   → 提取：要避免的 API
4. `docs/engine-reference/[engine]/current-best-practices.md`
   → 提取：与训练数据不同的 post-cutoff 最佳实践
5. `docs/engine-reference/[engine]/modules/` 中的所有文件
   → 提取：每个域的当前 API 模式

如果未配置引擎，停止并提示：
> "未配置引擎。先运行 `/setup-engine`。在不知道你针对的引擎和版本的情况下，
> 无法编写架构。"

### 0b. 设计上下文 + 技术需求提取

读取所有已批准的设计文档并从每个文档中提取技术需求：

1. `design/gdd/game-concept.md` —— 游戏支柱、类型、核心循环
2. `design/gdd/systems-index.md` —— 所有系统、依赖关系、优先级层级
3. `.claude/docs/technical-preferences.md` —— 命名约定、性能预算、
   允许的库、禁止的模式
4. **`design/gdd/` 中的每个 GDD** —— 对于每个，提取技术需求：
   - 游戏规则暗示的数据结构
   - 陈述或暗示的性能约束
   - 系统所需的引擎能力
   - 跨系统通信模式（什么与什么通信，如何通信）
   - 必须持续的状态（保存/加载影响）
   - 线程或计时需求

构建一个**技术需求基线** —— 跨所有 GDD 的所有提取需求的平面列表，编号为 `TR-[gdd-slug]-[NNN]`。这是架构必须覆盖的完整集合。展示为：

```
## Technical Requirements Baseline
Extracted from [N] GDDs | [X] total requirements

| Req ID | GDD | System | Requirement | Domain |
|--------|-----|--------|-------------|--------|
| TR-combat-001 | combat.md | Combat | Hitbox detection per-frame | Physics |
| TR-combat-002 | combat.md | Combat | Combo state machine | Core |
| TR-inventory-001 | inventory.md | Inventory | Item persistence | Save/Load |
```

此基线输入每个后续阶段。到本会话结束时，不应有任何 GDD 需求
没有架构决策来支持它。

### 0c. 现有架构决策

读取 `docs/architecture/` 中的所有文件以了解已经决定的内容。
列出找到的任何 ADR 及其域。

### 0d. 生成知识缺口清单

在继续之前，展示一个结构化摘要：

```
## Engine Knowledge Gap Inventory
Engine: [name + version]
LLM Training Covers: up to approximately [version]
Post-Cutoff Versions: [list]

### HIGH RISK Domains (must verify against engine reference before deciding)
- [Domain]: [Key changes]

### MEDIUM RISK Domains (verify key APIs)
- [Domain]: [Key changes]

### LOW RISK Domains (in training data, likely reliable)
- [Domain]: [no significant post-cutoff changes]

### Systems from GDD that touch HIGH/MEDIUM risk domains:
- [GDD system name] → [domain] → [risk level]
```

使用 `AskUserQuestion`：
- 提示："一个或多个引擎域是 HIGH RISK —— LLM 的知识对于这些领域可能不可靠。这些域中的架构建议在采取行动之前应与引擎文档交叉引用。你希望如何继续？"
- 选项：
  - `[A] Proceed — flag HIGH RISK domains throughout the output`
  - `[B] Let me check the engine reference first — pause here`
  - `[C] Show me which domains are HIGH RISK and why`

---

## 阶段 1：系统层映射

将 `systems-index.md` 中的每个系统映射到架构层。标准
游戏架构层是：

```
┌─────────────────────────────────────────────┐
│  PRESENTATION LAYER                         │  ← UI、HUD、菜单、VFX、音频
├─────────────────────────────────────────────┤
│  FEATURE LAYER                              │  ← 游戏系统、AI、任务
├─────────────────────────────────────────────┤
│  CORE LAYER                                 │  ← 物理、输入、战斗、移动
├─────────────────────────────────────────────┤
│  FOUNDATION LAYER                           │  ← 引擎集成、保存/加载、
│                                             │    场景管理、事件总线
├─────────────────────────────────────────────┤
│  PLATFORM LAYER                             │  ← OS、硬件、引擎 API 表面
└─────────────────────────────────────────────┘
```

对于每个 GDD 系统，询问：
- 它属于哪一层？
- 它的模块边界是什么？
- 它独占拥有什么？（数据、状态、行为）

展示提议的层分配并在继续下一部分之前请求批准。立即将批准的层图写入骨架文件。

**引擎意识检查**：对于分配到 Core 和 Foundation 层的每个系统，标记它是否触及 HIGH 或 MEDIUM 风险引擎域。内联显示相关的引擎参考摘录。

---

## 阶段 2：模块所有权映射

对于阶段 1 中定义的每个模块，定义所有权：

- **Owns**：此模块全权负责什么数据和状态
- **Exposes**：其他模块可以读取或调用什么
- **Consumes**：它从其他模块读取什么
- **Engine APIs used**：此模块直接调用哪些特定的引擎类/节点/信号（注明版本和风险级别）

格式为每层一个表，然后是一个 ASCII 依赖图。

**引擎意识检查**：对于列出的每个引擎 API，根据相关的模块参考文档进行验证。如果 API 是 post-cutoff 的，标记它：

```
⚠️  [ClassName.method()] — Godot 4.6 (post-cutoff, HIGH risk)
    Verified against: docs/engine-reference/godot/modules/[domain].md
    Behaviour confirmed: [yes / NEEDS VERIFICATION]
```

在写入之前获取用户对所有权映射的批准。

---

## 阶段 3：数据流

定义关键游戏场景中数据如何在模块之间移动。至少涵盖：

1. **帧更新路径**：输入 → 核心系统 → 状态 → 渲染
2. **事件/信号路径**：系统如何在不紧密耦合的情况下通信
3. **保存/加载路径**：什么状态被序列化，哪个模块拥有序列化
4. **初始化顺序**：哪些模块必须在其他模块之前启动

在有用时使用 ASCII 序列图。对于每个数据流：
- 命名正在传输的数据
- 识别生产者和消费者
- 说明这是同步调用、信号/事件还是共享状态
- 标记任何跨线程边界的数据流

在写入之前获取用户对每个场景的批准。

---

## 阶段 4：API 边界

定义模块之间的公共契约。对于每个边界：

- 模块向系统其余部分暴露的接口是什么？
- 入口点是什么（函数/信号/属性）？
- 调用者必须尊重什么不变量？
- 模块必须向调用者保证什么？

用伪代码或项目的实际语言（来自技术偏好）编写。
这些成为程序员实施所依据的契约。

**引擎意识检查**：如果任何接口使用引擎特定类型（例如 Godot 中的
`Node`、`Resource`、`Signal`），标记版本并验证类型
是否存在且未在目标引擎版本更改签名。

---

## 阶段 5：ADR 审计 + 可追溯性检查

根据阶段 1-4 中构建的架构以及阶段 0b 中的技术需求基线审查阶段 0c 中的所有现有 ADR。

### ADR 质量检查

对于每个 ADR：
- [ ] 它有 Engine Compatibility 部分吗？
- [ ] 引擎版本是否已记录？
- [ ] Post-cutoff API 是否已标记？
- [ ] 它有"GDD Requirements Addressed"部分吗？
- [ ] 它是否与本会话中做出的层/所有权决策冲突？
- [ ] 它对于固定的引擎版本仍然有效吗？

| ADR | Engine Compat | Version | GDD Linkage | Conflicts | Valid |
|-----|--------------|---------|-------------|-----------|-------|
| ADR-0001: [title] | ✅/❌ | ✅/❌ | ✅/❌ | None/[conflict] | ✅/⚠️ |

### 可追溯性覆盖检查

将技术需求基线中的每个需求映射到现有 ADR。
对于每个需求，检查任何 ADR 的"GDD Requirements Addressed"部分
或决策文本是否涵盖它：

| Req ID | Requirement | ADR Coverage | Status |
|--------|-------------|--------------|--------|
| TR-combat-001 | Hitbox detection per-frame | ADR-0003 | ✅ |
| TR-combat-002 | Combo state machine | — | ❌ GAP |

计数：X 已覆盖，Y 缺口。对于每个缺口，它成为一个**必需的新 ADR**。

### 必需的新 ADR

列出本次架构会话（阶段 1-4）中做出的所有尚
未对应 ADR 的决策，加上所有未覆盖的技术需求。
按层分组 —— Foundation 优先：

**Foundation Layer（在任何编码之前必须创建）：**
- `/architecture-decision [title]` → covers: TR-[id], TR-[id]

**Core Layer:**
- `/architecture-decision [title]` → covers: TR-[id]

---

## 阶段 6：缺失 ADR 列表

基于完整架构，生成应该存在但尚不存在的所有 ADR 的完整列表。按优先级分组：

**必须在编码开始之前拥有（Foundation 和 Core 决策）：**
- [例如"场景管理和场景加载策略"]
- [例如"事件总线 vs 直接信号架构"]

**应该在相关系统构建之前拥有：**
- [例如"库存序列化格式"]

**可以推迟到实施时：**
- [例如"水的特定着色器技术"]

---

## 阶段 7：写入主架构文档

一旦所有部分都获得批准，将完整文档写入
`docs/architecture/architecture.md`。

展示文档将包含的内容的一段摘要（层、模块、数据流、ADR 缺口）。然后使用 `AskUserQuestion`：
- "所有部分已批准。我可以写入主架构文档吗？"
  - [A] 是 —— 现在写入 `docs/architecture/architecture.md`
  - [B] 先向我展示完整的内联草稿，然后再次询问
  - [C] 还不行 —— 我有更多要讨论的更改

文档结构：

```markdown
# [Game Name] — Master Architecture

## Document Status
- Version: [N]
- Last Updated: [date]
- Engine: [name + version]
- GDDs Covered: [list]
- ADRs Referenced: [list]

## Engine Knowledge Gap Summary
[Condensed from Phase 0d inventory — HIGH/MEDIUM risk domains and their implications]

## System Layer Map
[From Phase 1]

## Module Ownership
[From Phase 2]

## Data Flow
[From Phase 3]

## API Boundaries
[From Phase 4]

## ADR Audit
[From Phase 5]

## Required ADRs
[From Phase 6]

## Architecture Principles
[3-5 key principles that govern all technical decisions for this project,
derived from the game concept, GDDs, and technical preferences]

## Open Questions
[Decisions deferred — must be resolved before the relevant layer is built]
```

---

## 阶段 7b：技术总监签字 +  Lead Programmer 可行性审查

写入主架构文档后，在交接前执行明确的签字。

**步骤 1 —— 技术总监自我审查**（此 skill 作为 technical-director 运行）：

将门 **TD-ARCHITECTURE**（`.claude/docs/director-gates.md`）作为自我审查应用。根据该门定义的所有四个标准检查完成的文档。

**审查模式检查** —— 在派生 LP-FEASIBILITY 之前应用：
- `solo` → 跳过。注意："LP-FEASIBILITY skipped — Solo mode." 继续阶段 8 交接。
- `lean` → 跳过（不是 PHASE-GATE）。注意："LP-FEASIBILITY skipped — Lean mode." 继续阶段 8 交接。
- `full` → 正常派生。

**步骤 2 —— 使用门 LP-FEASIBILITY 通过 Task 派生 `lead-programmer`（`.claude/docs/director-gates.md`）：**

传入：架构文档路径、技术需求基线摘要、ADR 列表。

**步骤 3 —— 向用户展示两个评估：**

并排展示技术总监评估和 Lead Programmer 裁决。

使用 `AskUserQuestion` —— "Technical Director 和 Lead Programmer 已审查架构。你希望如何继续？"
选项：`接受 —— 继续交接` / `先修改标记的项目` / `讨论具体关注点`

**步骤 4 —— 在架构文档中记录签字：**

更新 Document Status 部分：
```
- Technical Director Sign-Off: [date] — APPROVED / APPROVED WITH CONDITIONS
- Lead Programmer Feasibility: FEASIBLE / CONCERNS ACCEPTED / REVISED
```

内联展示提议的 Document Status 块，然后使用 `AskUserQuestion`：
- "我可以用签字结果更新 Document Status 部分吗？"
  - [A] 是 —— 应用于 `docs/architecture/architecture.md`
  - [B] 还不行 —— 我想先重新审视关注点

---

## 阶段 8：交接

**步骤 1 —— 更新会话状态**：写入摘要到 `production/session-state/active.md`，涵盖：写入的产物、TD/LP 签字裁决、任何阻塞、剩余的必需 ADR 和下一步。

**步骤 2 —— 使用确切的此模板输出交接**（无自由格式散文，无部分标题的改写）：

---

## Architecture Complete

`docs/architecture/architecture.md` v1.0 — [TD verdict: APPROVED / APPROVED WITH CONCERNS / CONCERNS]。[一句话说明架构涵盖的内容。]

---

## Run These ADRs Next

**1. `/architecture-decision "[Title]"` → ADR-[XXXX]**
[一句话：它定义了什么以及它解锁了什么。]

**2. `/architecture-decision "[Title]"` → ADR-[XXXX]**
[一句话。]

**3. `/architecture-decision "[Title]"` → ADR-[XXXX]**
[一句话。]

按优先级顺序列出阶段 6 中的前 3 个。如果剩余少于 3 个，仅列出未完成的。

---

## Gate-Check Readiness

> **`/gate-check [stage]` 之前必需：**
> - [ ] Accept ADRs: [列出必须 Accepted 的 Proposed ADR ID]
> - [ ] Write ADRs: [列出仍必须写入的 ADR ID]
> - [ ] Run `/test-setup` —— 引导 `tests/unit/`、`tests/integration/`、CI 工作流和示例测试文件
> - [ ] Run `/ux-design` —— 创建 `design/ux/interaction-patterns.md` 和 `design/accessibility-requirements.md`
>
> 勾选所有框后运行 `/gate-check [stage]`。

如果没有阻塞，改为写：
> 无阻塞 —— 现在运行 `/gate-check [stage]`。

---

## Open Questions to Watch

| ID | Summary | Priority | Resolution Path |
|----|---------|----------|-----------------|
| QQ-XX | [short description] | High / Medium / Low | [ADR or system that resolves it] |

如果没有开放的 QQ，完全省略此部分。

---

（交接结束。不要在结束规则后添加尾随评论。）

---

## 协作协议

此 skill 在每个阶段遵循协作设计原则：

1. **静默加载上下文** —— 不要叙述文件读取
2. **展示发现** —— 展示知识缺口清单和层提议
3. **决定前询问** —— 为每个架构选择展示选项
4. **批准前起草** —— 在询问写入之前内联展示内容。
   永远不要要求批准用户尚未看到的部分。
5. **使用 `AskUserQuestion` 进行写入批准** —— 纯文本"可以吗？"不够。
   使用带有标记选项 [A]/[B]/[C] 的结构化工具（立即写入 /
   先展示完整草稿 / 还不行）。对于多文件更改集，列出每个文件
   及其更改，然后分组询问一次 —— 不是每个文件单独的纯文本问题。
6. **增量写入** —— 立即写入每个批准的部分；不要
   累积所有内容并在最后写入。这可以在会话崩溃中幸存。

未经用户输入，永远不要做出具有约束力的架构决策。如果用户
不确定，在要求他们决定之前展示 2-4 个选项及其利弊。

---

## 推荐的下一步

- 为阶段 6 中列出的每个必需 ADR 运行 `/architecture-decision [title]` —— Foundation 层 ADR 优先
- 运行 `/architecture-review` —— 从刚写入的 ADR 引导需求可追溯性矩阵和 TR registry。在 Pre-Production 门之前必需。
- 运行 `/test-setup` 以引导 `tests/unit/`、`tests/integration/`、CI 工作流和示例测试（gate-check 必需）
- 运行 `/ux-design` 以初始化 `design/ux/interaction-patterns.md` 和 `design/accessibility-requirements.md`（gate-check 必需）
- 一旦写入了必需的 ADR，运行 `/create-control-manifest` 以生成层级规则 manifest
- 当所有必需 ADR、`/test-setup` 和 `/ux-design` 都完成时，运行 `/gate-check pre-production`
