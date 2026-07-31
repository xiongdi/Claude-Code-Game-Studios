---
name: create-stories
description: "Break a single epic into implementable story files. Reads the epic, its GDD, governing ADRs, and control manifest. Each story embeds its GDD requirement TR-ID, ADR guidance, acceptance criteria, story type, and test evidence path. Run after /create-epics for each epic."
argument-hint: "[epic-slug | epic-path] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Task, AskUserQuestion
model: sonnet
agent: lead-programmer
---

# Create Stories

story 是一个单一的可实现行为——足够小，可以在一个专注的会话中完成，自包含，并可完全追溯到 GDD 需求和 ADR 决策。story 是开发者接手的单元。epic 是架构师定义的单元。

**每个 epic 运行此技能**，不是每个层级。先为 Foundation epic 运行，然后是 Core，依此类推——匹配依赖顺序。

**输出：** `production/epics/[epic-slug]/story-NNN-[slug].md` 文件

**上一步：** `/create-epics [system]`
**story 存在后的下一步：** `/story-readiness [story-path]` 然后 `/dev-story [story-path]`

---

## 1. 解析参数

如果存在，提取 `--review [full|lean|solo]` 并存储为本次运行的审查模式覆盖。如果未提供，读取 `production/review-mode.txt`（如果缺失则默认为 `lean`）。此解析后的模式适用于此技能中的所有门生成——在每次门调用前应用 `.claude/docs/director-gates.md` 中的检查模式。

- `/create-stories [epic-slug]` — 例如 `/create-stories combat`
- `/create-stories production/epics/combat/EPIC.md` — 也接受完整路径
- 无参数 — 询问："你想将哪个 epic 分解为 story？" Glob `production/epics/*/EPIC.md` 并列出具有 Ready 状态的可用 epic。

---

## 2. 加载此 Epic 的所有内容

完整读取：

- `production/epics/[epic-slug]/EPIC.md` — epic overview、管辖 ADR、GDD 需求表
- epic 的 GDD（`design/gdd/[filename].md`）— 读取所有 8 个部分，特别是 Acceptance Criteria、Formulas 和 Edge Cases
- epic 中列出的所有管辖 ADR — 读取 Decision、Implementation Guidelines、Engine Compatibility 和 Engine Notes 部分
- `docs/architecture/control-manifest.md` — 提取此 epic 层级的规则；从头部记录 Manifest Version 日期
- `docs/architecture/tr-registry.yaml` — 加载此系统的所有 TR-ID

**ADR 存在性验证**：从 epic 读取管辖 ADR 列表后，确认每个 ADR 文件在磁盘上存在。如果找不到任何 ADR 文件，在分解任何 story **之前立即停止**：

> "Epic 引用了 [ADR-NNNN: title]，但未找到 `docs/architecture/[adr-file].md`。
> 检查 epic 的 Governing ADRs 列表中的文件名，或运行 `/architecture-decision` 创建它。在所有引用的 ADR 文件存在之前无法创建 story。"

在所有引用的 ADR 文件确认存在之前，不要进入第 3 步。

报告："已加载 epic [name]、GDD [filename]、[N] 个管辖 ADR（全部确认存在）、control manifest v[date]。"

---

## 3. 按类型分类 Story

**Story 类型分类**——根据其验收标准为每个 story 分配类型：

| Story 类型 | 当标准引用...时分配 |
|---|---|
| **Logic** | 公式、数值阈值、状态转换、AI 决策、计算 |
| **Integration** | 两个或多个系统交互、信号跨越边界、save/load 往返 |
| **Visual/Feel** | 动画行为、VFX、"感觉响应迅速"、时机、屏幕震动、音频同步 |
| **UI** | 菜单、HUD 元素、按钮、屏幕、对话框、工具提示 |
| **Config/Data** | 平衡调优值、仅数据文件更改——无新代码逻辑 |

混合 story：分配具有最高实现风险的类型。
类型决定 `/story-done` 关闭 story 之前需要什么测试证据。

---

## 4. 将 GDD 分解为 Story

对于每个 GDD 验收标准：

1. 将需要相同核心实现的相关标准分组
2. 每组 = 一个 story
3. 排序 story：基础行为优先，边缘情况其次，UI 最后

**story 大小规则：** 一个 story = 一个专注的会话（约 2-4 小时）。如果一组标准需要更长时间，拆分为两个 story。

对于每个 story，确定：
- **GDD 需求**：这满足哪个验收标准？
- **TR-ID**：在 `tr-registry.yaml` 中查找。使用稳定 ID。如果没有匹配，使用 `TR-[system]-???` 并警告。
- **管辖 ADR**：哪个 ADR 管辖如何实现？
  - `Status: Accepted` → 正常嵌入
  - `Status: Proposed` → 设置 story `Status: Blocked` 并注明："BLOCKED: ADR-NNNN 是 Proposed — 运行 `/architecture-decision` 推进它"
  - **多个 ADR 适用**：在 story 的 `Governing ADRs:` 字段中列出所有管辖 ADR。将最直接控制实现模式的指定为主要（列表中的第一个）。其他列为次要参考。
  - **没有 ADR 适用**：在 story 的 ADR 字段中写 `ADR: N/A — [简要原因，例如 "纯数据配置，不需要架构模式"]`。不要留空——空白 ADR 字段意味着"未检查"，而不是"不适用"。
- **Story 类型**：来自第 3 步分类
- **引擎风险**：来自 ADR 的 Knowledge Risk 字段

---

## 4b. QA Lead Story 就绪门

**审查模式检查** — 在生成 QL-STORY-READY 之前应用：
- `solo` → 跳过。注意："QL-STORY-READY 已跳过 — Solo 模式。" 进入第 5 步（展示 story 供审查）。
- `lean` → 跳过（不是 PHASE-GATE）。注意："QL-STORY-READY 已跳过 — Lean 模式。" 进入第 5 步（展示 story 供审查）。
- `full` → 正常生成。

在所有 story 分解完成（第 4 步完成）后，但在展示它们供写入批准之前，通过 Task 使用门 **QL-STORY-READY**（`.claude/docs/director-gates.md`）生成 `qa-lead`。

传递：完整的 story 列表及验收标准、story 类型和 TR-ID；epic 的 GDD 验收标准供参考。

展示 QA lead 的评估。对于每个标记为 GAPS 或 INADEQUATE 的 story，在继续之前修改验收标准——具有不可测试标准的 story 无法正确实现。一旦所有 story 达到 ADEQUATE，继续。

**生成测试规范之前**：Glob `production/qa/qa-plan-*.md` 查找最近修改的文件。如果找到，读取它并检查它是否包含此 epic 中 story 的测试用例规范（在计划的 Automated Tests Required 部分查找 story 标题或 slug）。如果存在匹配的规范：
- 使用 `AskUserQuestion`：
  - 提示："在 [path] 存在一个 QA 计划，其中包含其中一些 story 的测试规范。你想如何继续？"
  - 选项：
    - `使用 QA 计划中的现有规范 — 将它们嵌入到 story 文件中（推荐）`
    - `让 qa-lead 生成新规范 — 覆盖 QA 计划`
    - `跳过测试规范生成 — 我将手动填写 ## QA Test Cases`
- 如果"使用现有规范"：从 qa-plan 中提取每个匹配 story 的测试用例规范，并直接嵌入到 `## QA Test Cases` 部分。这些 story 不需要生成 qa-lead。仅为 qa-plan 中没有覆盖的 story 生成 qa-lead。
- 如果"生成新规范"：按以下正常方式继续 qa-lead 生成。
- 如果"跳过"：在 `## QA Test Cases` 中保留占位符：`*测试用例尚未定义 — 运行 /qa-plan 生成它们。*`

**ADEQUATE 之后**（或 qa-plan 导入后）：对于每个 Logic 和 Integration story，让 qa-lead 生成具体的测试用例规范——每个验收标准一个——使用此格式：

```
测试：[criterion text]
  给定：[precondition]
  当：[action]
  那么：[预期结果 / 断言]
  边缘情况：[要测试的边界值或失败状态]
```

对于 Visual/Feel 和 UI story，生成手动验证步骤：

```
手动检查：[criterion text]
  设置：[如何到达该状态]
  验证：[寻找什么]
  通过条件：[明确的通过描述]
```

这些测试用例规范直接嵌入到每个 story 的 `## QA Test Cases` 部分。开发者针对这些用例实现。程序员不会从头开始编写测试——QA 已经定义了"完成"的样子。

---

## 5. 展示 Story 供审查

在写入任何文件之前，展示完整的 story 列表：

```
## Epic: [name] 的 Story

Story 001: [title] — Logic — ADR-NNNN
  覆盖：TR-[system]-001（[需求的 1 行摘要]）
  所需测试：tests/unit/[system]/[slug]_test.[ext]

Story 002: [title] — Integration — ADR-MMMM
  覆盖：TR-[system]-002、TR-[system]-003
  所需测试：tests/integration/[system]/[slug]_test.[ext]

Story 003: [title] — Visual/Feel — ADR-NNNN
  覆盖：TR-[system]-004
  所需证据：production/qa/evidence/[slug]-evidence.md

[共 N 个 story：N 个 Logic、N 个 Integration、N 个 Visual/Feel、N 个 UI、N 个 Config/Data]
```

使用 `AskUserQuestion`：
- 提示："我可以将这 [N] 个 story 写入 `production/epics/[epic-slug]/` 吗？"
- 选项：`[A] 是 — 写入所有 [N] 个 story` / `[B] 还不行 — 我想先审查或调整`

---

## 6. 写入 Story 文件

对于每个 story，写入 `production/epics/[epic-slug]/story-[NNN]-[slug].md`：

```markdown
# Story [NNN]: [title]

> **Epic**：[epic name]
> **状态**：Ready
> **层级**：[Foundation / Core / Feature / Presentation]
> **类型**：[Logic | Integration | Visual/Feel | UI | Config/Data]
> **估算**：[小时或 t-shirt 大小 — 在 sprint 计划前填写]
> **Manifest 版本**：[来自 control-manifest.md 头部的日期]
> **最后更新**：[由 /dev-story 在实现开始时设置]

## Context

**GDD**：`design/gdd/[filename].md`
**需求**：`TR-[system]-NNN`
*（需求文本位于 `docs/architecture/tr-registry.yaml` 中 — 在审查时重新读取）*

**管辖实现的 ADR**：[ADR-NNNN: title]
**ADR 决策摘要**：[ADR 决策内容的 1-2 句摘要]

**引擎**：[name + version] | **风险**：[LOW / MEDIUM / HIGH]
**引擎备注**：[来自 ADR Engine Compatibility 部分 — cutoff 后 API、需要验证的内容]

**Control Manifest 规则（此层级）**：
- 必需：[相关的必需模式]
- 禁止：[相关的禁止模式]
- 护栏：[相关的性能护栏]

---

## Acceptance Criteria

*来自 GDD `design/gdd/[filename].md`，范围限定于此 story：*

- [ ] [标准 1 — 直接来自 GDD]
- [ ] [标准 2]
- [ ] [如适用，性能标准]

---

## Implementation Notes

*源自 ADR-NNNN Implementation Guidelines：*

[来自 ADR 的具体、可操作的指导。不要以改变含义的方式意译。
这是程序员阅读的内容，而不是 ADR。]

---

## Out of Scope

*由相邻 story 处理 — 不要在此实现：*

- [Story NNN+1]：[它处理什么]

---

## QA Test Cases

*由 qa-lead 在 story 创建时编写。开发者针对这些实现 — 不要在实现期间发明新的测试用例。*

**[对于 Logic / Integration story — 自动化测试规范]：**

- **AC-1**：[criterion text]
  - 给定：[precondition]
  - 当：[action]
  - 那么：[assertion]
  - 边缘情况：[边界值 / 失败状态]

**[对于 Visual/Feel / UI story — 手动验证步骤]：**

- **AC-1**：[criterion text]
  - 设置：[如何到达该状态]
  - 验证：[寻找什么]
  - 通过条件：[明确的通过描述]

---

## Test Evidence

**Story 类型**：[type]
**所需证据**：
- Logic：`tests/unit/[system]/[story-slug]_test.[ext]` — 必须存在并通过
- Integration：`tests/integration/[system]/[story-slug]_test.[ext]` 或试玩文档
- Visual/Feel：`production/qa/evidence/[story-slug]-evidence.md` + 签字确认
- UI：`production/qa/evidence/[story-slug]-evidence.md` 或交互测试
- Config/Data：冒烟检查通过（`production/qa/smoke-*.md`）

**状态**：[ ] 尚未创建

---

## Dependencies

- 依赖于：[Story NNN-1 必须为 DONE，或 "None"]
- 解锁：[Story NNN+1，或 "None"]
```

### 同时更新 `production/epics/[epic-slug]/EPIC.md`

将 "Stories: Not yet created" 行替换为填充的表：

```markdown
## Stories

| # | Story | 类型 | 状态 | ADR |
|---|-------|------|--------|-----|
| 001 | [title] | Logic | Ready | ADR-NNNN |
| 002 | [title] | Integration | Ready | ADR-MMMM |
```

### 同时更新 `production/epics/index.md`

在索引表中查找匹配此 epic 的行（按 epic 名称或 slug）。将其 `Stories` 列从 `Not yet created` 更新为 `[N] stories`（其中 N 是刚刚写入的数量）。如果索引文件不存在，静默跳过。

---

## 7. 写入后

使用 `AskUserQuestion` 以上下文感知的后续步骤结束：

检查：
- `production/epics/` 中是否还有其他没有 story 的 epic？列出它们。
- 这是最后一个 epic 吗？如果是，将 `/sprint-plan` 作为选项包含。

Widget：
- 提示："[N] 个 story 已写入 `production/epics/[epic-slug]/`。下一步？"
- 选项（包含所有适用的）：
  - `[A] 开始实现 — 运行 /story-readiness [first-story-path]`（推荐）
  - `[B] 为 [next-epic-slug] 创建 story — 运行 /create-stories [slug]`（仅当其他 epic 还没有 story 时）
  - `[C] 计划 sprint — 运行 /sprint-plan new`（仅当所有 epic 都有 story 时）
  - `[D] 本次会话停在这里`

在输出中注明："按顺序处理 story — 每个 story 的 `Depends on:` 字段告诉你开始它之前什么必须为 DONE。"

---

## 协作协议

1. **展示前先读取** — 在显示 story 列表之前静默加载所有输入
2. **问一次** — 在一个摘要中展示 epic 的所有 story，而不是一次一个
3. **对受阻 story 发出警告** — 在写入前标记任何具有 Proposed ADR 的 story
4. **写入前询问** — 在写入文件之前获得完整 story 集的批准
5. **不发明** — 验收标准来自 GDD，实现备注来自 ADR，规则来自 manifest
6. **绝不开始实现** — 此技能在 story 文件级别停止

写入（或拒绝）后：

- **裁决：COMPLETE** — [N] 个 story 已写入 `production/epics/[epic-slug]/`。运行 `/story-readiness` → `/dev-story` 开始实现。
- **裁决：BLOCKED** — 用户拒绝。未写入 story 文件。
