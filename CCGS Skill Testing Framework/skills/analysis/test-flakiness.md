# 技能测试规格: /test-flakiness

## 技能摘要

`/test-flakiness` 通过分析测试历史日志（如果可用）或扫描测试源代码中的常见 flakiness 模式（无种子的随机数、实时等待、外部 I/O）来检测非确定性测试。不触发任何 director gate。Skill 未经用户批准不写入。判定结果：NO FLAKINESS、SUSPECT TESTS FOUND 或 CONFIRMED FLAKY。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：NO FLAKINESS、SUSPECT TESTS FOUND、CONFIRMED FLAKY
- [ ] 不要求 "May I write" 语言（只读；可选报告需批准）
- [ ] 具有下一步交接说明（flakiness 结果后如何处理）

---

## 导演门控检查

无。Flakiness 检测是供 QA 负责人使用的建议性质量技能；不触发任何 gate。

---

## 测试用例

### Case 1: Happy Path — 干净的测试历史，无 flakiness

**Fixture:**
- `production/qa/test-history/` 包含 10 次测试运行的日志
- 所有测试在所有 10 次运行中一致通过（每个测试 100% 通过率）
- 没有测试有失败模式

**Input:** `/test-flakiness`

**Expected behavior:**
1. Skill 读取 `production/qa/test-history/` 中的测试历史日志
2. Skill 计算 10 次运行中每个测试的通过率
3. 所有测试通过所有 10 次运行 — 未检测到不一致
4. 判定为 NO FLAKINESS

**Assertions:**
- [ ] 可用时 Skill 读取测试历史日志
- [ ] 跨所有可用运行计算每个测试的通过率
- [ ] 所有测试一致通过时判定为 NO FLAKINESS
- [ ] 不写入任何文件

---

### Case 2: Suspect Tests Found — 测试在历史中间歇性失败

**Fixture:**
- `production/qa/test-history/` 包含 10 次测试运行的日志
- `test_combat_damage_applies_crit_multiplier` 通过 7 次，失败 3 次
- 失败消息不同（有时超时，有时值错误）

**Input:** `/test-flakiness`

**Expected behavior:**
1. Skill 读取测试历史日志 — 计算通过率
2. `test_combat_damage_applies_crit_multiplier` 通过率为 70%（阈值：95%）
3. Skill 将其标记为 SUSPECT，附带通过率 (7/10) 和失败模式说明
4. 判定为 SUSPECT TESTS FOUND
5. Skill 建议调查测试的时间或状态依赖

**Assertions:**
- [ ] 低于通过率阈值的测试按名称标记
- [ ] 每个可疑测试显示通过率（分数和百分比）
- [ ] 失败模式（如不一致的错误消息）如可检测则注明
- [ ] 判定为 SUSPECT TESTS FOUND
- [ ] Skill 建议调查步骤

---

### Case 3: Source Pattern — 使用随机数但未设置种子

**Fixture:**
- 没有测试历史日志存在
- `tests/unit/loot/loot_drop_test.gd` 包含：
  ```gdscript
  var roll = randf()  # 无种子随机数 — 非确定性
  assert_gt(roll, 0.5, "Loot should drop above 50%")
  ```

**Input:** `/test-flakiness`

**Expected behavior:**
1. Skill 未找到测试历史日志
2. Skill 回退到源代码分析
3. Skill 检测到 `randf()` 调用前没有 `seed()` 调用
4. Skill 将测试标记为 FLAKINESS RISK（源模式，未确认）
5. 判定为 SUSPECT TESTS FOUND（检测到模式，未通过历史确认）
6. Skill 建议在调用前设置种子或 mock 随机函数

**Assertions:**
- [ ] 没有历史日志时使用源代码分析作为回退
- [ ] 检测到无种子随机数使用为 flakiness 风险
- [ ] 判定为 SUSPECT TESTS FOUND（非 CONFIRMED FLAKY — 无历史确认）
- [ ] 修复建议为设置种子或 mock

---

### Case 4: No Test History — 仅源分析，使用常见模式

**Fixture:**
- `production/qa/test-history/` 不存在
- `tests/` 包含 15 个测试文件
- 扫描发现 2 个测试使用 `OS.get_ticks_msec()` 进行时间断言
- 未发现其他 flakiness 模式

**Input:** `/test-flakiness`

**Expected behavior:**
1. Skill 检查测试历史 — 未找到
2. Skill 注明："No test history available — analyzing source code for flakiness patterns only"
3. Skill 扫描所有测试文件的已知模式：无种子随机、实时等待、系统时钟使用
4. 发现 2 个测试使用 `OS.get_ticks_msec()` — 标记为 FLAKINESS RISK
5. 判定为 SUSPECT TESTS FOUND

**Assertions:**
- [ ] Skill 明确注明正在执行仅源分析（无历史）
- [ ] 扫描常见 flakiness 模式：随机、基于时间的断言、外部 I/O
- [ ] `OS.get_ticks_msec()` 用于断言时标记为 flakiness 风险
- [ ] 发现源模式时判定为 SUSPECT TESTS FOUND

---

### Case 5: Gate Compliance — 无 gate；flakiness 报告为建议性

**Fixture:**
- 测试历史显示 1 个 CONFIRMED FLAKY 测试（10 次运行中失败 6 次）
- `review-mode.txt` 内容为 `full`

**Input:** `/test-flakiness`

**Expected behavior:**
1. Skill 分析测试历史；识别 1 个确认的 flaky 测试
2. 无论审查模式如何，不触发任何 director gate
3. 判定为 CONFIRMED FLAKY
4. Skill 呈现结果并提供可选的书面报告
5. 如果用户选择："May I write to `production/qa/flakiness-report-[date].md`?"

**Assertions:**
- [ ] 任何审查模式下都不触发 director gate
- [ ] CONFIRMED FLAKY 判定需要基于历史的证据（不仅是源模式）
- [ ] 可选报告在写入前需要 "May I write"
- [ ] Flakiness 报告供 qa-lead 参考；skill 不自动禁用测试

---

## Protocol Compliance

- [ ] 可用时读取测试历史日志；不可用时回退到源分析
- [ ] 明确注明使用哪种分析模式（历史 vs. 仅源）
- [ ] 使用 flakiness 阈值（如 95% 通过率）进行 SUSPECT 分类
- [ ] CONFIRMED FLAKY 需要历史证据；SUSPECT 仅涵盖源模式
- [ ] 不禁用或修改任何测试文件
- [ ] 不触发任何 director gate
- [ ] 判定为以下之一：NO FLAKINESS、SUSPECT TESTS FOUND、CONFIRMED FLAKY

---

## 覆盖说明

- SUSPECT 分类的通过率阈值（上面建议的 95%）是实现细节；测试验证间歇性失败被标记，而非精确阈值值。
- 因环境问题（缺少资源、错误平台）失败的测试不属于 flakiness — skill 区分环境失败与测试本身的非确定性；此处未显式测试此区分。
