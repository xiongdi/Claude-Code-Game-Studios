---
name: test-flakiness
description: "Detect non-deterministic (flaky) tests by reading CI run logs or test result history. Aggregates pass rates per test, identifies intermittent failures, recommends quarantine or fix, and maintains a flaky test registry. Best run during Polish phase or after multiple CI runs."
argument-hint: "[ci-log-path | scan | registry]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash
model: sonnet
---

# Test Flakiness Detection

不稳定测试（flaky test）是指有时通过有时失败的测试，而代码没有任何变更。在某些方面，不稳定测试比没有测试更糟 — 它们训练团队忽略红色 CI 运行，掩盖真正的失败。此 skill 识别它们，解释可能原因，并推荐是隔离还是修复每个测试。

**输出：** 更新 `tests/regression-suite.md` 隔离部分 + 可选的 `production/qa/flakiness-report-[date].md`

**何时运行：**
- Polish 阶段（测试已运行多次；统计信号可靠）
- 当开发者开始将 CI 失败视为"可能不稳定"时
- 在 `/regression-suite` 识别出需要诊断的已隔离测试后

---

## 1. 解析参数

**模式：**
- `/test-flakiness [ci-log-path]` — 分析特定的 CI 运行日志文件
- `/test-flakiness scan` — 扫描 `.github/` 或标准日志输出目录中的所有可用 CI 日志
- `/test-flakiness registry` — 读取现有 regression-suite.md 隔离部分，并为已知的不稳定测试提供修复指导
- 无参数 — 自动检测：如果 CI 日志可访问则运行 `scan`，否则 `registry`

---

## 2. 定位 CI 日志数据

### 选项 A — GitHub Actions（首选）

检查测试结果产物：
```bash
ls -t .github/ 2>/dev/null
ls -t test-results/ 2>/dev/null
```

对于 Godot 项目：GdUnit4 输出与 JUnit 格式兼容的 XML 结果。检查 `test-results/` 中的 `.xml` 文件。

对于 Unity 项目：game-ci 测试运行器默认将 NUnit XML 输出到 `test-results/`。

对于 Unreal 项目：自动化日志进入 `Saved/Logs/`。Grep 查找 `Result: Success` 和 `Result: Fail` 模式。

### 选项 B — 本地日志文件

如果提供了路径参数，直接读取该文件。

### 选项 C — 无日志数据可用

如果未找到日志：
> "No CI log data found. To detect flaky tests, this skill needs test result
> history from multiple runs. Options:
> 1. Run the test suite at least 3 times and collect the output logs
> 2. Check CI pipeline output and save a log to `test-results/`
> 3. Run `/test-flakiness registry` to review tests already flagged as flaky
>    in `tests/regression-suite.md`"

停止并询问用户选择哪个选项。

---

## 3. 解析测试结果

对于每个找到的 CI 日志或结果文件，解析：

**JUnit XML 格式**（GdUnit4 / Unity）：
- Grep 查找 `<testcase name=` 获取测试名称
- Grep 查找 `<failure` 或 `<error` 识别失败
- 解析 `classname` 和 `name` 属性获取完整测试标识符

**纯文本日志**：
- Grep 查找通过/失败模式：
  - Godot：`PASSED` / `FAILED` 与测试名称相邻
  - Unreal：`Result: Success` / `Result: Fail`
  - Unity：`Test passed` / `Test failed`

构建表格：`test_id → [run1_result, run2_result, run3_result, ...]`

---

## 4. 识别不稳定测试

如果测试在结果历史中同时出现 PASS 和 FAIL 结果，且它们之间没有代码变更，则该测试是 **不稳定的**。

不稳定阈值：
- **高不稳定性**：在 >25% 的运行中失败 — 立即隔离
- **中等不稳定性**：在 5-25% 的运行中失败 — 调查并尽快修复
- **低/疑似不稳定性**：在 1-5% 的运行中失败 — 监控；可能是真正罕见的失败

对于每个不稳定测试，分类可能原因：

### 原因分类

| 原因 | 症状 | 修复方向 |
|-------|----------|---------------|
| **Timing / async** | 在等待信号或计时器后失败；通过率与系统负载相关 | 添加显式 await/同步；避免基于时间的延迟 |
| **Order dependency** | 在特定其他测试之后运行时失败；隔离时通过 | 添加适当的 setup/teardown；确保测试隔离 |
| **Random seed** | 无模式地间歇性失败；涉及 RNG | 传递显式种子；不要在测试中使用 `randf()` |
| **Resource leak** | 在测试运行后期更频繁失败 | 在 teardown 中修复清理；检查孤立节点（Godot）或对象处置（Unity） |
| **External state** | 当文件、场景或全局变量来自先前测试时失败 | 将测试与文件系统隔离；使用内存中的 mock |
| **Floating point** | 在 `== 0.5` 等比较上失败 | 使用 epsilon 比较（`is_equal_approx`、`Assert.AreApproximately`） |
| **Scene/prefab load race** | 在场景尚未就绪时失败 | 实例化后等待一帧；使用 `await get_tree().process_frame` |

使用 Grep 检查测试文件中的计时调用、randf、全局状态访问或浮点数上的相等比较，以缩小原因范围。

---

## 5. 推荐操作

对于每个不稳定测试：

**隔离（高不稳定性）：**
> "Quarantine this test immediately. Disable it in CI by adding
> `@pytest.mark.skip` / `[Ignore]` / `GdUnitSkip` annotation. Log it in
> `tests/regression-suite.md` quarantine section. The test is now opt-in only.
> Fix the root cause before removing quarantine."

**调查并尽快修复（中等）：**
> "This test is intermittently unreliable. Root cause appears to be [cause].
> Suggested fix: [specific fix based on cause classification]. Do not quarantine
> yet — fix the test directly."

**监控（低/疑似）：**
> "This test shows suspected flakiness. Collect more run data before
> quarantining. Note it as 'suspected' in the regression suite."

---

## 6. 生成报告

### 对话中总结

```
## Flakiness Detection Results

**Runs analysed**: [N]
**Tests tracked**: [N]

### Flaky Tests Found

| Test | System | Fail Rate | Likely Cause | Recommendation |
|------|--------|-----------|--------------|----------------|
| [test_name] | [system] | [N]% | Timing | Quarantine + fix async |
| [test_name] | [system] | [N]% | Float comparison | Fix: use epsilon compare |
| [test_name] | [system] | [N]% | Order dependency | Investigate teardown |

### Clean Tests (no flakiness detected)

[N] tests ran across [N] runs with consistent results — no flakiness detected.

### Data Limitations

[Note if fewer than 5 runs were available — fewer runs = less statistical confidence]
```

---

## 7. 更新回归套件 + 可选报告文件

询问："May I update the quarantine section of `tests/regression-suite.md` with the flaky tests found?"

如果同意：使用 `Edit` 将条目追加到 Quarantined Tests 表格。永远不要删除现有隔离条目 — 只添加新条目。

（单独）询问："May I write a full flakiness report to `production/qa/flakiness-report-[date].md`?"

完整报告包含每个测试的分析，带有原因细节和引擎特定的修复代码片段。

写入后：

- 对于每个隔离的测试："Add the engine-specific skip annotation to disable this test in CI. Re-enable after the root cause is fixed."
- 对于可修复的测试："The fix for [test] is straightforward — change the equality comparison on line [N] to use `is_equal_approx`."
- 总结："Once all quarantine annotations are applied, CI should run green. Schedule fix work for the [N] quarantined tests before the release gate."

---

## 协作协议

- **永远不要删除测试文件** — 隔离意味着注释 + 列出，不是移除
- **统计置信度很重要** — 少于 3 次运行时，将发现标记为"疑似"而非"确认"；询问是否有更多运行数据可用
- **修复始终是目标** — 隔离是临时的；即使在推荐隔离时也要暴露修复方向
- **写入前先询问** — regression-suite 更新和报告文件都需要明确批准。写入时：结论：**COMPLETE** — 不稳定性报告已写入。拒绝时：结论：**BLOCKED** — 用户拒绝写入。
- **CI 中的不稳定性是团队问题** — 清晰地暴露列表和推荐操作；不要只在团队不知情的情况下静默隔离
