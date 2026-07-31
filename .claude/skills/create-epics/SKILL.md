---
name: create-epics
description: "Translate approved GDDs + architecture into epics — one epic per architectural module. Defines scope, governing ADRs, engine risk, and untraced requirements. Does NOT break into stories — run /create-stories [epic-slug] after each epic is created."
argument-hint: "[system-name | layer: foundation|core|feature|presentation | all] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Task, AskUserQuestion
model: sonnet
agent: technical-director
---

# Create Epics

epic 是一个命名的、有边界的工作体，对应一个架构模块。它定义**需要构建什么**以及**架构上由谁拥有**。它不规定实现步骤——那是 story 的工作。

**每个层级运行一次此技能**，随着开发接近该层级。在 Core 接近完成之前不要创建 Feature 层 epic——设计会发生变化。

**输出：** `production/epics/[epic-slug]/EPIC.md` + `production/epics/index.md`

**每个 epic 后的下一步：** `/create-stories [epic-slug]`

**何时运行：** `/create-control-manifest` 和 `/architecture-review` 通过后。

---

## 1. 解析参数

解析审查模式（一次，存储供本次运行的所有门生成使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

参见 `.claude/docs/director-gates.md` 了解完整的检查模式。

**模式：**
- `/create-epics all` — 按层级顺序处理所有系统
- `/create-epics layer: foundation` — 仅 Foundation 层
- `/create-epics layer: core` — 仅 Core 层
- `/create-epics layer: feature` — 仅 Feature 层
- `/create-epics layer: presentation` — 仅 Presentation 层
- `/create-epics [system-name]` — 一个特定系统
- 无参数 — 询问："你想为哪个层级或系统创建 epic？"

---

## 2. 加载输入

### 步骤 2a — 摘要扫描（快速）

在完整读取任何内容之前，Grep 所有 GDD 的 `## Summary` 部分：

```
Grep pattern="## Summary" glob="design/gdd/*.md" output_mode="content" -A 5
```

对于 `layer:` 或 `[system-name]` 模式：根据摘要快速参考仅筛选范围内的 GDD。跳过范围外任何内容的完整读取。

### 步骤 2b — 完整文档加载（仅范围内的系统）

使用步骤 2a 的 grep 结果，确定哪些系统在范围内。**仅对范围内的系统**读取完整文档——不要读取范围外系统或层级的 GDD 或 ADR。

为范围内的系统读取：

- `design/gdd/systems-index.md` — 权威系统列表、层级、优先级
- 仅范围内的 GDD（Approved 或 Designed 状态，按步骤 2a 结果筛选）
- `docs/architecture/architecture.md` — 模块所有权和 API 边界
- **仅涵盖范围内系统的** Accepted ADR — 读取 "GDD Requirements Addressed"、"Decision" 和 "Engine Compatibility" 部分；跳过不相关领域的 ADR
- `docs/architecture/control-manifest.md` — 从头部获取 manifest 版本日期
- `docs/architecture/tr-registry.yaml` — 用于追踪需求到 ADR 覆盖
- `docs/engine-reference/[engine]/VERSION.md` — 引擎名称、版本、风险级别

报告："已加载 [N] 个 GDD，[M] 个 ADR，引擎：[name + version]。"

---

## 3. 处理顺序

按依赖安全的层级顺序处理：
1. **Foundation**（无依赖）
2. **Core**（依赖 Foundation）
3. **Feature**（依赖 Core）
4. **Presentation**（依赖 Feature + Core）

每个层级内，使用 `systems-index.md` 中的顺序。

---

## 4. 定义每个 Epic

对于每个系统，将其映射到 `architecture.md` 中的一个架构模块。

对照 TR 注册表检查 ADR 覆盖：
- **已追踪需求**：有 Accepted ADR 覆盖的 TR-ID
- **未追踪需求**：没有 ADR 的 TR-ID — 在继续前警告

在写入任何内容之前向用户展示：

```
## Epic: [系统名称]

**层级**：[Foundation / Core / Feature / Presentation]
**GDD**：design/gdd/[filename].md
**架构模块**：[来自 architecture.md 的模块名称]
**管辖 ADR**：[ADR-NNNN, ADR-MMMM]
**引擎风险**：[LOW / MEDIUM / HIGH — 管辖 ADR 中的最高风险]
**ADR 覆盖的 GDD 需求**：[N / 总数]
**未追踪需求**：[列出没有 ADR 的 TR-ID，或 "None"]
```

如果存在未追踪需求：
> "⚠️ [system] 中有 [N] 个需求没有 ADR。可以创建 epic，但这些需求的 story 将被标记为 Blocked，直到 ADR 存在。先运行 `/architecture-decision`，或使用占位符继续。"

使用 `AskUserQuestion`：
- 提示："我可以创建 Epic: [name] 吗？"
- 选项：
  - `[A] 是，创建它`
  - `[B] 跳过此 epic`
  - `[C] 暂停 — 我需要先写 ADR`

---

## 4b. Producer Epic 结构门

**审查模式检查** — 在生成 PR-EPIC 之前应用：
- `solo` → 跳过。注意："PR-EPIC 已跳过 — Solo 模式。" 进入第 5 步（写入 epic 文件）。
- `lean` → 跳过（不是 PHASE-GATE）。注意："PR-EPIC 已跳过 — Lean 模式。" 进入第 5 步（写入 epic 文件）。
- `full` → 正常生成。

在当前层级的所有 epic 都已定义（第 4 步对所有范围内的系统完成）后，在写入任何文件之前，通过 Task 使用门 **PR-EPIC**（`.claude/docs/director-gates.md`）生成 `producer`。

传递：完整的 epic 结构摘要（所有 epic、它们的范围摘要、管辖 ADR 计数）、正在处理的层级、里程碑时间线和团队产能。

展示 producer 的评估。

如果 UNREALISTIC：提供修改 epic 边界（拆分过大或合并过小的 epic）。在写入前修改并重新运行门。

如果 CONCERNS，使用 `AskUserQuestion`：
- 提示："Producer 对 epic 结构提出了担忧。你想如何继续？"
- 选项：
  - `[A] 按计划继续 — 我接受 producer 的担忧`
  - `[B] 修改 epic 边界 — 按建议拆分或合并`
  - `[C] 停止 — 我想重新考虑范围`

如果 [A]：进入第 5 步。
如果 [B]：从第 4 步修改 epic 定义并重新运行 producer 门。
如果 [C]：停止。裁决：**BLOCKED** — 用户想重新考虑 epic 范围。

在 producer 门解决之前不要写入 epic 文件。

---

## 5. 写入 Epic 文件

批准后，询问："我可以将 epic 文件写入 `production/epics/[epic-slug]/EPIC.md` 吗？"

用户确认后，写入：

### `production/epics/[epic-slug]/EPIC.md`

```markdown
# Epic: [系统名称]

> **层级**：[Foundation / Core / Feature / Presentation]
> **GDD**：design/gdd/[filename].md
> **架构模块**：[模块名称]
> **状态**：Ready
> **Stories**：尚未创建 — 运行 `/create-stories [epic-slug]`

## Overview

[1 段描述此 epic 实现什么，源自 GDD Overview 和架构模块的声明职责]

## Governing ADRs

| ADR | Decision Summary | Engine Risk |
|-----|-----------------|-------------|
| ADR-NNNN: [title] | [1 行摘要] | LOW/MEDIUM/HIGH |

## GDD Requirements

| TR-ID | Requirement | ADR Coverage |
|-------|-------------|--------------|
| TR-[system]-001 | [来自注册表的需求文本] | ADR-NNNN ✅ |
| TR-[system]-002 | [需求文本] | ❌ 无 ADR |

## Definition of Done

此 epic 在以下情况完成：
- 所有 story 已实现、审查并通过 `/story-done` 关闭
- `design/gdd/[filename].md` 中的所有验收标准已验证
- 所有 Logic 和 Integration story 在 `tests/` 中有通过的测试文件
- 所有 Visual/Feel 和 UI story 在 `production/qa/evidence/` 中有签字确认的证据文档

## Next Step

运行 `/create-stories [epic-slug]` 将此 epic 分解为可实现的 story。
```

### 更新 `production/epics/index.md`

创建或更新主索引：

```markdown
# Epics Index

最后更新：[date]
引擎：[name + version]

| Epic | 层级 | 系统 | GDD | Stories | 状态 |
|------|-------|--------|-----|---------|--------|
| [name] | Foundation | [system] | [file] | 尚未创建 | Ready |
```

---

## 6. Gate-Check 提醒

在写入请求范围内的所有 epic 后：

- **Foundation + Core 完成**：这些是 Pre-Production → Production 门所必需的。运行 `/gate-check production` 检查准备情况。
- **提醒**：Epic 定义范围。Story 定义实现步骤。在每个开发者可以接手工作之前，为每个 epic 运行 `/create-stories [epic-slug]`。

---

## 协作协议

1. **一次一个 epic** — 在询问创建之前展示每个 epic 定义
2. **对缺口发出警告** — 在继续前标记未追踪的需求
3. **写入前询问** — 在写入任何文件之前获得每个 epic 的批准
4. **不发明** — 所有内容来自 GDD、ADR 和架构文档
5. **绝不创建 story** — 此技能在 epic 级别停止

处理完所有请求的 epic 后：

- **裁决：COMPLETE** — 已写入 [N] 个 epic。每个 epic 运行 `/create-stories [epic-slug]`。
- **裁决：BLOCKED** — 用户拒绝了所有 epic，或未找到符合条件的系统。
