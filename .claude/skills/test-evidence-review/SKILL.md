---
name: test-evidence-review
description: "Quality review of test files and manual evidence documents. Goes beyond existence checks — evaluates assertion coverage, edge case handling, naming conventions, and evidence completeness. Produces ADEQUATE/INCOMPLETE/MISSING verdict per story. Run before QA sign-off or on demand."
argument-hint: "[story-path | sprint | system-name]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write
model: sonnet
---

# Test Evidence Review

`/smoke-check` 验证测试文件 **存在** 且 **通过**。此 skill 走得更远 — 它审查这些测试和证据文档的 **质量**。一个存在且通过的测试文件仍可能遗漏关键行为。一个存在的证据文档可能缺少关闭所需的签字。

**输出：** 总结报告（在对话中）+ 可选的 `production/qa/evidence-review-[date].md`

**何时运行：**
- 在 QA 交接签字之前（`/team-qa` 阶段 5）
- 在任何测试质量有疑问的 story 上
- 作为里程碑审查的一部分，用于 Logic 和 Integration story 质量审计

---

## 1. 解析参数

**模式：**
- `/test-evidence-review [story-path]` — 审查单个 story 的证据
- `/test-evidence-review sprint` — 审查当前 sprint 中的所有 story
- `/test-evidence-review [system-name]` — 审查 epic/system 中的所有 story
- 无参数 — 询问范围："Single story"、"Current sprint"、"A system"

---

## 2. 加载范围内的 Story

根据参数：

**单个 story**：直接读取 story 文件。提取：Story Type、Test Evidence 部分、story slug、system name。

**Sprint**：读取 `production/sprints/` 中最近修改的文件。从 sprint 计划中提取 story 文件路径列表。读取每个 story 文件。

**System**：Glob `production/epics/[system-name]/story-*.md`。读取每个。

对于每个 story，收集：
- `Type:` 字段（Logic / Integration / Visual/Feel / UI / Config/Data）
- `## Test Evidence` 部分 — 声明的预期测试文件路径或证据文档
- Story slug（从文件名）
- System name（从目录路径）
- Acceptance Criteria 列表（所有复选框项）

---

## 3. 定位证据文件

对于每个 story，找到证据：

**Logic story**：Glob `tests/unit/[system]/[story-slug]_test.*`
  - 如果未找到，也尝试：在 `tests/unit/[system]/` 中 Grep 包含 story slug 的文件

**Integration story**：Glob `tests/integration/[system]/[story-slug]_test.*`
  - 也检查 `production/session-logs/` 中提及 story 的试玩记录

**Visual/Feel 和 UI story**：Glob `production/qa/evidence/[story-slug]-evidence.*`

**Config/Data story**：Glob `production/qa/smoke-*.md`（任何烟雾测试报告）

记录每个 story 发现了什么（路径）或未发现什么（缺口）。

---

## 4. 审查自动化测试质量（Logic / Integration）

对于每个找到的测试文件，读取并评估：

### 断言覆盖

计算不同断言的数量（包含 assert、expect、check、verify 或引擎特定断言模式的行）。低断言数量是一个质量信号 — 每个测试函数只有 1 个断言的测试可能无法覆盖预期行为的范围。

阈值：
- **每个测试函数 3+ 断言** → 正常
- **每个测试函数 1-2 断言** → 注明可能较薄弱
- **0 断言**（测试存在但无断言） → 标记为 BLOCKING — 测试空洞通过，不证明任何事

### 边缘情况覆盖

对于 story 中包含数字、阈值或 "when X happens" 条件每个验收标准：检查测试函数名或测试体是否引用该特定情况。

启发式方法：
- Grep 测试文件中的 "zero"、"max"、"null"、"empty"、"min"、"invalid"、"boundary"、"edge" — 存在任何一个是积极信号
- 如果 story 有带特定边界的 Formulas 部分：检查测试是否在最小/最大值处执行

### 命名质量

测试函数名应描述：场景 + 预期结果。模式：`test_[scenario]_[expected_outcome]`

将通用命名的函数（`test_1`、`test_run`、`testBasic`）标记为 **命名问题** — 它们使失败更难诊断。

### 公式可追溯性

对于 GDD 有 Formulas 部分的 Logic story：检查测试文件是否包含至少一个名称或注释引用公式名称或公式值的测试。按公式名称执行测试的维护成本在公式变更时更高。

---

## 5. 审查手动证据质量（Visual/Feel / UI）

对于每个找到的证据文档，读取并评估：

### 标准关联

证据文档应引用 story 中的每个验收标准。检查：证据文档是否包含每个标准（或清晰的重新表述）？缺失的标准意味着该标准从未被验证。

### 签字完整性

检查三行签字（或等效字段）：
- 开发者签字
- 设计师 / 美术负责人签字（对于 Visual/Feel）
- QA 负责人签字

如果有任何缺失或空白：标记为 INCOMPLETE — 没有所有必需签字，story 无法完全关闭。

### 截图 / 产物完整性

对于 Visual/Feel story：检查证据文档中是否引用了截图文件路径。如果引用了，Glob 查找它们以确认存在。

对于 UI story：检查是否存在演练序列（逐步交互日志）。

### 日期覆盖

证据文档应有日期。如果日期早于 story 的最后重大变更（启发式方法：与 sprint 计划中的 sprint 开始日期比较），标记为 POTENTIALLY STALE — 证据可能不覆盖最终实现。

---

## 6. 构建审查报告

对于每个 story，分配一个结论：

| 结论 | 含义 |
|---------|---------|
| **ADEQUATE** | 测试/证据存在，通过质量检查，所有标准已覆盖 |
| **INCOMPLETE** | 测试/证据存在但存在质量缺口（薄弱断言、缺失签字） |
| **MISSING** | 对于需要它的 story 类型，未找到测试或证据 |

整个 sprint/system 的结论是最差的 story 结论。

```markdown
## Test Evidence Review

> **Date**: [date]
> **Scope**: [single story path | Sprint [N] | [system name]]
> **Stories reviewed**: [N]
> **Overall verdict**: ADEQUATE / INCOMPLETE / MISSING

---

### Story-by-Story Results

#### [Story Title] — [Type] — [ADEQUATE/INCOMPLETE/MISSING]

**Test/evidence path**: `[path]` (found) / (not found)

**Automated test quality** *(Logic/Integration only)*:
- Assertion coverage: [N per function on average] — [adequate / thin / none]
- Edge cases: [covered / partial / not found]
- Naming: [consistent / [N] generic names flagged]
- Formula traceability: [yes / no — formula names not referenced in tests]

**Manual evidence quality** *(Visual/Feel/UI only)*:
- Criterion linkage: [N/M criteria referenced]
- Sign-offs: [Developer ✓ | Designer ✗ | QA Lead ✗]
- Artefacts: [screenshots present / missing / N/A]
- Freshness: [dated [date] — current / potentially stale]

**Issues**:
- BLOCKING: [description] *(prevents story-done)*
- ADVISORY: [description] *(should fix before release)*

---

### Summary

| Story | Type | Verdict | Issues |
|-------|------|---------|--------|
| [title] | Logic | ADEQUATE | None |
| [title] | Integration | INCOMPLETE | Thin assertions (avg 1.2/function) |
| [title] | Visual/Feel | INCOMPLETE | QA lead sign-off missing |
| [title] | Logic | MISSING | No test file found |

**BLOCKING items** (must resolve before story can be closed): [N]
**ADVISORY items** (should address before release): [N]
```

---

## 7. 写入输出（可选）

在对话中展示报告。

询问："May I write this test evidence review to `production/qa/evidence-review-[date].md`?"

这是可选的 — 报告本身就有用。仅在用户想要持久记录时写入。

报告之后：

- 对于 BLOCKING 项："These must be resolved before `/story-done` can mark the story Complete. Would you like to address any of them now?"
- 对于薄弱断言："Consider running `/test-helpers [system]` to see scaffolded assertion patterns for common cases."
- 对于缺失签字："Manual sign-off is required from [role]. Share `[evidence-path]` with them to complete sign-off."

结论：**COMPLETE** — 证据审查完成。如果发现 BLOCKING 项则使用 CONCERNS。

---

## 协作协议

- **报告质量问题，不修复它们** — 此 skill 读取和评估；它不修改测试文件或证据文档
- **ADEQUATE 意味着足以发布，而非完美** — 避免对功能正常且足够全面的测试吹毛求疵
- **BLOCKING 与 ADVISORY 的区别很重要** — 仅在缺口真正导致 story 标准未验证时才标记 BLOCKING
- **写入前先询问** — 报告文件是可选的；写入前始终确认
