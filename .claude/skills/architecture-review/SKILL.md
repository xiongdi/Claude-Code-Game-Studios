---
name: architecture-review
description: "Validates completeness and consistency of the project architecture against all GDDs. Builds a traceability matrix mapping every GDD technical requirement to ADRs, identifies coverage gaps, detects cross-ADR conflicts, verifies engine compatibility consistency across all decisions, and produces a PASS/CONCERNS/FAIL verdict. The architecture equivalent of /design-review."
argument-hint: "[focus: full | coverage | consistency | engine | single-gdd path/to/gdd.md]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Task, AskUserQuestion
agent: technical-director
model: opus
---

# Architecture Review

架构审查验证完整的架构决策体系是否覆盖所有游戏设计需求，
内部一致性如何，以及是否正确针对项目的固定引擎版本。
它是 Technical Setup 和 Pre-Production 之间的质量门。

**参数模式：**
- **无参数 / `full`**：完整审查 —— 所有阶段
- **`coverage`**：仅可追溯性 —— 哪些 GDD 需求没有 ADR
- **`consistency`**：仅跨 ADR 冲突检测
- **`engine`**：仅引擎兼容性审计
- **`single-gdd [path]`**：审查单个特定 GDD 的架构覆盖
- **`rtm`**：需求可追溯性矩阵 —— 扩展标准矩阵以包含 story 文件路径和测试文件路径；输出 `docs/architecture/requirements-traceability.md`，包含完整的 GDD 需求 → ADR → Story → Test 链。在 Production 阶段当 story 和测试存在时使用。

---

## 阶段 1：加载所有内容

### 阶段 1a —— L0：摘要扫描（快速，低 token）

在读取任何完整文档之前，使用 Grep 从所有 GDD 和 ADR 中提取 `## Summary` 部分：

```
Grep pattern="## Summary" glob="design/gdd/*.md" output_mode="content" -A 4
Grep pattern="## Summary" glob="docs/architecture/adr-*.md" output_mode="content" -A 3
```

对于 `single-gdd [path]` 模式：使用目标 GDD 的摘要来识别哪些 ADR 引用了相同的系统（按系统名称 grep ADR），然后仅完整读取那些 ADR。跳过完整读取不相关的 GDD。

对于 `engine` 模式：仅完整读取 ADR —— 引擎检查不需要 GDD。

对于 `coverage` 或 `full` 模式：继续完整读取下面的所有内容。

### 阶段 1b —— L1/L2：完整文档加载

读取适用于模式的所有输入：

### 设计文档
- `design/gdd/` 范围内的所有 GDD —— 完整读取每个文件
- `design/gdd/systems-index.md` —— 权威的系统列表

### 架构文档
- `docs/architecture/` 范围内的所有 ADR —— 完整读取每个文件
- `docs/architecture/architecture.md`（如果存在）

### 引擎参考
- `docs/engine-reference/[engine]/VERSION.md`
- `docs/engine-reference/[engine]/breaking-changes.md`
- `docs/engine-reference/[engine]/deprecated-apis.md`
- `docs/engine-reference/[engine]/modules/` 中的所有文件

### 项目标准
- `.claude/docs/technical-preferences.md`

报告计数："Loaded [N] GDDs, [M] ADRs, engine: [name + version]."

**如果存在，还要读取 `docs/consistency-failures.md`**。提取域与正在审查的系统匹配的条目（Architecture、Engine 或正在覆盖的任何 GDD 域）。在阶段 4 冲突检测输出的顶部将重复出现的模式展示为"已知冲突易发区域"说明。

---

## 阶段 2：从每个 GDD 提取技术需求

### 预加载 TR Registry

在提取任何需求之前，如果存在，读取 `docs/architecture/tr-registry.yaml`。按 `id` 和规范化后的 `requirement` 文本（小写，修剪）索引现有条目。这可以防止跨审查运行的 ID 重新编号。

对于你提取的每个需求，匹配规则是：
1. **精确/近似匹配**到同一系统的现有 registry 条目 →
   原样重用该条目的 TR-ID。仅当 GDD 措辞更改时（相同意图，更清晰的措辞）才更新 registry 中的 `requirement` 文本 —— 添加 `revised: [date]` 字段。
2. **无匹配** → 分配新 ID：该系统的下一个可用 `TR-[system]-NNN`，从最高现有序列 + 1 开始。
3. **模糊**（部分匹配，意图不清楚）→ 询问用户：
   > "'[新需求文本]' 指的是与 `TR-[system]-NNN: [现有文本]' 相同的需求，还是新需求？"
   用户回答："相同需求"（重用 ID）或"新需求"（新 ID）。

对于 registry 中任何 `status: deprecated` 的条目 —— 跳过它。
它是从 GDD 中故意删除的。

对于每个 GDD，读取它并提取所有**技术需求** —— 架构必须为系统工作提供的任何东西。技术需求是暗示特定架构决策的任何陈述。

要提取的类别：

| 类别 | 示例 |
|----------|---------|
| **数据结构** | "每个实体有生命值、最大生命值、状态效果" → 需要组件/数据模式 |
| **性能约束** | "碰撞检测必须在 200 个实体下以 60fps 运行" → 物理预算 ADR |
| **引擎能力** | "角色动画的逆向运动学" → IK 系统 ADR |
| **跨系统通信** | "伤害系统同时通知 UI 和音频" → 事件/信号架构 ADR |
| **状态持久化** | "玩家进度在会话之间保持" → 存档系统 ADR |
| **线程/计时** | "AI 决策在主线程外发生" → 并发 ADR |
| **平台需求** | "支持键盘、手柄、触摸" → 输入系统 ADR |

对于每个 GDD，生成一个结构化列表：

```
GDD: [filename]
System: [system name]
Technical Requirements:
  TR-[GDD]-001: [requirement text] → Domain: [Physics/Rendering/etc]
  TR-[GDD]-002: [requirement text] → Domain: [...]
```

这成为**需求基线** —— 架构必须覆盖的完整集合。

---

## 阶段 3：构建可追溯性矩阵

对于阶段 2 中提取的每个技术需求，搜索 ADR：

1. 读取每个 ADR 的"GDD Requirements Addressed"部分
2. 检查它是否明确引用该需求或其 GDD
3. 检查 ADR 的决策文本是否隐式覆盖该需求
4. 标记覆盖状态：

| 状态 | 含义 |
|--------|---------|
| ✅ **Covered** | ADR 明确解决此需求 |
| ⚠️ **Partial** | ADR 部分覆盖此，或覆盖模糊 |
| ❌ **Gap** | 没有 ADR 解决此需求 |

构建完整矩阵：

```
## Traceability Matrix

| Requirement ID | GDD | System | Requirement | ADR Coverage | Status |
|---------------|-----|--------|-------------|--------------|--------|
| TR-combat-001 | combat.md | Combat | Hitbox detection < 1 frame | ADR-0003 | ✅ |
| TR-combat-002 | combat.md | Combat | Combo window timing | — | ❌ GAP |
| TR-inventory-001 | inventory.md | Inventory | Persistent item storage | ADR-0005 | ✅ |
```

统计总数：X 已覆盖，Y 部分，Z 缺口。

---

## 阶段 3b：Story 和测试链接（仅 RTM 模式）

*除非参数是 `rtm` 或存在 story 的 `full`，否则跳过此阶段。*

此阶段扩展阶段 3 的矩阵以包含实现每个需求的 story 和验证它的测试 —— 生成完整的需求可追溯性矩阵（RTM）。

### 步骤 3b-1 —— 加载 story

Glob `production/epics/**/*.md`（不包括 EPIC.md 索引文件）。对于每个 story 文件：
- 从 story 的 Context 部分提取 `TR-ID`
- 提取 story 文件路径、标题、Status
- 提取 `## Test Evidence` 部分 —— 声明的测试文件路径

### 步骤 3b-2 —— 加载测试文件

Glob `tests/unit/**/*_test.*` 和 `tests/integration/**/*_test.*`。
构建索引：system → [test file paths]。

对于步骤 3b-1 中的每个测试文件路径，通过 Glob 确认文件是否实际存在。如果声明的路径不存在，标记为 MISSING。

### 步骤 3b-3 —— 构建扩展 RTM

对于阶段 3 矩阵中的每个 TR-ID，添加：
- **Story**：引用此 TR-ID 的 story 文件路径（可能有多个）
- **Test File**：story 的 Test Evidence 部分中声明的测试文件路径
- **Test Status**：COVERED（测试文件存在）/ MISSING（路径声明但未找到）/ NONE（未声明测试路径，story 类型可能是 Visual/Feel/UI）/ NO STORY（需求还没有 story —— 预制作缺口）

扩展矩阵格式：

```
## Requirements Traceability Matrix (RTM)

| TR-ID | GDD | Requirement | ADR | Story | Test File | Test Status |
|-------|-----|-------------|-----|-------|-----------|-------------|
| TR-combat-001 | combat.md | Hitbox < 1 frame | ADR-0003 | story-001-hitbox.md | tests/unit/combat/hitbox_test.gd | COVERED |
| TR-combat-002 | combat.md | Combo window | — | story-002-combo.md | — | NONE (Visual/Feel) |
| TR-inventory-001 | inventory.md | Persistent storage | ADR-0005 | — | — | NO STORY |
```

RTM 覆盖摘要：
- COVERED: [N] —— 有 ADR + story + 通过测试的需求
- MISSING test: [N] —— story 存在但测试文件未找到
- NO STORY: [N] —— 有 ADR 但还没有 story 的需求
- NO ADR: [N] —— 没有架构覆盖的需求（来自阶段 3 缺口）
- 完整链完成（COVERED）：[N/total]（[%]）

---

## 阶段 4：跨 ADR 冲突检测

将每个 ADR 与每个其他 ADR 进行比较以检测矛盾。当以下情况时存在冲突：

- **数据所有权冲突**：两个 ADR 声称对同一数据拥有独占所有权
- **集成契约冲突**：ADR-A 假设系统 X 有接口 Y，但
  ADR-B 用不同的接口定义系统 X
- **性能预算冲突**：ADR-A 为物理分配 N 毫秒，ADR-B 为 AI 分配
  N 毫秒，它们一起超过了总帧预算
- **依赖循环**：ADR-A 说系统 X 在 Y 之前初始化；ADR-B 说 Y
  在 X 之前初始化
- **架构模式冲突**：ADR-A 对子系统使用事件驱动通信；
  ADR-B 对同一子系统使用直接函数调用
- **状态管理冲突**：两个 ADR 定义对同一游戏状态的权威
  （例如 Combat ADR 和 Character ADR 都声称拥有生命值）

对于发现的每个冲突：

```
## Conflict: [ADR-NNNN] vs [ADR-MMMM]
Type: [Data ownership / Integration / Performance / Dependency / Pattern / State]
ADR-NNNN claims: [...]
ADR-MMMM claims: [...]
Impact: [What breaks if both are implemented as written]
Resolution options:
  1. [Option A]
  2. [Option B]
```

### ADR 依赖排序

冲突检测后，分析所有 ADR 的依赖图：

1. **收集所有 `Depends On` 字段**来自每个 ADR 的"ADR Dependencies"部分
2. **拓扑排序**：确定正确的实现顺序 —— 没有依赖的 ADR 首先（Foundation），依赖于这些的 ADR 其次，等等。
3. **标记未解决的依赖**：如果 ADR-A 的"Depends On"字段引用一个仍然是 `Proposed` 或不存在的 ADR，标记它：
   ```
   ⚠️  ADR-0005 depends on ADR-0002 — but ADR-0002 is still Proposed.
       ADR-0005 cannot be safely implemented until ADR-0002 is Accepted.
   ```
4. **循环检测**：如果 ADR-A 依赖于 ADR-B 且 ADR-B 依赖于 ADR-A（直接
   或传递），将其标记为 `DEPENDENCY CYCLE`：
   ```
   🔴 DEPENDENCY CYCLE: ADR-0003 → ADR-0006 → ADR-0003
      This cycle must be broken before either can be implemented.
   ```
5. **输出推荐的实现顺序**：
   ```
   ### Recommended ADR Implementation Order (topologically sorted)
   Foundation (no dependencies):
     1. ADR-0001: [title]
     2. ADR-0003: [title]
   Depends on Foundation:
     3. ADR-0002: [title] (requires ADR-0001)
     4. ADR-0005: [title] (requires ADR-0003)
   Feature layer:
     5. ADR-0004: [title] (requires ADR-0002, ADR-0005)
   ```

---

## 阶段 5：引擎兼容性交叉检查

跨所有 ADR 检查引擎一致性：

### 版本一致性
- 所有提及引擎版本的 ADR 是否同意同一版本？
- 如果任何 ADR 是为较旧的引擎版本编写的，将其标记为可能过时

### Post-Cutoff API 一致性
- 从所有 ADR 收集所有"Post-Cutoff APIs Used"字段
- 对每个字段，根据相关的模块参考文档进行验证
- 检查没有两个 ADR 对同一 post-cutoff API 做出矛盾假设

### 弃用 API 检查
- 对所有 ADR 中 `deprecated-apis.md` 中列出的 API 名称进行 Grep
- 标记任何引用已弃用 API 的 ADR

### 缺少引擎兼容性部分
- 列出完全缺少 Engine Compatibility 部分的所有 ADR
- 这些是盲点 —— 它们的引擎假设未知

输出格式：
```
### Engine Audit Results
Engine: [name + version]
ADRs with Engine Compatibility section: X / Y total

Deprecated API References:
  - ADR-0002: uses [deprecated API] — deprecated since [version]

Stale Version References:
  - ADR-0001: written for [older version] — current project version is [version]

Post-Cutoff API Conflicts:
  - ADR-0004 and ADR-0007 both use [API] with incompatible assumptions
```

---

### 引擎专家咨询

完成上述引擎审计后，通过 Task 派生**主要引擎专家**以获取域专家的第二意见：
- 读取 `.claude/docs/technical-preferences.md` 的 `Engine Specialists` 部分以获取主要专家
- 如果未配置引擎，跳过此咨询
- 派生 `subagent_type: [primary specialist]`，传入：所有包含引擎特定决策或 `Post-Cutoff APIs Used` 字段的 ADR、引擎参考文档和阶段 5 审计发现。要求他们：
  1. 确认或挑战每个审计发现 —— 专家可能知道参考文档中未捕获的引擎细微差别
  2. 识别审计可能遗漏的 ADR 中的引擎特定反模式（例如，使用错误的 Godot 节点类型、Unity 组件耦合、Unreal 子系统误用）
  3. 标记对引擎行为做出与实际固定版本不同假设的 ADR

将额外的发现纳入阶段 5 输出中的 `### Engine Specialist Findings`。这些输入最终裁决 —— 专家识别的问题与审计识别的问题具有相同的权重。

---

## 阶段 5b：设计修订标记（架构 → GDD 反馈）

对于阶段 5 中的每个**高风险引擎发现**，检查是否有任何 GDD 做出了与验证的引擎现实相矛盾的假设。

要检查的具体情况：

1. **Post-cutoff API 行为与训练数据假设不同**：如果 ADR 记录了与默认 LLM 假设不同的已验证 API 行为，检查引用相关系统的所有 GDD。查找围绕旧（假设的）行为编写的设计规则。

2. **ADR 中的已知引擎限制**：如果 ADR 记录了已知的引擎限制（例如"Jolt 忽略 HingeJoint3D 阻尼"、"D3D12 现在是默认后端"），检查围绕受影响功能设计机制的 GDD。

3. **弃用 API 冲突**：如果阶段 5 标记了 ADR 中使用的已弃用 API，检查是否有任何 GDD 包含假设已弃用 API 行为的机制。

对于发现的每个冲突，将其记录在 GDD 修订标记表中：

```
### GDD Revision Flags (Architecture → Design Feedback)
These GDD assumptions conflict with verified engine behaviour or accepted ADRs.
The GDD should be revised before its system enters implementation.

| GDD | Assumption | Reality (from ADR/engine-reference) | Action |
|-----|-----------|--------------------------------------|--------|
| combat.md | "Use HingeJoint3D damp for weapon recoil" | Jolt ignores damp — ADR-0003 | Revise GDD |
```

如果没有找到修订标记，写："No GDD revision flags —— all GDD assumptions are consistent with verified engine behaviour."

在询问之前，内联展示提议的更改 —— 并排显示每个标记 GDD 的当前 systems-index 行和提议的更新行，以便用户可以看到确切会更改什么。

然后使用 `AskUserQuestion`：
- "我发现了 [N] 个 GDD 修订标记。我可以更新系统索引吗？"
  - [A] 是 —— 现在将所有 [N] 个更新应用到系统索引
  - [B] 先向我展示完整的差异，然后再次询问
  - [C] 不 —— 暂时保持系统索引不变

如果 [A]：应用更新。Status 字段必须恰好是 `Needs Revision` —— 没有括号
（其他 skill 匹配那个精确字符串，括号会破坏匹配）。
如果 [B]：显示完整的提议 systems-index 部分，然后使用 `AskUserQuestion` 重新询问。

---

## 阶段 6：架构文档覆盖

如果 `docs/architecture/architecture.md` 存在，根据 GDD 验证它：

- `systems-index.md` 中的每个系统是否都出现在架构层中？
- 数据流部分是否覆盖 GDD 中定义的所有跨系统通信？
- API 边界是否支持 GDD 中的所有集成需求？
- 架构文档中是否有没有对应 GDD 的系统
  （孤立的架构）？

---

## 阶段 7：输出审查报告

```
## Architecture Review Report
Date: [date]
Engine: [name + version]
GDDs Reviewed: [N]
ADRs Reviewed: [M]

---

### Traceability Summary
Total requirements: [N]
✅ Covered: [X]
⚠️ Partial: [Y]
❌ Gaps: [Z]

### Coverage Gaps (no ADR exists)
For each gap:
  ❌ TR-[id]: [GDD] → [system] → [requirement]
     Suggested ADR: "/architecture-decision [suggested title]"
     Domain: [Physics/Rendering/etc]
     Engine Risk: [LOW/MEDIUM/HIGH]

### Cross-ADR Conflicts
[List all conflicts from Phase 4]

### ADR Dependency Order
[Topologically sorted implementation order from Phase 4 — dependency ordering section]
[Unresolved dependencies and cycles if any]

### GDD Revision Flags
[GDD assumptions that conflict with verified engine behaviour — from Phase 5b]
[Or: "None — all GDD assumptions consistent with verified engine behaviour"]

### Engine Compatibility Issues
[List all engine issues from Phase 5]

### Architecture Document Coverage
[List missing systems and orphaned architecture from Phase 6]

---

### Verdict: [PASS / CONCERNS / FAIL]

PASS: All requirements covered, no conflicts, engine consistent
CONCERNS: Some gaps or partial coverage, but no blocking conflicts
FAIL: Critical gaps (Foundation/Core layer requirements uncovered),
      or blocking cross-ADR conflicts detected

### Blocking Issues (must resolve before PASS)
[List items that must be resolved — FAIL verdict only]

### Required ADRs
[Prioritised list of ADRs to create, most foundational first]
```

---

## 阶段 8：写入和更新可追溯性索引

使用 `AskUserQuestion` 进行写入批准：
- "审查完成。你希望写入什么？"
  - [A] 写入所有三个文件（审查报告 + 可追溯性索引 + TR registry）
  - [B] 仅写入审查报告 —— `docs/architecture/architecture-review-[date].md`
  - [C] 暂不写入任何内容 —— 我需要先审查发现

### RTM 输出（仅 rtm 模式）

对于 `rtm` 模式，使用 `AskUserQuestion`：
- "我可以写入完整的需求可追溯性矩阵吗？"
  - [A] 是 —— 写入 `docs/architecture/requirements-traceability.md`
  - [B] 还不行 —— 先向我展示完整的 RTM 数据，然后再次询问

RTM 文件格式：

```markdown
# Requirements Traceability Matrix (RTM)

> Last Updated: [date]
> Mode: /architecture-review rtm
> Coverage: [N]% full chain complete (GDD → ADR → Story → Test)

## How to read this matrix

| Column | Meaning |
|--------|---------|
| TR-ID | Stable requirement ID from tr-registry.yaml |
| GDD | Source design document |
| ADR | Architectural decision governing implementation |
| Story | Story file that implements this requirement |
| Test File | Automated test file path |
| Test Status | COVERED / MISSING / NONE / NO STORY |

## Full Traceability Matrix

| TR-ID | GDD | Requirement | ADR | Story | Test File | Status |
|-------|-----|-------------|-----|-------|-----------|--------|
[Full matrix rows from Phase 3b]

## Coverage Summary

| Status | Count | % |
|--------|-------|---|
| COVERED — full chain complete | [N] | [%] |
| MISSING test — story exists, no test | [N] | [%] |
| NO STORY — ADR exists, not yet implemented | [N] | [%] |
| NO ADR — architectural gap | [N] | [%] |
| **Total requirements** | **[N]** | **100%** |

## Uncovered Requirements (Priority Fix List)

Requirements where the full chain is broken, prioritised by layer:

### Foundation layer gaps
[list with suggested action per gap]

### Core layer gaps
[list]

### Feature / Presentation layer gaps
[list — lower priority]

## History

| Date | Full Chain % | Notes |
|------|-------------|-------|
| [date] | [%] | Initial RTM |
```

### TR Registry 更新

还要询问："我可以用此审查中的新需求 ID 更新 `docs/architecture/tr-registry.yaml` 吗？"

如果同意：
- **追加**此审查之前 registry 中没有的任何新 TR-ID
- **更新**其 GDD 措辞已更改的任何条目的 `requirement` 文本和 `revised` 日期（ID 保持不变）
- **标记**其 GDD 需求不再存在的任何 registry 条目的 `status: deprecated`（在标记为弃用前与用户确认）
- **永远不要**重新编号或删除现有条目
- 更新顶部的 `last_updated` 和 `version` 字段

这确保所有未来的 story 文件都可以引用稳定的 TR-ID，这些 ID 在每次后续架构审查中持续存在。

### Reflexion 日志更新

写入审查报告后，将阶段 4 中发现的任何 🔴 CONFLICT 条目追加到 `docs/consistency-failures.md`（如果文件存在）：

```markdown
### [YYYY-MM-DD] — /architecture-review — 🔴 CONFLICT
**Domain**: Architecture / [specific domain e.g. State Ownership, Performance]
**Documents involved**: [ADR-NNNN] vs [ADR-MMMM]
**What happened**: [specific conflict — what each ADR claims]
**Resolution**: [how it was or should be resolved]
**Pattern**: [generalised lesson for future ADR authors in this domain]
```

仅追加 CONFLICT 条目 —— 不记录 GAP 条目（在架构完成之前缺少 ADR 是预期的）。如果文件不存在，不要创建 —— 仅在已存在时追加。

### 会话状态更新

写入所有批准的文件后，静默追加到 `production/session-state/active.md`：

    ## Session Extract — /architecture-review [date]
    - Verdict: [PASS / CONCERNS / FAIL]
    - Requirements: [N] total — [X] covered, [Y] partial, [Z] gaps
    - New TR-IDs registered: [N, or "None"]
    - GDD revision flags: [comma-separated GDD names, or "None"]
    - Top ADR gaps: [top 3 gap titles from the report, or "None"]
    - Report: docs/architecture/architecture-review-[date].md

如果 `active.md` 不存在，创建它并将此块作为初始内容。
在对话中确认："Session state updated."

可追溯性索引格式：

```markdown
# Architecture Traceability Index
Last Updated: [date]
Engine: [name + version]

## Coverage Summary
- Total requirements: [N]
- Covered: [X] ([%])
- Partial: [Y]
- Gaps: [Z]

## Full Matrix
[Complete traceability matrix from Phase 3]

## Known Gaps
[All ❌ items with suggested ADRs]

## Superseded Requirements
[Requirements whose GDD was changed after the ADR was written]
```

---

## 阶段 9：交接

完成审查和写入批准的文件后，展示：

1. **立即行动**：列出要创建的前 3 个 ADR（影响最大的缺口优先，
   Foundation 层在 Feature 层之前）
2. **门前清单**：通过 Glob 检查这些是否存在，并标记每个 ✅ 或 ❌：
   - `tests/unit/` 和 `tests/integration/` 目录 —— 如果 ❌：运行 `/test-setup`
   - `.github/workflows/tests.yml` —— 如果 ❌：运行 `/test-setup`
   - `design/accessibility-requirements.md` —— 如果 ❌：运行 `/ux-design`
   - `design/ux/interaction-patterns.md` —— 如果 ❌：运行 `/ux-design`
   将 ❌ 项目展示为 gate-check 之前所需的步骤。如果任何项目是 ❌，不要提供 `/gate-check`
   作为选项 —— 提供要运行的缺失 skill。
3. **重新运行触发器**："每个新 ADR 写入后重新运行 `/architecture-review`，
   验证覆盖范围改善"

然后使用根据门前清单状态定制的 `AskUserQuestion` 结束：
- 如果 ADR 缺口仍然存在或任何门前项目是 ❌：
  - "架构审查完成。你希望接下来做什么？"
    - [A] 写入缺失的 ADR —— 打开新会话并运行 `/architecture-decision [system]`
    - [B] 运行 `/test-setup` —— gate-check 之前需要（仅在测试基础设施为 ❌ 时显示）
    - [C] 运行 `/ux-design` —— gate-check 之前需要（仅在 UX/无障碍文件为 ❌ 时显示）
    - [D] 在此会话停止
- 如果所有门前清单项目都是 ✅ 且没有阻塞的 ADR 缺口：
  - "架构审查完成。所有门前项目已确认。你希望接下来做什么？"
    - [A] 运行 `/gate-check pre-production`
    - [B] 写入缺失的 ADR —— 打开新会话并运行 `/architecture-decision [system]`
    - [C] 在此会话停止

---

## 错误恢复协议

如果任何派生的 agent 返回 BLOCKED、错误或未能完成：

1. **立即暴露**：在继续之前报告 "[AgentName]: BLOCKED —— [reason]"
2. **评估依赖**：如果被阻塞的 agent 的输出是后续阶段所需的，在没有用户输入的情况下不要超越该阶段
3. **通过 AskUserQuestion 提供选项**，三个选择：
   - 跳过此 agent 并在最终报告中注明缺口
   - 用更窄的范围重试（更少的 GDD，单系统焦点）
   - 在此停止并首先解决阻塞
4. **始终生成部分报告** —— 输出任何已完成的内容，以便工作不会丢失

---

## 协作协议

1. **静默读取** —— 不要叙述每个文件读取
2. **展示矩阵** —— 在询问任何内容之前展示完整的可追溯性矩阵；让用户看到状态
3. **不要猜测** —— 如果需求模糊，询问："[X] 是技术需求还是设计偏好？"
4. **批准前起草** —— 始终在对话中内联展示将要写入的内容（报告、更新的 ADR 部分、systems-index 行），然后再请求批准。永远不要要求批准用户尚未看到的内容。
5. **使用 `AskUserQuestion` 进行写入批准** —— 纯文本"可以吗？"不够。使用带有标记选项 [A]/[B]/[C] 的结构化工具，以便用户可以在"立即写入"、"先展示完整草稿"和"还不行"之间选择。多文件更改集必须列出每个文件及其更改，然后分组询问一次 —— 而不是每个文件单独的纯文本问题。
6. **非阻塞** —— 裁决是建议性的；用户决定是否继续，尽管有 CONCERNS 甚至 FAIL 发现
