---
name: team-qa
description: "编排 QA 团队完成完整测试周期。协调 qa-lead（策略 + 测试计划）和 qa-tester（测试用例编写 + bug 报告），为 sprint 或功能生成完整的 QA 包。涵盖：测试计划生成、测试用例编写、smoke check 关卡、手动 QA 执行和签字报告。"
argument-hint: "[sprint | feature: system-name] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Task, AskUserQuestion
model: sonnet
agent: qa-lead
---

当此 skill 被调用时，通过结构化测试周期编排 QA 团队。

**决策点：** 在每个阶段转换时，使用 `AskUserQuestion` 向用户
展示子 agent 的提案作为可选项。在对话中写入 agent 的
完整分析，然后用简洁标签捕获决策。
用户必须批准才能进入下一阶段。

## Phase 0: 解析 Review 模式

1. 如果传入了 `--review [mode]` 作为参数，使用该模式。
2. 否则读取 `production/review-mode.txt` — 使用那里写的内容。
3. 否则默认为 `lean`。

模式：
- `full` — 按所述生成所有 director 和 lead gates
- `lean` — 跳过 director gates，除非它们是 PHASE-GATE 类型（CD-PHASE-GATE, TD-PHASE-GATE, PR-PHASE-GATE, AD-PHASE-GATE）
- `solo` — 完全跳过所有 director gate 生成；在没有任何 agent gates 的情况下运行 skill

存储解析后的模式供所有后续阶段使用。

## 团队组成

- **qa-lead** — QA 策略、测试计划生成、story 分类、签字报告
- **qa-tester** — 测试用例编写、bug 报告编写、手动 QA 文档

## 如何委托

使用 Task 工具将每个团队成员生成为子 agent：
- `subagent_type: qa-lead` — 策略、规划、分类、签字
- `subagent_type: qa-tester` — 测试用例编写和 bug 报告编写

始终在每个 agent 的提示中提供完整上下文（story 文件路径、QA 计划路径、范围约束）。在可能的地方并行启动独立的 qa-tester 任务（如 Phase 5 中的多个 stories 可以同时搭建）。

## 管线

### Phase 1: 加载上下文

在做任何其他事情之前，收集完整范围：

1. 从参数检测当前 sprint 或功能范围：
   - 如果参数是 sprint 标识符（如 `sprint-03`）：Glob `production/sprints/` 查找匹配 `*[sprint-identifier]*.md` 的文件。读取匹配的文件。如果多个匹配，使用最近修改的。
   - 如果参数是 `feature: [system-name]`：glob 标记为该系统的 story 文件
   - 如果没有参数：读取 `production/session-state/active.md` 和 `production/sprint-status.yaml`（如存在）以推断活动 sprint

2. 读取 `production/stage.txt` 确认当前项目阶段。

3. 统计找到的 stories 并向用户报告：
   > "QA cycle starting for [sprint/feature]. Found [N] stories. Current stage: [stage]. Ready to begin QA strategy?"

### Phase 2: QA 策略（qa-lead）

通过 Task 生成 `qa-lead` 来审查所有范围内的 stories 并生成 QA 策略。

提示 qa-lead：
- 读取每个 story 文件
- 按类型分类每个 story：**Logic** / **Integration** / **Visual/Feel** / **UI** / **Config/Data**
- 识别哪些 stories 需要自动化测试证据 vs. 手动 QA
- 标记任何缺少验收标准或缺少测试证据的 stories（这些会阻塞 QA）
- 估算手动 QA 工作量（需要的测试会话数）
- **在评估 smoke 状态之前，检查是否存在 smoke check 报告**：Glob `production/qa/smoke-*.md` 并读取最近修改的文件（如果找到）。如果报告存在，直接使用其裁决和发现 — 不要重新询问用户。如果报告不存在，注明："No prior smoke check report found — run `/smoke-check sprint` before proceeding." 并将 smoke check 状态设置为 UNKNOWN（视为 PASS WITH WARNINGS 以便继续）。生成 smoke check 裁决：**PASS** / **PASS WITH WARNINGS [list]** / **FAIL [list of failures]** / **UNKNOWN (no report found)**
- 生成策略摘要表和 smoke check 结果：

  | Story | Type | Automated Required | Manual Required | Blocker? |
  |-------|------|--------------------|-----------------|----------|

  **Smoke Check**: [PASS / PASS WITH WARNINGS / FAIL / UNKNOWN] — [来源：`production/qa/smoke-[date].md` 或 "no report found"] — [如非 PASS 的详情]

如果 smoke check 结果为 **FAIL**，qa-lead 必须突出列出失败项。QA 无法在 smoke check 失败的情况下通过策略阶段。

向用户展示 qa-lead 的完整策略，然后使用 `AskUserQuestion`：

```
question: "QA Strategy Review"
options:
  - "Looks good — proceed to test plan"
  - "Adjust story types before proceeding"
  - "Skip blocked stories and proceed with the rest"
  - "Smoke check failed — fix issues and re-run /team-qa"
  - "Cancel — resolve blockers first"
```

如果 smoke check **FAIL**：不要继续到 Phase 3。展示 smoke check 报告中的失败项并停止。用户必须修复它们，重新运行 `/smoke-check sprint`，然后重新运行 `/team-qa`。
如果 smoke check **UNKNOWN**：展示警告 — "No smoke check report found. Recommend running `/smoke-check sprint` before QA. Proceeding with caution."
如果 smoke check **PASS WITH WARNINGS**：为签字报告记录警告并继续。
如果存在阻塞：明确列出它们。用户可以选择跳过被阻塞的 stories 或取消周期。

### Phase 3: 测试计划生成

使用 Phase 2 的策略，生成结构化测试计划文档。

测试计划应涵盖：
- **范围**：sprint/功能名称、story 数量、日期
- **Story 分类表**：来自 Phase 2 策略
- **自动化测试要求**：哪些 stories 需要测试文件、`tests/` 中的预期路径
- **手动 QA 范围**：哪些 stories 需要手动演练以及验证什么
- **范围外**：此周期明确不测试的内容及原因
- **进入标准**：QA 开始前必须满足的条件。始终包括：(1) Smoke check PASS 或 PASS WITH WARNINGS 报告存在于 `production/qa/smoke-*.md`，(2) 构建稳定（启动时无崩溃），(3) 所有 Must Have stories 在 `production/sprint-status.yaml` 中的 Status 为 in-progress 或 done。除此之外添加任何 sprint 专属标准。
- **退出标准**：什么构成完成的 QA 周期（所有 stories PASS 或 FAIL 且已提交 bug）

询问："May I write the QA plan to `production/qa/qa-plan-[sprint]-[date].md`?"

仅在收到批准后写入。

### Phase 4: 测试用例编写（qa-tester）

> **Smoke check** 作为 Phase 2（QA 策略）的一部分执行。如果 smoke check 在 Phase 2 返回 FAIL，周期在那里停止。此阶段仅在 Phase 2 smoke check 为 PASS、PASS WITH WARNINGS 或 UNKNOWN 时运行。

对于每个需要手动 QA 的 story（Visual/Feel、UI、无自动化测试的 Integration）：

通过 Task 为每个 story 生成 `qa-tester`（在可能的地方并行运行），提供：
- story 文件路径
- QA 计划中该 story 的相关章节
- 被测系统的 GDD 验收标准（如可用）
- 编写覆盖所有验收标准的详细测试用例的指令

每个测试用例集应包括：
- **前置条件**：测试开始前需要的游戏状态
- **步骤**：编号的、明确的操作
- **预期结果**：应该发生什么
- **实际结果**：留空供测试者填写
- **通过/失败**：留空

在执行前向用户展示测试用例供审查。按 story 分组。

使用 `AskUserQuestion` 按 story 分组（每次批量 3-4 个）：

```
question: "Test cases ready for [Story Group]. Review before manual QA begins?"
options:
  - "Approved — begin manual QA for these stories"
  - "Revise test cases for [story name]"
  - "Skip manual QA for [story name] — not ready"
```

### Phase 5: 手动 QA 执行

遍历批准的手动 QA 列表中的每个 story。

将 stories 分成 3-4 个一组，对每组使用 `AskUserQuestion`：

```
question: "Manual QA — [Story Title]\n[brief description of what to test]"
options:
  - "PASS — all acceptance criteria verified"
  - "PASS WITH NOTES — minor issues found (describe after)"
  - "FAIL — criteria not met (describe after)"
  - "BLOCKED — cannot test yet (reason)"
```

每次 FAIL 结果后：使用 `AskUserQuestion` 收集失败描述，然后通过 Task 生成 `qa-tester` 在 `production/qa/bugs/` 中编写正式的 bug 报告。

Bug 报告命名：`BUG-[NNN]-[short-slug].md`（从目录中现有 bug 递增 NNN）。

收集所有结果后，汇总：
- Stories PASS：[count]
- Stories PASS WITH NOTES：[count]
- Stories FAIL：[count] — bugs filed：[IDs]
- Stories BLOCKED：[count]

### Phase 6: QA 签字报告

通过 Task 生成 `qa-lead` 使用 Phase 4-6 的所有结果生成签字报告。

签字报告格式：

```markdown
## QA Sign-Off Report: [Sprint/Feature]
**Date**: [date]

### Test Coverage Summary
| Story | Type | Auto Test | Manual QA | Result |
|-------|------|-----------|-----------|--------|
| [title] | Logic | PASS | — | PASS |
| [title] | Visual | — | PASS | PASS |

### Bugs Found
| ID | Story | Severity | Status |
|----|-------|----------|--------|
| BUG-001 | [story] | S2 | Open |

### Verdict: APPROVED / APPROVED WITH CONDITIONS / NOT APPROVED

**Conditions**（如有）：[列出构建推进前必须修复的内容]

### Next Step
[基于裁决的指导]
```

裁决规则：
- **APPROVED**：所有 stories PASS 或 PASS WITH NOTES；无 S1/S2 bug 未解决
- **APPROVED WITH CONDITIONS**：S3/S4 bug 未解决，或记录了 PASS WITH NOTES 问题；无 S1/S2 bug
- **NOT APPROVED**：任何 S1/S2 bug 未解决；或 stories FAIL 且无记录的变通方案

按裁决的后续步骤指导：
- APPROVED："Build is ready for the next phase. Run `/gate-check` to validate advancement."
- APPROVED WITH CONDITIONS："Resolve conditions before advancing. S3/S4 bugs may be deferred to polish."
- NOT APPROVED："Resolve S1/S2 bugs and re-run `/team-qa` or targeted manual QA before advancing."

询问："May I write this QA sign-off report to `production/qa/qa-signoff-[sprint]-[date].md`?"

仅在收到批准后写入。

## 错误恢复协议

如果任何生成的 agent（通过 Task）返回 BLOCKED、错误或无法完成：

1. **立即展示**：在继续到依赖阶段之前向用户报告"[AgentName]: BLOCKED — [原因]"
2. **评估依赖关系**：检查被阻塞 agent 的输出是否是后续阶段所需的。如果是，未经用户输入不要超过该依赖点。
3. **通过 AskUserQuestion 提供选项**，选项为：
   - 跳过此 agent 并在最终报告中注明缺口
   - 以更窄的范围重试
   - 在此停止并首先解决阻塞
4. **始终生成部分报告** — 输出任何已完成的内容。绝不因为一个 agent 阻塞而丢弃工作。

常见阻塞：
- 输入文件缺失（story 未找到、GDD 缺失） → 重定向到创建它的 skill
- ADR 状态为 Proposed → 不要实现；先运行 `/architecture-decision`
- 范围太大 → 通过 `/create-stories` 拆分为两个 stories
- ADR 和 story 之间的冲突指令 → 展示冲突，不要猜测

## 输出

涵盖以下内容的摘要：范围内的 stories、smoke check 结果、手动 QA 结果、提交的 bug（含 ID 和严重程度），以及最终的 APPROVED / APPROVED WITH CONDITIONS / NOT APPROVED 裁决。

裁决：**COMPLETE** — QA 周期已完成。
裁决：**BLOCKED** — smoke check 失败或关键阻塞阻止了周期完成；已生成部分报告。

## 会话状态更新

最终阶段完成后（签字报告已写入或达到 BLOCKED 裁决），静默追加到 `production/session-state/active.md`：

```
<!-- QA RUN: [date] | Sprint: [sprint identifier or "ad-hoc"] | Verdict: [PASS/FAIL/CONCERNS] | Report: production/qa/qa-[date].md -->
```
