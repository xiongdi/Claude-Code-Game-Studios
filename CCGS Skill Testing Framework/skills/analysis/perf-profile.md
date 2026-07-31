# Skill Test Spec: /perf-profile

## Skill Summary

`/perf-profile` 是结构化的性能分析工作流，用于识别瓶颈并推荐优化。如果提供了性能分析器数据或性能日志，则直接分析。如果没有，则引导用户完成手动分析清单。不触发任何 director gate。Skill 在持久化报告前会询问 "May I write to `production/qa/perf-[date].md`?"。判定结果：WITHIN BUDGET、CONCERNS 或 OVER BUDGET。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：WITHIN BUDGET、CONCERNS、OVER BUDGET
- [ ] 包含 "May I write" 语言（skill 写入 perf 报告）
- [ ] 具有下一步交接说明（审查性能结果后如何处理）

---

## Director Gate Checks

无。性能分析是建议性分析技能；不触发任何 gate。

---

## Test Cases

### Case 1: Happy Path — 提供帧数据，发现 draw call 峰值

**Fixture:**
- 用户提供 `production/qa/profiler-export-2026-03-15.json`，包含帧时间数据
- 数据显示：平均帧时间 14ms（在 16.6ms 预算内），但第 42–48 帧峰值达 28ms
- 峰值与一个 450 draw calls 的场景相关（预算：200）

**Input:** `/perf-profile production/qa/profiler-export-2026-03-15.json`

**Expected behavior:**
1. Skill 读取性能分析器数据
2. Skill 识别平均帧时间在预算内
3. Skill 识别第 42–48 帧的 draw call 峰值（450 calls vs 200 预算）
4. 判定为 CONCERNS（平均值正常，但峰值表明存在问题）
5. Skill 建议对识别的场景进行批处理或剔除
6. Skill 询问 "May I write to `production/qa/perf-2026-04-06.md`?"

**Assertions:**
- [ ] 峰值帧按帧号识别
- [ ] draw call 数量和预算明确比较
- [ ] 即使平均值正常，峰值超过预算时判定为 CONCERNS
- [ ] 至少提供一条具体优化建议
- [ ] 写入报告前出现 "May I write" 提示

---

### Case 2: No Profiler Data — 输出手动清单

**Fixture:**
- 用户无参数运行 `/perf-profile`
- `production/qa/` 中不存在性能分析器数据文件

**Input:** `/perf-profile`

**Expected behavior:**
1. Skill 未找到性能分析器数据
2. Skill 输出手动分析清单供用户执行：
   - 启用 Godot 性能分析器或目标引擎的性能分析器
   - 录制 60 秒游戏会话
   - 导出帧时间数据
   - 注意任何掉帧或卡顿
3. Skill 要求用户在提供数据后运行分析

**Assertions:**
- [ ] 未提供数据时 Skill 不崩溃或输出判定
- [ ] 输出手动分析清单（可操作步骤，不仅是错误）
- [ ] 不输出判定（尚无内容可评估）
- [ ] 不写入任何文件

---

### Case 3: Over Budget — 目标平台超出帧预算

**Fixture:**
- 性能分析器数据一致显示 22ms 帧时间（目标：60fps 对应 16.6ms）
- 所有帧都超出预算；无单一峰值 — 系统性问题
- `technical-preferences.md` 指定目标平台：PC，60fps

**Input:** `/perf-profile production/qa/profiler-export-2026-03-20.json`

**Expected behavior:**
1. Skill 读取性能分析器数据和技术偏好中的性能预算
2. 所有帧都超出 16.6ms 预算
3. 判定为 OVER BUDGET
4. Skill 输出优先级优化列表（如 LOD 系统、shader 复杂度、物理 tick rate）
5. Skill 在写入报告前询问 "May I write"

**Assertions:**
- [ ] 所有或大多数帧超出预算时判定为 OVER BUDGET
- [ ] 目标帧预算从 `technical-preferences.md` 读取（非硬编码）
- [ ] 提供优化优先级列表，而非仅原始判定
- [ ] 报告写入前出现 "May I write" 提示

---

### Case 4: Previous Perf Report Exists — 增量比较

**Fixture:**
- `production/qa/perf-2026-03-28.md` 存在，包含先前结果（平均 15ms，最大 19ms）
- 新的性能分析器导出显示：平均 13ms，最大 17ms
- 两个报告针对同一场景

**Input:** `/perf-profile production/qa/profiler-export-2026-04-05.json`

**Expected behavior:**
1. Skill 读取新的性能分析器数据
2. Skill 检测到同一场景的先前报告
3. Skill 计算增量：平均改善 2ms，最大改善 2ms
4. Skill 呈现回归检查：未检测到回归
5. 判定为 WITHIN BUDGET；报告注明自上次分析以来的改善

**Assertions:**
- [ ] Skill 在写入前检查 `production/qa/` 中的先前 perf 报告
- [ ] 显示增量比较（关键指标的先前 vs. 当前）
- [ ] 当前指标在预算内时判定为 WITHIN BUDGET
- [ ] 报告积极注明改善趋势

---

### Case 5: Gate Compliance — 无 gate；performance-analyst 单独处理

**Fixture:**
- 性能分析器数据显示 CONCERNS 级别结果（一些峰值）
- `review-mode.txt` 内容为 `full`

**Input:** `/perf-profile production/qa/profiler-export-2026-04-01.json`

**Expected behavior:**
1. Skill 分析性能分析器数据；判定为 CONCERNS
2. 无论审查模式如何，不触发任何 director gate
3. 输出注明："For in-depth analysis, consider running `/perf-profile` with the performance-analyst agent"
4. Skill 询问 "May I write" 并在用户批准后写入报告

**Assertions:**
- [ ] 任何审查模式下都不触发 director gate
- [ ] 建议（非强制）咨询 performance-analyst
- [ ] 报告写入前出现 "May I write" 提示
- [ ] 基于峰值的结果判定为 CONCERNS

---

## Protocol Compliance

- [ ] 提供时读取性能分析器数据；未提供时输出清单
- [ ] 从 `technical-preferences.md` 读取目标平台帧预算
- [ ] 检查先前 perf 报告以启用增量比较
- [ ] 写入报告前始终询问 "May I write"
- [ ] 不触发任何 director gate
- [ ] 判定为以下之一：WITHIN BUDGET、CONCERNS、OVER BUDGET

---

## Coverage Notes

- 平台特定的性能分析工作流（主机、移动端）此处未测试；Case 2 中的清单输出在实践中会是平台特定的。
- Case 4 中的增量比较假设报告涵盖同一场景；跨场景比较未显式处理。
