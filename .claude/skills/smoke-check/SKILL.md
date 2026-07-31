---
name: smoke-check
description: "在交接 QA 之前运行关键路径 smoke test 关卡。执行自动化测试套件，验证核心功能，并生成 PASS/FAIL 报告。在 sprint 的 stories 实现后、手动 QA 开始前运行。smoke check 失败意味着 build 尚未准备好交给 QA。"
argument-hint: "[sprint | quick | --platform pc|console|mobile|all]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write, AskUserQuestion
model: sonnet
---

# Smoke Check

此 skill 是"实现完成"与"准备交接 QA"之间的关卡。它运行自动化测试套件，
检查测试覆盖率缺口，与开发者批量验证关键路径，并生成 PASS/FAIL
报告。

规则很简单：**未通过 smoke check 的 build 不交给 QA。**
将有问题的 build 交给 QA 会浪费他们的时间并打击团队士气。

**输出：** `production/qa/smoke-[date].md`

---

## 解析参数

参数可以组合使用：`/smoke-check sprint --platform console`

**基础模式**（第一个参数，默认：`sprint`）：
- `sprint` — 针对当前 sprint 的 stories 执行完整 smoke check
- `quick` — 跳过覆盖率扫描（Phase 3）和 Batch 3；用于快速复检

**平台标志**（`--platform`，默认：无）：
- `--platform pc` — 添加 PC 专属检查（键盘、鼠标、窗口模式）
- `--platform console` — 添加主机专属检查（gamepad、TV 安全区、
  平台认证要求）
- `--platform mobile` — 添加移动端专属检查（触摸、横屏/竖屏、
  电池/散热行为）
- `--platform all` — 添加所有平台变体；输出每个平台的裁决表

如果提供了 `--platform`，Phase 4 会添加平台专属批次，
Phase 5 除整体裁决外还输出每个平台的裁决表。

---

## Phase 1: 检测测试环境

在运行任何内容之前，先了解环境：

1. **测试框架检查**：验证 `tests/` 目录是否存在。
   如果不存在："No test directory found at `tests/`. Run `/test-setup`
   to scaffold the testing infrastructure, or create the directory manually
   if tests live elsewhere." 然后停止。

2. **CI 检查**：检查 `.github/workflows/` 是否包含引用测试的工作流文件。
   在报告中注明是否配置了 CI。

3. **引擎检测**：读取 `.claude/docs/technical-preferences.md` 并
   提取 `Engine:` 值。存储此值供 Phase 2 选择测试命令使用。

4. **Smoke 测试列表**：检查 `production/qa/smoke-tests.md` 或
   `tests/smoke/` 是否存在。如果找到 smoke 测试列表，加载它供
   Phase 4 使用。如果都不存在，smoke 测试将从当前 QA 计划中抽取
   （Phase 4 回退方案）。

5. **QA 计划检查**：glob `production/qa/qa-plan-*.md` 并取最近修改的文件。
   如果找到，记录路径 — 它将在 Phase 3 和 Phase 4 中使用。如果未找到，
   注明："No QA plan found. Run `/qa-plan sprint` before smoke-checking for best results."

报告发现后再继续："Environment: [engine]. Test directory:
[found / not found]. CI configured: [yes / no]. QA plan: [path / not found]."

---

## Phase 2: 运行自动化测试

尝试通过 Bash 运行测试套件。根据 Phase 1 检测到的引擎选择命令：

**Godot 4:**
```bash
godot --headless --script tests/gdunit4_runner.gd 2>&1
```
如果该路径不存在 GDUnit4 运行脚本，尝试：
```bash
godot --headless -s addons/gdunit4/GdUnitRunner.gd 2>&1
```
如果两个路径都不存在，注明："GDUnit4 runner not found — confirm the runner
path for your test framework."

**Unity:**
Unity 测试需要编辑器，在大多数环境中无法通过 shell 无头运行。检查最近的测试结果产物：
```bash
# List most recent test results (bash) — on Windows PowerShell use the fallback below
ls -t test-results/ 2>/dev/null | head -5 \
  || powershell -Command "Get-ChildItem test-results/ -ErrorAction SilentlyContinue | Sort-Object LastWriteTime -Descending | Select-Object -First 5 -ExpandProperty Name"
```
如果存在测试结果文件（XML 或 JSON），读取最近的一个并解析
PASS/FAIL 计数。如果不存在产物："Unity tests must be run from the
editor or CI pipeline. Please confirm test status manually before proceeding."

**Unreal Engine:**
```bash
# List most recent Unreal automation logs (bash) — on Windows PowerShell use the fallback below
ls -t Saved/Logs/ 2>/dev/null | grep -i "test\|automation" | head -5 \
  || powershell -Command "Get-ChildItem Saved/Logs/ -ErrorAction SilentlyContinue | Where-Object { $_.Name -match 'test|automation' } | Sort-Object LastWriteTime -Descending | Select-Object -First 5 -ExpandProperty Name"
```
如果未找到匹配的日志："UE automation tests must be run via the Session
Frontend or CI pipeline. Please confirm test status manually."

**未知引擎 / 未配置：**
"Engine not configured in `.claude/docs/technical-preferences.md`. Run
`/setup-engine` to specify the engine, then re-run `/smoke-check`."

**如果此环境中测试运行器不可用**（引擎二进制文件不在 PATH 上、
运行脚本未找到等），明确报告：

"Automated tests could not be executed — engine binary not found on PATH.
Status will be recorded as NOT RUN. Confirm test results from your local IDE
or CI pipeline. Unconfirmed NOT RUN is treated as PASS WITH WARNINGS, not
FAIL — the developer must manually confirm results."

不要将 NOT RUN 视为自动 FAIL。将其记录为警告。
开发者在 Phase 4 的手动确认可以解决它。

解析运行器输出并提取：
- 运行测试总数
- 通过数量
- 失败数量
- 任何失败测试的名称（最多 10 个；如果更多，注明数量）
- 运行器本身的任何崩溃或错误输出

---

## Phase 3: 检查测试覆盖率

按优先级顺序从以下来源抽取 story 列表：
1. Phase 1 找到的 QA 计划（其测试摘要表列出了每个 story 的预期测试
   文件路径）
2. `production/sprints/` 中的当前 sprint 计划（最近修改的文件）
3. 如果传入了 `quick` 参数，完全跳过此阶段并注明：
   "Coverage scan skipped — run `/smoke-check sprint` for full coverage
   analysis."

对于范围内的每个 story：

1. 从 story 的文件路径提取系统 slug
   （如 `production/epics/combat/story-001.md` → `combat`）
2. Glob `tests/unit/[system]/` 和 `tests/integration/[system]/` 查找文件名
   包含 story  slug 或密切相关术语的文件
3. 检查 story 文件本身是否有 `Test file:` 头部字段或
   "Test Evidence" 章节

为每个 story 分配覆盖率状态：

| 状态 | 含义 |
|--------|---------|
| **COVERED** | 找到了与此 story 的系统和范围匹配的测试文件 |
| **MANUAL** | Story 类型为 Visual/Feel 或 UI；找到了测试证据文档 |
| **MISSING** | Logic 或 Integration story 没有匹配的测试文件 |
| **EXPECTED** | Config/Data story — 不需要测试文件；抽查即可 |
| **UNKNOWN** | Story 文件缺失或无法读取 |

MISSING 条目是建议性缺口。它们不会导致 FAIL 裁决，但必须在报告中显著显示，
并且必须在 `/story-done` 完全关闭这些 story 之前解决。

---

## Phase 4: 运行手动 Smoke 检查

按优先级顺序从以下来源抽取 smoke 测试检查清单：
1. QA 计划的 "Smoke Test Scope" 章节（如果在 Phase 1 找到了 QA 计划）
2. `production/qa/smoke-tests.md`（如果存在）
3. `tests/smoke/` 目录内容（如果存在）
4. 下面的标准回退列表（仅当以上都不存在时使用）

根据 sprint 或 QA 计划识别的实际系统定制批次 2 和 3。用当前 sprint 的 stories 中的真实机制名称替换括号占位符。

使用 `AskUserQuestion` 进行批量验证。最多 3 次调用。

**Batch 1 — 核心稳定性（始终运行）：**
```
question: "Core stability — select any items that FAILED (leave all unselected if everything passed):"
multiSelect: true
options:
  - "Game does not launch or crashes before reaching the main menu"
  - "New game / session fails to start"
  - "Main menu does not respond to inputs"
  - "Crash or hang observed during basic navigation"
```

对于任何选中的项目，在生成报告前要求用户简要描述失败情况。

**Batch 2 — Sprint 变更和回归（始终运行）：**
```
question: "Sprint changes and regression — select any items that FAILED (leave all unselected if everything passed):"
multiSelect: true
options:
  - "[Primary mechanic this sprint] — FAILED"
  - "[Second notable change this sprint, if any] — FAILED"
  - "Regression in a previous sprint's feature — FAILED"
  - "Other unexpected breakage observed — FAILED"
```

对于任何选中的项目，在生成报告前要求用户简要描述故障情况。

**Batch 3 — 数据完整性和性能（除非传入 `quick` 参数否则运行）：**
```
question: "Data integrity and performance — select any items that FAILED or were skipped (leave all unselected if everything passed):"
multiSelect: true
options:
  - "Save / load — FAILED (data loss or corruption observed)"
  - "Save / load — N/A (save system not yet implemented)"
  - "Frame rate drops or hitches observed — FAILED"
  - "Performance not checked this session"
```

对于任何选中的 FAILED 项目，在生成报告前要求用户描述故障情况。

将每个响应逐字记录供 Phase 5 报告使用。

**平台批次** *（仅在传入了 `--platform` 参数时运行）*：

**PC 平台**（`--platform pc` 或 `--platform all`）：
```
question: "PC Platform — select any items that FAILED (leave all unselected if everything passed):"
multiSelect: true
options:
  - "Keyboard controls — FAILED (describe issue after)"
  - "Mouse input or cursor visibility — FAILED (describe issue after)"
  - "Windowed / fullscreen mode — FAILED (describe issue after)"
  - "Resolution change — FAILED (describe issue after)"
```

对于任何选中的项目，在生成报告前要求用户简要描述失败情况。

**主机平台**（`--platform console` 或 `--platform all`）：
```
question: "Console Platform — select any items that FAILED (leave all unselected if everything passed):"
multiSelect: true
options:
  - "Gamepad input — FAILED (describe issue after)"
  - "UI outside TV safe zone / text clipped — FAILED (describe what is clipped after)"
  - "Keyboard/mouse fallback shown to gamepad user — FAILED (describe after)"
  - "Cold start (no prior save) — FAILED (describe issue after)"
```

对于任何选中的项目，在生成报告前要求用户简要描述失败情况。

**移动平台**（`--platform mobile` 或 `--platform all`）：
```
question: "Mobile Platform — select any items that FAILED (leave all unselected if everything passed):"
multiSelect: true
options:
  - "Touch controls — FAILED (describe issue after)"
  - "Orientation change (portrait ↔ landscape) — FAILED (describe what breaks after)"
  - "Background / foreground transition (home button) — FAILED (describe issue after)"
  - "Performance / thermal throttling on target device — FAILED (describe after)"
```

对于任何选中的项目，在生成报告前要求用户简要描述失败情况。

---

## Phase 5: 生成报告

组装完整的 smoke check 报告：

````markdown
## Smoke Check Report
**Date**: [date]
**Sprint**: [sprint name / number, or "Not identified"]
**Engine**: [engine]
**QA Plan**: [path, or "Not found — run /qa-plan first"]
**Argument**: [sprint | quick | blank]

---

### Automated Tests

**Status**: [PASS ([N] tests, [N] passing) | FAIL ([N] failures) |
NOT RUN ([reason])]

[If FAIL, list failing tests:]
- `[test name]` — [brief failure description from runner output]

[If NOT RUN:]
"Manual confirmation required: did tests pass in your local IDE or CI? This
will determine whether the automated test row contributes to a FAIL verdict."

---

### Test Coverage

| Story | Type | Test File | Coverage Status |
|-------|------|-----------|----------------|
| [title] | Logic | `tests/unit/[system]/[slug]_test.[ext]` | COVERED |
| [title] | Visual/Feel | `tests/evidence/[slug]-screenshots.md` | MANUAL |
| [title] | Logic | — | MISSING ⚠ |
| [title] | Config/Data | — | EXPECTED |

**Summary**: [N] covered, [N] manual, [N] missing, [N] expected.

---

### Manual Smoke Checks

- [x] Game launches without crash — PASS
- [x] New game starts — PASS
- [x] [Core mechanic] — PASS
- [ ] [Other check] — FAIL: [user's description]
- [x] Save / load — PASS
- [-] Performance — not checked this session

---

### 缺失的测试证据

在通过 `/story-done` 标记为 COMPLETE 之前必须有测试证据的 stories：

- **[story title]** (`[path]`) — Logic story 没有测试文件。
  Expected location: `tests/unit/[system]/[story-slug]_test.[ext]`

[If none:] "All Logic and Integration stories have test coverage."

---

### Platform-Specific Results *(only if `--platform` was provided)*

| Platform | Checks Run | Passed | Failed | Platform Verdict |
|----------|-----------|--------|--------|-----------------|
| PC | [N] | [N] | [N] | PASS / FAIL |
| Console | [N] | [N] | [N] | PASS / FAIL |
| Mobile | [N] | [N] | [N] | PASS / FAIL |

**Platform notes**: [any platform-specific observations not captured in pass/fail]

Any platform with one or more FAIL checks contributes to the overall FAIL verdict.

---

### Verdict: [PASS | PASS WITH WARNINGS | FAIL]

[Verdict rules — first matching rule wins:]

**FAIL** 如果满足以下任一条件：
- Automated test suite ran and reported one or more test failures
- Any Batch 1 (core stability) check returned FAIL
- Any Batch 2 (primary sprint mechanic or regression check) returned FAIL

**PASS WITH WARNINGS** 如果满足以下所有条件：
- Automated tests PASS or NOT RUN (developer has not yet confirmed)
- All Batch 1 and Batch 2 smoke checks PASS
- One or more Logic/Integration stories have MISSING test evidence

**PASS** 如果满足以下所有条件：
- Automated tests PASS
- All smoke checks in all batches PASS or N/A
- No MISSING test evidence entries
````

---

## Phase 6: 写入并设闸

在对话中展示完整报告，然后询问：

"May I write this smoke check report to `production/qa/smoke-[date].md`?"

仅在获得批准后写入。

写入后，传达关卡裁决：

**如果裁决为 FAIL：**

"The smoke check failed. Do not hand off to QA until these failures are
resolved:

[List each failing automated test or smoke check with a one-line description]

Fix the failures and run `/smoke-check` again to re-gate before QA hand-off."

**如果裁决为 PASS WITH WARNINGS：**

"Smoke check passed with warnings. The build is ready for manual QA.

Advisory items to resolve before running `/story-done` on affected stories:
[list MISSING test evidence entries]

QA hand-off: share `production/qa/qa-plan-[sprint].md` with the qa-tester
agent to begin manual verification."

**如果裁决为 PASS：**

"Smoke check passed cleanly. The build is ready for manual QA.

QA hand-off: share `production/qa/qa-plan-[sprint].md` with the qa-tester
agent to begin manual verification."

---

## 协作协议

- **绝不将 NOT RUN 视为自动 FAIL** — 将其记录为 NOT RUN 并让
  开发者手动确认状态。未确认的 NOT RUN 归入
  PASS WITH WARNINGS，而非 FAIL。
- **绝不自动修复失败** — 报告它们并说明必须解决的内容。
  不要尝试编辑源代码或测试文件。
- **PASS WITH WARNINGS 不阻塞 QA 交接** — 它记录建议性缺口供
  `/story-done` 跟进。
- **`quick` 参数** 跳过 Phase 3（覆盖率扫描）和 Phase 4 Batch 3。
  用于修复特定失败后的快速复检。
- 使用 `AskUserQuestion` 进行所有手动 smoke 检查验证。
- **绝不未经询问就写入报告** — Phase 6 在创建任何文件前需要明确
  批准。
