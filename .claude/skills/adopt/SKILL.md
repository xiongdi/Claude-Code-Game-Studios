---
name: adopt
description: "Brownfield onboarding — audits existing project artifacts for template format compliance (not just existence), classifies gaps by impact, and produces a numbered migration plan. Run this when joining an in-progress project or upgrading from an older template version. Distinct from /project-stage-detect (which checks what exists) — this checks whether what exists will actually work with the template's skills."
argument-hint: "[focus: full | gdds | adrs | stories | infra]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion
model: sonnet
agent: technical-director
---

# Adopt — Brownfield 模板采用

此 skill 审计现有项目的产物是否符合模板 skill 管道的**格式合规性**，然后生成一个按优先级排序的迁移计划。

**这不是 `/project-stage-detect`。**
`/project-stage-detect` 回答：*存在什么？*
`/adopt` 回答：*存在的内容能否真正与模板的 skill 一起工作？*

一个项目可能有 GDD、ADR 和 story —— 但如果这些产物的内部格式不正确，每个格式敏感的 skill 仍会静默失败或产生错误结果。

**输出：** `docs/adoption-plan-[date].md` —— 一个持久、可检查的迁移计划。

**参数模式：**

**审计模式：** `$ARGUMENTS[0]`（空白 = `full`）

- **无参数 / `full`**：完整审计 —— 所有产物类型
- **`gdds`**：仅 GDD 格式合规性
- **`adrs`**：仅 ADR 格式合规性
- **`stories`**：仅 story 格式合规性
- **`infra`**：仅基础设施产物缺口（registry、manifest、sprint-status、stage.txt）

---

## 阶段 1：检测项目状态

在读取之前输出一行：`"Scanning project artifacts..."` —— 这确认 skill 在静默读取阶段正在运行。

然后在展示任何其他内容之前静默读取。

### 存在性检查
- `production/stage.txt` —— 如果存在，读取它（权威阶段）
- `design/gdd/game-concept.md` —— 概念是否存在？
- `design/gdd/systems-index.md` —— 系统索引是否存在？
- 统计 GDD 文件：`design/gdd/*.md`（不包括 game-concept.md 和 systems-index.md）
- 统计 ADR 文件：`docs/architecture/adr-*.md`
- 统计 story 文件：`production/epics/**/*.md`（不包括 EPIC.md）
- `.claude/docs/technical-preferences.md` —— 引擎是否已配置？
- `docs/engine-reference/` —— 引擎参考文档是否存在？
- Glob `docs/adoption-plan-*.md` —— 如果存在先前的计划，记录其文件名

### 推断阶段（如果没有 stage.txt）
使用与 `/project-stage-detect` 相同的启发式方法：
- `src/` 中有 10+ 个源文件 → Production
- `production/epics/` 中有 story → Pre-Production
- ADR 存在 → Technical Setup
- systems-index.md 存在 → Systems Design
- game-concept.md 存在 → Concept
- 什么都没有 → Fresh（不是 brownfield 项目 —— 建议 `/start`）

如果项目看起来是全新的（完全没有产物），使用 `AskUserQuestion`：
- "这看起来是一个全新项目 —— 未找到现有产物。`/adopt` 适用于有工作要迁移的项目。你想做什么？"
  - "运行 `/start` —— 开始引导式首次 onboarding"
  - "我的产物在非标准位置 —— 帮我找到它们"
  - "取消"

然后停止 —— 无论用户选择哪个选项，都不要继续审计
（每个选项导向不同的 skill 或手动调查）。

报告："Detected phase: [phase]. Found: [N] GDDs, [M] ADRs, [P] stories."

---

## 阶段 2：格式审计

对于范围内的每个产物类型（基于参数模式），不仅检查文件是否存在，还检查它是否包含模板所需的内部结构。

### 2a：GDD 格式审计

对于找到的每个 GDD 文件，通过扫描标题检查 8 个必需章节：

| 必需章节 | 要查找的标题模式 |
|---|---|
| Overview | `## Overview` |
| Player Fantasy | `## Player Fantasy` |
| Detailed Rules / Design | `## Detailed` 或 `## Core Rules` 或 `## Detailed Design` |
| Formulas | `## Formulas` 或 `## Formula` |
| Edge Cases | `## Edge Cases` |
| Dependencies | `## Dependencies` 或 `## Depends` |
| Tuning Knobs | `## Tuning` |
| Acceptance Criteria | `## Acceptance` |

对于每个 GDD，记录：
- 哪些章节存在
- 哪些章节缺失
- 存在的章节是否有实际内容或只是占位文本
  （`[To be designed]` 或类似内容）

还要检查：每个 GDD 的头部块中是否有 `**Status**:` 字段？
有效值：`In Design`、`Designed`、`In Review`、`Approved`、`Needs Revision`。

### 2b：ADR 格式审计

对于找到的每个 ADR 文件，检查这些关键章节：

| 章节 | 缺失时的影响 |
|---|---|
| `## Status` | **BLOCKING** —— `/story-readiness` ADR 状态检查会静默通过一切 |
| `## ADR Dependencies` | HIGH —— `/architecture-review` 中的依赖排序会中断 |
| `## Engine Compatibility` | HIGH —— post-cutoff API 风险未知 |
| `## GDD Requirements Addressed` | MEDIUM —— 可追溯性矩阵失去覆盖 |
| `## Performance Implications` | LOW —— 非管道关键 |

对于每个 ADR，记录：哪些章节存在，哪些缺失，如果 Status 章节存在，记录当前 Status 值。

### 2c：systems-index.md 格式审计

如果 `design/gdd/systems-index.md` 存在：

1. **括号状态值** —— Grep 查找任何包含括号的 Status 单元格：
   `"Needs Revision ("`、`"In Progress ("` 等。
   这些会破坏 `/gate-check`、`/create-stories` 和 `/architecture-review` 中的精确字符串匹配。**BLOCKING。**

2. **有效状态值** —— 检查 Status 列的值仅来自：
   `Not Started`、`In Progress`、`In Review`、`Designed`、`Approved`、`Needs Revision`
   标记任何无法识别的值。

3. **列结构** —— 检查表格至少包含：System name、
   Layer、Priority、Status 列。缺失列会降低 skill 功能。

### 2d：Story 格式审计

对于找到的每个 story 文件：

- **`Manifest Version:` 字段** —— 是否存在于 story 头部？（LOW —— 如果缺失则自动通过）
- **TR-ID 引用** —— story 是否包含 `TR-[a-z]+-[0-9]+` 模式？（MEDIUM —— 无过时跟踪）
- **ADR 引用** —— story 是否引用至少一个 ADR？（检查 `ADR-` 模式）
- **Status 字段** —— 是否存在且可读？
- **验收标准** —— story 是否有复选框列表（`- [ ]`）？

### 2e：基础设施审计

| 产物 | 路径 | 缺失时的影响 |
|---|---|---|
| TR registry | `docs/architecture/tr-registry.yaml` | HIGH —— 无稳定需求 ID |
| Control manifest | `docs/architecture/control-manifest.md` | HIGH —— 无 story 的层级规则 |
| Manifest version stamp | 在 manifest 头部：`Manifest Version:` | MEDIUM —— 过时检查盲目 |
| Sprint status | `production/sprint-status.yaml` | MEDIUM —— `/sprint-status` 回退到 markdown |
| Stage file | `production/stage.txt` | MEDIUM —— 阶段自动检测不可靠 |
| Engine reference | `docs/engine-reference/[engine]/VERSION.md` | HIGH —— ADR 引擎检查盲目 |
| Architecture traceability | `docs/architecture/architecture-traceability.md` | MEDIUM —— 无持久矩阵 |

### 2f：技术偏好审计

读取 `.claude/docs/technical-preferences.md`。检查每个字段是否为 `[TO BE CONFIGURED]`：
- Engine、Language、Rendering、Physics → 如果未配置则为 HIGH（ADR skill 会失败）
- 命名约定 → MEDIUM
- 性能预算 → MEDIUM
- Forbidden Patterns、Allowed Libraries → LOW（设计上空着开始）

---

## 阶段 3：分类和优先排序缺口

将所有审计中发现的所有缺口组织到四个严重性层级中：

**BLOCKING** —— 会导致模板 skill 现在静默产生错误结果。
示例：ADR 缺少 Status 字段，systems-index 括号状态值，
ADR 存在时引擎未配置。

**HIGH** —— 会导致 story 生成时缺少安全检查，或
基础设施引导会失败。
示例：ADR 缺少 Engine Compatibility，GDD 缺少 Acceptance Criteria
（无法从中生成 story），tr-registry.yaml 缺失。

**MEDIUM** —— 降低质量和管道跟踪但不破坏功能。
示例：GDD 缺少 Tuning Knobs 或 Formulas 章节，story 缺少 TR-ID，
sprint-status.yaml 缺失。

**LOW** —— 追溯性改进，有更好但不紧急。
示例：story 缺少 Manifest Version 戳，GDD 缺少 Open Questions 章节。

统计每个层级的总数。如果零个 BLOCKING 和零个 HIGH 缺口：报告项目
与模板兼容，仅剩下建议性改进。

---

## 阶段 4：构建迁移计划

编写一个编号的、有序的操作计划。排序规则：
1. BLOCKING 缺口优先（在任何管道 skill 可靠运行之前必须修复）
2. 接下来是 HIGH 缺口，基础设施在 GDD/ADR 内容之前（引导需要正确格式）
3. MEDIUM 缺口排序：GDD 缺口在 ADR 缺口之前，ADR 缺口在 story 缺口之前（story 依赖于 GDD 和 ADR）
4. LOW 缺口最后

对于每个缺口，生成一个计划条目，包含：
- 清晰的问题陈述（一句话，无行话）
- 如果 skill 处理，给出确切的修复命令
- 如果需要直接编辑，给出手动步骤
- 时间估算（粗略：5 分钟 / 30 分钟 / 1 个会话）
- 用于跟踪的复选框 `- [ ]`

**特殊情况 —— systems-index 括号状态值：**
如果存在，这始终是第一个项目。显示需要更改的确切值和确切的替换文本。提供在写入计划之前立即修复此问题。

**特殊情况 —— ADR 缺少 Status 字段：**
对于每个受影响的 ADR，修复方法是：
`/architecture-decision retrofit docs/architecture/adr-[NNNN]-[slug].md`
将每个 ADR 列为单独的可检查项目。

**特殊情况 —— GDD 缺少章节：**
对于每个受影响的 GDD，列出缺少哪些章节以及修复方法：
`/design-system retrofit design/gdd/[filename].md`

**基础设施引导顺序** —— 始终按此序列呈现：
1. 首先修复 ADR 格式（registry 依赖于读取 ADR Status 字段）
2. 运行 `/architecture-review` → 引导 `tr-registry.yaml`
3. 运行 `/create-control-manifest` → 创建带版本戳的 manifest
4. 运行 `/sprint-plan update` → 创建 `sprint-status.yaml`
5. 运行 `/gate-check [phase]` → 权威地写入 `stage.txt`

**现有 story** —— 明确说明：
> "现有 story 继续使用所有模板 skill —— 当字段缺失时，所有新格式检查自动通过。在重新生成之前，它们不会从 TR-ID 过时跟踪或 manifest 版本检查中受益。这是故意的：不要重新生成已经在进行中的 story。"

---

## 阶段 5：展示摘要并询问是否写入

在写入之前展示一个紧凑的摘要：

```
## Adoption Audit Summary
Phase detected: [phase]
Engine: [configured / NOT CONFIGURED]
GDDs audited: [N] ([X] fully compliant, [Y] with gaps)
ADRs audited: [N] ([X] fully compliant, [Y] with gaps)
Stories audited: [N]

Gap counts:
  BLOCKING: [N] — template skills will malfunction without these fixes
  HIGH:     [N] — unsafe to run /create-stories or /story-readiness
  MEDIUM:   [N] — quality degradation
  LOW:      [N] — optional improvements

Estimated remediation: [X blocking items × ~Y min each = roughly Z hours]
```

在询问写入之前，展示一个**缺口预览**：
- 将每个 BLOCKING 缺口列为一行项目符号，描述实际问题
  （例如 `systems-index.md: 3 rows have parenthetical status values`，
  `adr-0002.md: missing ## Status section`）。不显示计数 —— 显示实际项目。
- 将 HIGH / MEDIUM / LOW 仅显示为计数（例如 `HIGH: 4, MEDIUM: 2, LOW: 1`）。

这给用户足够的上下文来判断范围，然后再承诺写入文件。

如果在阶段 1 检测到先前的采用计划，添加说明：
> "先前的计划存在于 `docs/adoption-plan-[prior-date].md`。新计划将反映当前项目状态 —— 它不会与先前的运行进行差异比较。"

使用 `AskUserQuestion`：
- "准备好写入迁移计划了吗？"
  - "是 —— 写入 `docs/adoption-plan-[date].md`"
  - "先向我展示完整计划预览（暂不写入）"
  - "取消 —— 我会手动处理迁移"

如果用户选择"向我展示完整计划预览"，将完整计划输出为围栏 markdown 块。然后使用相同的三个选项再次询问。

---

## 阶段 6：写入采用计划

如果获得批准，使用以下结构写入 `docs/adoption-plan-[date].md`：

```markdown
# Adoption Plan

> **Generated**: [date]
> **Project phase**: [phase]
> **Engine**: [name + version, or "Not configured"]
> **Template version**: v1.0+

按顺序完成这些步骤。每完成一项就勾选。
随时重新运行 `/adopt` 检查剩余缺口。

---

## Step 1: Fix Blocking Gaps

[每个 blocking 缺口一个子章节，包含问题、修复命令、时间估算、复选框]

---

## Step 2: Fix High-Priority Gaps

[每个 high 缺口一个子章节]

---

## Step 3: Bootstrap Infrastructure

### 3a. Register existing requirements (creates tr-registry.yaml)
运行 `/architecture-review` —— 即使 ADR 已存在，此运行也会从现有 GDD 和 ADR 引导 TR registry。
**时间**: 1 个会话（对于大型代码库审查可能较长）
- [ ] tr-registry.yaml created

### 3b. Create control manifest
运行 `/create-control-manifest`
**时间**: 30 min
- [ ] docs/architecture/control-manifest.md created

### 3c. Create sprint tracking file
运行 `/sprint-plan update`
**时间**: 5 min（如果 sprint plan 已作为 markdown 存在）
- [ ] production/sprint-status.yaml created

### 3d. Set authoritative project stage
运行 `/gate-check [current-phase]`
**时间**: 5 min
- [ ] production/stage.txt written

---

## Step 4: Medium-Priority Gaps

[每个 medium 缺口一个子章节]

---

## Step 5: Optional Improvements

[每个 low 缺口一个子章节]

---

## What to Expect from Existing Stories

现有 story 继续使用所有模板 skill。当字段缺失时，新格式检查（TR-ID 验证、manifest 版本过时）会自动通过 —— 所以不会破坏任何东西。在重新生成之前，它们不会从过时跟踪中受益。不要重新生成正在进行中或已完成的 story。

---

## Re-run

完成 Step 3 后再次运行 `/adopt`，验证所有 blocking 和 high 缺口已解决。新运行将反映项目的当前状态。
```

---

## 阶段 6b：设置审查模式

写入采用计划后（或如果用户取消写入），检查 `production/review-mode.txt` 是否存在。

**如果存在**：读取它并注意当前模式 —— "Review mode is already set to `[current]`。" —— 跳过提示。

**如果不存在**：使用 `AskUserQuestion`：

- **提示**："还有一个设置步骤：你希望在完成工作流时进行多少设计审查？"
- **选项**：
  - `Full` —— Director 专家在每个关键工作流步骤进行审查。最适合团队、学习工作流，或当你希望对每个决策进行彻底反馈时。
  - `Lean (recommended)` —— 仅在阶段门转换时（/gate-check）进行 Director 审查。跳过每个 skill 的审查。适合 solo 开发者和小型团队。
  - `Solo` —— 完全没有 Director 审查。最大速度。最适合 game jam、原型，或如果审查感觉像开销。

选择后立即将选择写入 `production/review-mode.txt` —— 无需单独的"可以写入吗？"：
- `Full` → 写入 `full`
- `Lean (recommended)` → 写入 `lean`
- `Solo` → 写入 `solo`

如果 `production/` 目录不存在，创建它。

---

## 阶段 7：提供首个操作

写入计划后，不要在那里停下来。选择单个最高优先级的缺口，并立即使用 `AskUserQuestion` 提供处理。选择适用的第一个分支：

**如果 systems-index.md 中有括号状态值：**
使用 `AskUserQuestion`：
- "最紧急的修复是 `systems-index.md` —— [N] 行有括号状态值（例如 `Needs Revision (see notes)`）会立即破坏 /gate-check、/create-stories 和 /architecture-review。我可以就地修复这些。"
  - "现在修复 —— 编辑 systems-index.md"
  - "我会自己修复"
  - "完成 —— 把计划留给我"

**如果 ADR 缺少 `## Status`（且没有括号问题）：**
使用 `AskUserQuestion`：
- "最紧急的修复是为 [N] 个 ADR 添加 `## Status`：[列出文件名]。没有它，/story-readiness 会静默通过所有 ADR 检查。从 [第一个受影响的文件名] 开始？"
  - "是 —— 现在 retrofit [第一个受影响的文件名]"
  - "逐个 retrofit 所有 [N] 个 ADR"
  - "我会自己处理 ADR"

**如果 GDD 缺少 Acceptance Criteria（且没有上述 blocking 问题）：**
使用 `AskUserQuestion`：
- "最紧急的缺口是 [N] 个 GDD 缺少 Acceptance Criteria：[列出文件名]。没有它们，/create-stories 无法生成 story。从 [最高优先级的 GDD 文件名] 开始？"
  - "是 —— 现在为 [GDD 文件名] 添加 Acceptance Criteria"
  - "逐个处理所有 [N] 个 GDD"
  - "我会自己处理 GDD"

**如果没有 BLOCKING 或 HIGH 缺口：**
使用 `AskUserQuestion`：
- "没有 blocking 缺口 —— 此项目与模板兼容。接下来做什么？"
  - "带我浏览 medium 优先级的改进"
  - "运行 /project-stage-detect 进行更广泛的健康检查"
  - "完成 —— 我会按自己的节奏完成计划"

> **采用计划已保存到 `docs/adoption-plan-[date].md`。** 随时重新运行 `/adopt` 在你完成时重新检查剩余缺口。

---

## 协作协议

1. **静默读取** —— 在展示任何内容之前完成完整审计
2. **先展示摘要** —— 让用户在询问写入之前看到范围
3. **写入前询问** —— 在创建采用计划文件之前始终确认
4. **提供但不强迫** —— 计划是建议性的；用户决定修复什么以及何时修复
5. **一次一个操作** —— 交出计划后，提供一个具体的下一步，而不是同时列出六件事
6. **永远不要重新生成现有产物** —— 仅填补现有内容中的缺口；不要重写已有内容的 GDD、ADR 或 story
