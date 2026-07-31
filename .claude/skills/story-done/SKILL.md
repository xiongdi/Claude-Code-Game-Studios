---
name: story-done
description: "Story 完成审查。读取 story 文件，对照实现验证每个验收标准，检查 GDD/ADR 偏差，提示代码审查，将 story 状态更新为 Complete，并浮现 sprint 中下一个就绪的 story。"
argument-hint: "[story-file-path] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write, Edit, AskUserQuestion, Task
model: sonnet
---

# Story Done

此 skill 在设计和实现之间闭环。在任何 story 实现结束时运行它。它确保在 story 标记为完成之前验证每个验收标准，GDD 和 ADR 偏差被明确记录而非悄悄引入，代码审查被提示而非遗忘，story 文件反映实际完成状态。

**输出：** 更新的 story 文件（Status: Complete）+ 浮现的下一个 story。

---

## Phase 1: 查找 Story

解析 review 模式（一次性解析，存储供本运行的所有 gate 生成使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

参见 `.claude/docs/director-gates.md` 了解完整的检查模式。

**如果提供了文件路径**（如 `/story-done production/epics/core/story-damage-calculator.md`）：
直接读取该文件。

**如果没有提供参数：**

1. 检查 `production/session-state/active.md` 中当前活动的 story。
2. 如果在那里未找到，读取 `production/sprints/` 中最近的文件并
   查找标记为 IN PROGRESS 的 stories。
3. 如果找到多个进行中的 stories，使用 `AskUserQuestion`：
   - "Which story are we completing?"
   - 选项：列出进行中的 story 文件名。
4. 如果找不到 story，请用户提供路径。

---

## Phase 2: 读取 Story

完整读取 story 文件。提取并保持在上下文中：

- **Story 名称和 ID**
- **引用的 GDD Requirement TR-ID(s)**（如 `TR-combat-001`）
- **嵌入 story 头部的 Manifest Version**（如 `2026-03-10`）
- **引用的 ADR reference(s)**
- **Acceptance Criteria** — 完整列表（每个复选框项目）
- **实现文件** — 在"files to create/modify"下列出的文件
- **Story Type** — story 头部的 `Type:` 字段（Logic / Integration / Visual/Feel / UI / Config/Data）
- **Engine notes** — 注明的任何引擎专属约束
- **Definition of Done** — 如有，story 级别的 DoD
- **估算与实际范围** — 如果记录了估算

同时读取：
- `docs/architecture/tr-registry.yaml` — 查找 story 中的每个 TR-ID。
  读取登记条目的*当前* `requirement` 文本。这是 GDD 要求的
  真实来源 — 不要使用 story 内可能内联引用的任何需求文本
  （可能已过时）。
- 引用的 GDD 章节 — 仅验收标准和关键规则，不是
  完整文档。用于交叉核对登记文本是否仍然准确。
- 引用的 ADR(s) — 仅 Decision 和 Consequences 章节
- `docs/architecture/control-manifest.md` 头部 — 提取当前
  `Manifest Version:` 日期（用于 Phase 4 陈旧性检查）

---

## Phase 3: 验证验收标准

对于 story 中的每个验收标准，尝试使用三种方法之一进行验证：

### 自动验证（无需询问即可运行）

- **文件存在检查**：`Glob` 查找 story 说要创建的文件。
- **测试通过检查**：如果提到了测试文件路径，通过 `Bash` 运行它。
- **无硬编码值检查**：`Grep` 查找游戏代码路径中的数字字面量，
  这些应该在配置文件中。
- **无硬编码字符串检查**：`Grep` 查找 `src/` 中面向玩家的字符串，
  这些应该在本地化文件中。
- **依赖检查**：如果标准说"依赖于 X"，检查 X 是否存在。

### 带确认的手动验证（使用 `AskUserQuestion`）

- 关于主观质量的标准（"感觉响应迅速"、"动画播放正确"）
- 关于游戏行为的标准（"玩家受到伤害当..."、"敌人响应..."）
- 性能标准（"在 Xms 内完成"）— 询问是否已分析或接受为假设

将最多 4 个手动验证问题批量放入单次 `AskUserQuestion` 调用：

```
question: "[criterion] 是否满足？"
options: "Yes — 通过", "No — 失败", "Not tested yet"
```

### 无法验证的（标记但不阻塞）

- 需要完整游戏构建才能测试的标准（端到端游戏场景）
- 标记为：`DEFERRED — requires playtest session`

### 测试-标准可追溯性

完成上述通过/失败/延期检查后，将每个验收标准映射到覆盖它的测试：

对于 story 中的每个验收标准：

1. 问：是否有测试 — 单元、集成或确认的手动 playtest —
   直接验证此标准？
   - **单元测试**：检查 `tests/unit/` 中是否有测试文件或函数名
    与标准的主题匹配（使用 `Glob` 和 `Grep`）
   - **集成测试**：类似地检查 `tests/integration/`
   - **手动确认**：如果标准通过上述 `AskUserQuestion` 验证且
    答案为 "Yes — passes"，将其计为手动测试

2. 生成可追溯性表：

```
| Criterion | Test | Status |
|-----------|------|--------|
| AC-1: [criterion text] | tests/unit/test_foo.gd::test_bar | COVERED |
| AC-2: [criterion text] | Manual playtest confirmation | COVERED |
| AC-3: [criterion text] | — | UNTESTED |
```

3. 应用这些升级规则：

   - 如果 **>50% 的标准为 UNTESTED**：升级为 **BLOCKING** — 测试
     覆盖率不足以确认 story 实际完成。Phase 6 的裁决
     在覆盖率改善之前不能为 COMPLETE。
   - 如果 **部分（≤50%）标准为 UNTESTED**：保持 ADVISORY — 不阻塞
     完成，但必须出现在完成说明中。
   - 如果 **所有标准为 COVERED**：除在报告中包含表格外无需其他操作。

4. 对于任何 ADVISORY 未测试的标准，添加到 Phase 7 的完成说明中：
   `"未测试的标准：[AC-N 列表]。建议在后续 story 中添加测试。"`

### 测试证据要求

基于 Phase 2 中提取的 Story Type，检查所需证据：

| Story Type | 所需证据 | 关卡级别 |
|---|---|---|
| **Logic** | `tests/unit/[system]/` 中的自动化单元测试 — 必须存在并通过 | BLOCKING |
| **Integration** | `tests/integration/[system]/` 中的集成测试或 playtest 文档 | BLOCKING |
| **Visual/Feel** | `production/qa/evidence/` 中的截图 + 签字 | ADVISORY |
| **UI** | `production/qa/evidence/` 中的手动演练文档或交互测试 | ADVISORY |
| **Config/Data** | `production/qa/smoke-*.md` 中的 smoke check 通过报告 | ADVISORY |

**对于 Logic stories**：首先读取 story 的 **Test Evidence** 章节以提取
确切需要的文件路径。使用 `Glob` 检查该确切路径。如果未找到确切路径，
还要广泛搜索 `tests/unit/[system]/`（文件可能放在稍有不同的位置）。
如果在两个位置都未找到测试文件：
- 标记为 **BLOCKING**："Logic story 没有单元测试文件。Story 要求在 `[Test Evidence 章节中的确切路径]`。在标记此 story 为 Complete 之前创建并运行测试。"

**对于 Integration stories**：读取 story 的 **Test Evidence** 章节了解确切
需要的路径。首先使用 `Glob` 检查该确切路径，然后广泛搜索
`tests/integration/[system]/`，然后检查 `production/session/logs/` 中是否有引用此 story 的
playtest 记录。
如果都未找到：标记为 **BLOCKING**（与 Logic 相同规则）。

**对于 Visual/Feel 和 UI stories**：glob `production/qa/evidence/` 查找引用此 story 的文件。
- 如果未找到：标记为 **ADVISORY** — "未找到手动测试证据。使用 test-evidence 模板创建 `production/qa/evidence/[story-slug]-evidence.md` 并在最终关闭前获得签字。"
- 如果找到：读取文件并检查签字表中是否有未勾选的框。Grep 匹配 `| .* | .* | .* | \[ \] Approved` 的行（带有未勾选复选框的签字行）。如果找到任何未勾选的签字行：标记为 **ADVISORY** — "在 `[path]` 找到证据文件，但 [N] 个签字仍在等待中（在签字表中显示为 `[ ] Approved`）。在最终关闭前获得所需签字。注意：对于 solo 开发者，所有角色可由同一人签字。"
- 如果所有签字行显示 `[x] Approved` 或等价物：注明 "找到证据文件且所有签字已完成 — ADVISORY 通过。"

**对于 Config/Data stories**：检查是否有任何 `production/qa/smoke-*.md` 文件。
如果未找到：标记为 **ADVISORY** — "未找到 smoke check 报告。运行 `/smoke-check`。"

**如果未设置 Story Type**：标记为 **ADVISORY** —
"未声明 Story Type。在 story 头部添加 `Type: [Logic|Integration|Visual/Feel|UI|Config/Data]` 以在后续 story 中启用测试证据关卡执行。"

任何 BLOCKING 测试证据缺口都会阻止 Phase 6 的 COMPLETE 裁决。

---

## Phase 4: 检查偏差

将实现与设计文档进行比较。

自动运行这些检查：

1. **GDD 规则检查**：使用 `tr-registry.yaml` 中的当前需求文本
   （通过 story 的 TR-ID 查找），检查实现是否反映了 GDD
   当前实际要求的内容 — 而非 story 编写时的要求。
   `Grep` 实现文件中当前 GDD 章节提到的关键函数名、
   数据结构或类名。

2. **Manifest 版本陈旧性检查**：比较 story 头部嵌入的 `Manifest Version:` 日期
   与当前 `docs/architecture/control-manifest.md` 头部的 `Manifest Version:` 日期。
   - 如果匹配 → 静默通过。
   - 如果 story 的版本更旧 → 标记为 ADVISORY：
     `ADVISORY: Story 是针对 manifest v[story-date] 编写的；当前 manifest 为 v[current-date]。可能适用新规则。运行 /story-readiness 检查。`
   - 如果 control-manifest.md 不存在 → 跳过此检查。

3. **ADR 约束检查**：读取引用 ADR 的 Decision 章节。检查
   `docs/architecture/control-manifest.md`（如果存在）中的禁止模式。
   `Grep` 查找 ADR 中明确禁止的模式。

4. **硬编码值检查**：`Grep` 实现文件中游戏逻辑里的数字字面量，
   这些应该在数据文件中。

5. **范围检查**：实现是否触及了 story 声明范围之外的文件？
   （未在"files to create/modify"中列出的文件）

对于发现的每个偏差，分类为：

- **BLOCKING** — 实现与 GDD 或 ADR 矛盾（必须在标记完成前修复）
- **ADVISORY** — 实现与规范略有偏差但功能等效（记录，用户决定）
- **OUT OF SCOPE** — 触及了 story 声明边界之外的额外文件（标记以供注意 — 可能是有效的或范围蔓延）

---

## Phase 4b: QA 覆盖率关卡

**Review 模式检查** — 在生成 QL-TEST-COVERAGE 之前应用：
- `solo` → 跳过。注明："QL-TEST-COVERAGE skipped — Solo mode." 继续到 Phase 5。
- `lean` → 跳过（不是 PHASE-GATE）。注明："QL-TEST-COVERAGE skipped — Lean mode." 继续到 Phase 5。
- `full` → 正常生成。

完成 Phase 4 的偏差检查后，使用 gate **QL-TEST-COVERAGE**（`.claude/docs/director-gates.md`）通过 Task 生成 `qa-lead`。

传递：
- story 文件路径和 story 类型
- Phase 3 中发现的测试文件路径（确切路径，或"none found"）
- story 的 `## QA Test Cases` 章节（story 创建时预先编写的测试规范）
- story 的 `## Acceptance Criteria` 列表

qa-lead 审查测试是否实际覆盖了指定的内容 — 不仅仅是文件是否存在。

应用裁决：
- **ADEQUATE** → 继续到 Phase 5
- **GAPS** → 标记为 **ADVISORY**："QA lead 发现覆盖率缺口：[列表]。Story 可以完成，但缺口应在后续 story 中解决。"
- **INADEQUATE** → 标记为 **BLOCKING**："QA lead：关键逻辑未测试。在覆盖率改善之前裁决不能为 COMPLETE。具体缺口：[列表]。"

对于 Config/Data stories 跳过此阶段（不需要代码测试）。

---

## Phase 5: Lead Programmer 代码审查关卡

**Review 模式检查** — 在生成 LP-CODE-REVIEW 之前应用：
- `solo` → 跳过。注明："LP-CODE-REVIEW skipped — Solo mode." 继续到 Phase 6（完成报告）。
- `lean` → 在继续前使用 `AskUserQuestion`：
  - 提示："Lean 模式下跳过代码审查。你是否对实现文件运行了 `/code-review`？"
  - 选项：
    - `Yes — /code-review 通过或被批准带建议`
    - `No — 为此 story 跳过代码审查`
    - `No — 我会在 sprint 结束前运行 /code-review`
  - 将答案记录在完成说明中（Phase 7）。所有三个选项都继续到 Phase 6。
- `full` → 正常生成。

使用 gate **LP-CODE-REVIEW**（`.claude/docs/director-gates.md`）通过 Task 生成 `lead-programmer`。

传递：实现文件路径、story 文件路径、相关 GDD 章节、管辖 ADR。

向用户展示裁决。如果 CONCERNS，通过 `AskUserQuestion` 展示：
- 选项：`Revise flagged issues` / `Accept and proceed` / `Discuss further`
如果 REJECT，在问题解决之前不要继续到 Phase 6 裁决。

如果 story 还没有实现文件（裁决在编码完成前运行），跳过此阶段并注明："LP-CODE-REVIEW 已跳过 — 未找到实现文件。在实现完成后运行。"

---

## Phase 6: 展示完成报告

在更新任何文件之前，展示完整报告：

```markdown
## Story Done: [Story Name]
**Story**: [file path]
**Date**: [today]

### Acceptance Criteria: [X/Y passing]
- [x] [Criterion 1] — auto-verified (test passes)
- [x] [Criterion 2] — confirmed
- [ ] [Criterion 3] — FAILS: [reason]
- [?] [Criterion 4] — DEFERRED: requires playtest

### Test-Criterion Traceability
| Criterion | Test | Status |
|-----------|------|--------|
| AC-1: [text] | [test file::test name] | COVERED |
| AC-2: [text] | Manual confirmation | COVERED |
| AC-3: [text] | — | UNTESTED |

### Test Evidence
**Story Type**: [Logic | Integration | Visual/Feel | UI | Config/Data | Not declared]
**Required evidence**: [unit test file | integration test or playtest | screenshot + sign-off | walkthrough doc | smoke check pass]
**Evidence found**: [YES — `[path]` | NO — BLOCKING | NO — ADVISORY]

### Deviations
[NONE] OR:
- BLOCKING: [description] — [GDD/ADR reference]
- ADVISORY: [description] — user accepted / flagged for tech debt

### Scope
[All changes within stated scope] OR:
- Extra files touched: [list] — [note whether valid or scope creep]

### Verdict: COMPLETE / COMPLETE WITH NOTES / BLOCKED
```

**裁决定义：**
- **COMPLETE**：所有标准通过，无阻塞性偏差
- **COMPLETE WITH NOTES**：所有标准通过，建议性偏差已记录
- **BLOCKED**：失败的标准或阻塞性偏差必须先解决

如果裁决为 **BLOCKED**：不要继续到 Phase 7。列出必须修复的内容。
提供修复阻塞项目的帮助。

---

## Phase 7: 更新 Story 状态

在写入任何内容之前使用 `AskUserQuestion`：
- 提示："验证完成。你想如何进行？"
- 选项：
  - `关闭 story — 更新文件，标记为 Complete，记录说明（推荐）`
  - `关闭并将建议性偏差记录为 docs/tech-debt-register.md 中的 tech debt`
  - `有问题我想先修复 — 暂不关闭`
  - `接受偏差并按原样关闭`

如果选择"Close"、"Close and log tech debt"或"Accept deviations"：编辑 story 文件。
如果选择"Close and log tech debt"：更新 story 文件后，还要将建议性偏差追加到 `docs/tech-debt-register.md`（如果文件不存在则创建）。
如果选择"Fix first"：在此停止并列出用户标记的内容。不要写入任何文件。

1. 更新状态字段：`Status: Complete`
2. 将 story 头部的 `Last Updated:` 字段更新为今天的日期（格式：`YYYY-MM-DD`）。如果该字段不存在，在 `Status:` 行后添加。
3. 在底部添加 `## Completion Notes` 章节：

```markdown
## Completion Notes
**Completed**: [date]
**Criteria**: [X/Y passing] ([any deferred items listed])
**Deviations**: [None] or [list of advisory deviations]
**Test Evidence**: [Logic: test file at path | Visual/Feel: evidence doc at path | None required (Config/Data)]
**Code Review**: [Pending / Complete / Skipped]
```

4. 如果用户选择了"Close and log tech debt"：将每个建议性偏差追加到 `docs/tech-debt-register.md`，格式如下：
   ```
   - **[date]** ([story title]): [deviation description] — tracked from [story file path]
   ```
   如果文件不存在，用 `# Tech Debt Register` 标题创建。

5. **更新 `production/sprint-status.yaml`**（如果存在）：
   - 找到与此 story 的文件路径或 ID 匹配的条目
   - 设置 `status: done` 和 `completed: [today's date]`
   - 更新顶层 `updated` 字段
   - 这是静默更新 — 无需额外批准（已在上述步骤中批准）

6. **建议 git 提交**：输出一个现成的提交命令，涵盖 dev-story 摘要中的实现文件和更新的 story 文件：

```
Suggested commit:
git add [src/ and tests/ files changed during implementation] [story-file-path]
git commit -m "feat: [story title] ([TR-ID])"
```

`validate-commit.sh` hook 将自动验证设计文档引用并检查硬编码值。

### 会话状态更新

更新 story 文件后，静默追加到
`production/session-state/active.md`：

    ## Session Extract — /story-done [date]
    - Verdict: [COMPLETE / COMPLETE WITH NOTES / BLOCKED]
    - Story: [story file path] — [story title]
    - Tech debt logged: [N items, or "None"]
    - Next recommended: [next ready story title and path, or "None identified"]

如果 `active.md` 不存在，用此块作为初始内容创建。
在对话中确认："Session state updated."

---

## Phase 8: 浮现下一个 Story

完成后，帮助开发者保持势头：

1. 从 `production/sprints/` 读取当前 sprint 计划。
2. 找到符合以下条件的 stories：
   - Status: READY 或 NOT STARTED
   - 未被其他不完整 stories 阻塞
   - 在 Must Have 或 Should Have 层级

展示：

```
### 下一步
以下 stories 已准备好可以接手：
1. [Story name] — [一句话描述] — 估算：[X 小时]
2. [Story name] — [一句话描述] — 估算：[X 小时]

运行 `/story-readiness [path]` 在开始前确认 story 是否可实现。
```

如果此 sprint 中没有更多 Must Have stories 剩余（全部为 Complete 或 Blocked）：

```
### Sprint 收尾序列

所有 Must Have stories 已完成。推进前需要 QA 签字。
按顺序运行这些：

1. `/smoke-check sprint` — 验证关键路径仍然端到端工作
2. `/team-qa sprint` — 完整 QA 周期：测试用例执行、bug 分拣、签字报告
3. `/retrospective` — 记录进展顺利的内容、不顺利的内容和下一个 sprint 的行动项
4. `/gate-check` — 一旦 QA 批准就推进到下一阶段（仅在推进阶段时）
5. `/sprint-plan new` — 计划下一个 sprint，融入速度数据和 retrospective 行动项

在 `/team-qa` 返回 APPROVED 或 APPROVED WITH CONDITIONS 之前，不要运行 `/gate-check`。
```

如果还有未开始的 Should Have stories，将它们与收尾序列一起浮现，以便用户选择：现在关闭 sprint，还是先拉入更多工作。

如果没有更多 stories 准备就绪但 Must Have stories 仍在进行中（非完成）：
"没有更多 stories 可以开始 — [N] 个 Must Have stories 仍在进行中。在 sprint 收尾前继续实现这些。"

---

## 协作协议

- **未经用户批准绝不标记 story 为完成** — Phase 7 在任何文件编辑前需要
  明确的"是"。
- **绝不自动修复失败的标准** — 报告它们并询问要做什么。
- **偏差是事实，不是判断** — 中立地呈现它们；用户
  决定是否可接受。
- **BLOCKED 裁决是建议性的** — 用户可以覆盖并无论如何标记完成；
  如果他们这样做，明确记录风险。
- 使用 `AskUserQuestion` 进行代码审查提示和批量手动
  标准确认。

---

## 推荐的后续步骤

- 运行 `/story-readiness [next-story-path]` 在开始实现前验证下一个 story
- 如果所有 Must Have stories 都已完成：运行 `/smoke-check sprint` → `/team-qa sprint` → `/gate-check`
- 如果记录了 tech debt：通过 `/tech-debt` 跟踪以保持登记册最新
