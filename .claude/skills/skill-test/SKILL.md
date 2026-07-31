---
name: skill-test
description: "Validate skill files for structural compliance and behavioral correctness. Three modes: static (linter), spec (behavioral), audit (coverage report)."
argument-hint: "static [skill-name | all] | spec [skill-name] | category [skill-name | all] | audit"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write
model: sonnet
---

# Skill Test

验证 `.claude/skills/*/SKILL.md` 文件的结构合规性与行为正确性。无外部依赖 — 完全在现有的 skill/hook/template 架构内运行。

**四种模式：**

| 模式 | 命令 | 用途 | Token 成本 |
|------|---------|---------|------------|
| `static` | `/skill-test static [name\|all]` | 结构检查器 — 每个 skill 执行 7 项合规检查 | 低 (~1k/skill) |
| `spec` | `/skill-test spec [name]` | 行为验证器 — 评估测试规格中的断言 | 中 (~5k/skill) |
| `category` | `/skill-test category [name\|all]` | 分类评分表 — 根据 skill 所属分类的专属指标进行检查 | 低 (~2k/skill) |
| `audit` | `/skill-test audit` | 覆盖率报告 — skill、agent 规格、最后测试日期 | 低 (~3k 总计) |

---

## Phase 1: 解析参数

从第一个参数确定模式：

- `static [name]` → 对单个 skill 运行 7 项结构检查
- `static all` → 对所有 skill 运行 7 项结构检查（Glob `.claude/skills/*/SKILL.md`）
- `spec [name]` → 读取 skill + 测试规格，评估断言
- `category [name]` → 从 `CCGS Skill Testing Framework/quality-rubric.md` 运行分类专属评分表
- `category all` → 对 catalog 中所有带 `category:` 字段的 skill 运行分类评分表
- `audit`（或无参数）→ 读取 catalog，列出所有 skill 和 agent，显示覆盖率

如果参数缺失或无法识别，输出用法说明并停止。

---

## Phase 2A: Static 模式 — 结构检查器

对每个被测 skill，完整读取其 `SKILL.md` 并运行全部 7 项检查：

### 检查 1 — 必需的 Frontmatter 字段
文件必须在 YAML frontmatter 块中包含以下所有字段：
- `name:`
- `description:`
- `argument-hint:`
- `user-invocable:`
- `allowed-tools:`

**FAIL** 如果缺少任一字段。

### 检查 2 — 多个阶段
skill 必须具有 ≥2 个编号的阶段标题。查找模式如：
- `## Phase N` 或 `## Phase N:`
- `## N.`（编号的顶级章节）
- 如果阶段未明确编号，则至少 2 个不同的 `##` 标题

**FAIL** 如果找到的阶段标题少于 2 个。

### 检查 3 — 裁决关键词
skill 必须包含以下至少一个关键词：`PASS`、`FAIL`、`CONCERNS`、`APPROVED`、
`BLOCKED`、`COMPLETE`、`READY`、`COMPLIANT`、`NON-COMPLIANT`

**FAIL** 如果都不存在。

### 检查 4 — 协作协议语言
skill 必须包含"写入前询问"的语言。查找：
- `"May I write"`（规范形式）
- `"before writing"` 或 `"approval"` 靠近文件写入指令
- `"ask"` + `"write"` 彼此靠近（在同一章节内）

**WARN** 如果缺失（某些只读 skill 可合理跳过此项）。
**FAIL** 如果 `allowed-tools` 包含 `Write` 或 `Edit` 但未找到写入前询问语言。

### 检查 5 — 下一步交接
skill 末尾必须包含推荐的后续操作或跟进路径。查找：
- 提及另一个 skill 的最终章节（如 `/story-done`、`/gate-check`）
- "Recommended next" 或 "next step" 措辞
- "Follow-Up" 或 "After this" 章节

**WARN** 如果缺失。

### 检查 6 — Fork 上下文复杂度
如果 frontmatter 包含 `context: fork`，skill 应有 ≥5 个阶段标题
（`##` 级别或编号的 Phase N 标题）。Fork 上下文用于复杂的多阶段
skill；简单 skill 不应使用它。

**WARN** 如果设置了 `context: fork` 但找到的阶段少于 5 个。

### 检查 7 — 参数提示合理性
`argument-hint` 必须非空。如果 skill 正文提到多种模式
（如 "Mode A | Mode B"），提示应反映它们。将提示与第一阶段的"Parse Arguments"部分进行交叉核对。

**WARN** 如果提示为 `""` 或记录的模式与提示不匹配。

---

### Static 模式输出格式

对于单个 skill：
```
=== Skill Static Check: /[name] ===

Check 1 — Frontmatter Fields:    PASS
Check 2 — Multiple Phases:       PASS (7 phases found)
Check 3 — Verdict Keywords:      PASS (PASS, FAIL, CONCERNS)
Check 4 — Collaborative Protocol: PASS ("May I write" found)
Check 5 — Next-Step Handoff:     WARN (no follow-up section found)
Check 6 — Fork Context Complexity: PASS (8 phases, context: fork set)
Check 7 — Argument Hint:         PASS

Verdict: WARNINGS (1 warning, 0 failures)
Recommended: Add a "Follow-Up Actions" section at the end of the skill.
```

对于 `static all`，生成汇总表，然后列出所有不合规的 skill：
```
=== Skill Static Check: All 52 Skills ===

Skill                  | Result       | Issues
-----------------------|--------------|-------
gate-check             | COMPLIANT    |
design-review          | COMPLIANT    |
story-readiness        | WARNINGS     | Check 5: no handoff
...

Summary: 48 COMPLIANT, 3 WARNINGS, 1 NON-COMPLIANT
Aggregate Verdict: N WARNINGS / N FAILURES
```

---

## Phase 2B: Spec 模式 — 行为验证器

### 步骤 1 — 定位文件

在 `.claude/skills/[name]/SKILL.md` 查找 skill。
从 `CCGS Skill Testing Framework/catalog.yaml` 查找规格路径 — 使用匹配 skill 条目的
`spec:` 字段。

如果任一文件缺失：
- 缺失 skill："Skill '[name]' not found in `.claude/skills/`."
- catalog 中缺失规格路径："No spec path set for '[name]' in catalog.yaml."
- 路径处未找到规格文件："Spec file missing at [path]. Run `/skill-test audit`
   to see coverage gaps."

### 步骤 2 — 读取两个文件

完整读取 skill 文件和测试规格文件。

### 步骤 3 — 评估断言

对于规格中的每个 **Test Case**：

1. 读取 **Fixture** 描述（项目文件的假设状态）
2. 读取 **Expected behavior** 步骤
3. 读取每个 **Assertion** 复选框

对于每个断言，评估 skill 的书面指令在给定 fixture 状态下是否正确执行时能满足它。这是一个
Claude 评估的推理检查，而非代码执行。

标记每个断言：
- **PASS** — skill 指令明确满足此断言
- **PARTIAL** — skill 指令部分满足，但存在歧义
- **FAIL** — skill 指令在给定 fixture 状态下无法满足此断言

对于 **Protocol Compliance** 断言（始终存在）：
- 检查 skill 是否在文件写入前要求 "May I write"
- 检查 skill 是否在请求审批前展示发现
- 检查 skill 是否以推荐的下一步结束
- 检查 skill 是否避免未经审批自动创建文件

### 步骤 4 — 生成报告

```
=== Skill Spec Test: /[name] ===
Date: [date]
Spec: CCGS Skill Testing Framework/skills/[category]/[name].md

Case 1: [Happy Path — name]
  Fixture: [summary]
  Assertions:
    [PASS] [assertion text]
    [FAIL] [assertion text]
       Reason: The skill's Phase 3 says "..." but the fixture state means "..."
  Case Verdict: FAIL

Case 2: [Edge Case — name]
  ...
  Case Verdict: PASS

Protocol Compliance:
  [PASS] Uses "May I write" before file writes
  [PASS] Presents findings before asking approval
  [WARN] No explicit next-step handoff at end

Overall Verdict: FAIL (1 case failed, 1 warning)
```

### 步骤 5 — 提供写入结果

"May I write these results to `CCGS Skill Testing Framework/results/skill-test-spec-[name]-[date].md`
and update `CCGS Skill Testing Framework/catalog.yaml`?"

如果同意：
- 将结果文件写入 `CCGS Skill Testing Framework/results/`
- 更新 `CCGS Skill Testing Framework/catalog.yaml` 中该 skill 的条目：
  - `last_spec: [date]`
  - `last_spec_result: PASS|PARTIAL|FAIL`

---

## Phase 2D: Category 模式 — 评分表评估

### 步骤 1 — 定位 skill 和分类

在 `.claude/skills/[name]/SKILL.md` 查找 skill。
在 `CCGS Skill Testing Framework/catalog.yaml` 中查找 `category:` 字段。

如果未找到 skill："Skill '[name]' not found."
如果没有 `category:` 字段："No category assigned for '[name]' in catalog.yaml.
Add `category: [name]` to the skill entry first."

对于 `category all`：收集所有带 `category:` 字段的 skill 并逐个处理。
`category: utility` 的 skill 仅根据 U1（静态检查通过）和 U2（如适用，gate 模式正确）进行评估 — 对 U1 跳转到 static 模式。

### 步骤 2 — 读取评分表章节

读取 `CCGS Skill Testing Framework/quality-rubric.md`。
提取与 skill 分类匹配的章节（如 `### gate`、`### team`）。

### 步骤 3 — 读取 skill

完整读取 skill 的 `SKILL.md`。

### 步骤 4 — 评估评分表指标

对于分类评分表中的每个指标：
1. 检查 skill 的书面指令是否明确满足该标准
2. 标记 PASS、FAIL 或 WARN
3. 对于 FAIL/WARN，指出 skill 文本中的确切差距（引用相关章节
   或注明其缺失）

### 步骤 5 — 输出报告

```
=== Skill Category Check: /[name] ([category]) ===

Metric G1 — Review mode read:      PASS
Metric G2 — Full mode directors:   FAIL
  Gap: Phase 3 spawns only CD-PHASE-GATE; TD-PHASE-GATE, PR-PHASE-GATE, AD-PHASE-GATE absent
Metric G3 — Lean mode: PHASE-GATE only: PASS
Metric G4 — Solo mode: no directors:    PASS
Metric G5 — No auto-advance:       PASS

Verdict: FAIL (1 failure, 0 warnings)
Fix: Add TD-PHASE-GATE, PR-PHASE-GATE, and AD-PHASE-GATE to the full-mode director
     panel in Phase 3.
```

### 步骤 6 — 提供更新 catalog

"May I update `CCGS Skill Testing Framework/catalog.yaml` to record this category check
(`last_category`, `last_category_result`) for [name]?"

---

## Phase 2C: Audit 模式 — 覆盖率报告

### 步骤 1 — 读取 catalog

读取 `CCGS Skill Testing Framework/catalog.yaml`。如果缺失，注明 catalog 尚不存在
（首次运行状态）。

### 步骤 2 — 枚举所有 skill 和 agent

Glob `.claude/skills/*/SKILL.md` 获取完整的 skill 列表。
从每个路径提取 skill 名称（目录名）。

同时从 `CCGS Skill Testing Framework/catalog.yaml` 读取 `agents:` 部分以获取完整的
agent 列表。

### 步骤 3 — 构建 skill 覆盖率表

对于每个 skill：
- 检查规格文件是否存在（使用 catalog 中的 `spec:` 路径，或 glob `CCGS Skill Testing Framework/skills/*/[name].md`）
- 从 catalog 查找 `last_static`、`last_static_result`、`last_spec`、`last_spec_result`、
  `last_category`、`last_category_result`、`category`（如果在 catalog 中标记为
  "never" / "—"）
- 优先级来自 catalog 的 `priority:` 字段（critical/high/medium/low）

### 步骤 3b — 构建 agent 覆盖率表

对于 catalog `agents:` 部分中的每个 agent：
- 检查规格文件是否存在（使用 catalog 中的 `spec:` 路径，或 glob `CCGS Skill Testing Framework/agents/*/[name].md`）
- 从 catalog 查找 `last_spec`、`last_spec_result`、`category`

### 步骤 4 — 输出报告

```
=== Skill Test Coverage Audit ===
Date: [date]

SKILLS (72 total)
Specs written: 72 (100%) | Never static tested: 72 | Never category tested: 72

Skill                  | Cat      | Has Spec | Last Static | S.Result | Last Cat | C.Result | Priority
-----------------------|----------|----------|-------------|----------|----------|----------|----------
gate-check             | gate     | YES      | never       | —        | never    | —        | critical
design-review          | review   | YES      | never       | —        | never    | —        | critical
...

AGENTS (49 total)
Agent specs written: 49 (100%)

Agent                  | Category   | Has Spec | Last Spec   | Result
-----------------------|------------|----------|-------------|--------
creative-director      | director   | YES      | never       | —
technical-director     | director   | YES      | never       | —
...

Top 5 Priority Gaps (skills with no spec, critical/high priority):
(none if all specs are written)

Skill coverage:  72/72 specs (100%)
Agent coverage:  49/49 specs (100%)
```

audit 模式下不写入文件。

提供建议："Would you like to run `/skill-test static all` to check structural
compliance across all skills? `/skill-test category all` to run category rubric
checks? Or `/skill-test spec [name]` to run a specific behavioral test?"

---

## Phase 3: 推荐的后续步骤

任何模式完成后，提供上下文相关的跟进建议：

- `static [name]` 之后："Run `/skill-test spec [name]` to validate behavioral
  correctness if a test spec exists."
- `static all` 有失败时："Address NON-COMPLIANT skills first. Run
  `/skill-test static [name]` individually for detailed remediation guidance."
- `spec [name]` PASS 之后："Update `CCGS Skill Testing Framework/catalog.yaml` to record this
  pass date. Consider running `/skill-test audit` to find the next spec gap."
- `spec [name]` FAIL 之后："Review the failing assertions and update the skill
  or the test spec to resolve the mismatch."
- `audit` 之后："Start with the critical-priority gaps. Use the spec template
  at `CCGS Skill Testing Framework/templates/skill-test-spec.md` to create new specs."
