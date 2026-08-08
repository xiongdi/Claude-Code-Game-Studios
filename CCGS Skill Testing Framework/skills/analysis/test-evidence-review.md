# 技能测试规格: /test-evidence-review

## 技能摘要

`/test-evidence-review` 对 `tests/` 中的测试文件执行质量审查，检查测试命名规范、确定性、隔离性和无硬编码魔法数字 — 所有检查都基于 `coding-standards.md` 中定义的项目测试标准。结果可标记供 qa-lead 审查。不触发任何 director gate。Skill 未经用户批准不写入。判定结果：PASS、WARNINGS 或 FAIL。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：PASS、WARNINGS、FAIL
- [ ] 不要求 "May I write" 语言（只读；写入为可选标记报告）
- [ ] 具有下一步交接说明（审查结果后如何处理）

---

## 导演门控检查

无。测试证据审查是建议性质量技能；QL-TEST-COVERAGE gate 是单独的 skill 调用，不在此处触发。

---

## 测试用例

### Case 1: Happy Path — 测试遵循所有标准

**Fixture:**
- `tests/unit/combat/health_system_take_damage_test.gd` 存在，包含：
  - 命名：`test_health_system_take_damage_reduces_health()`（遵循 `test_[system]_[scenario]_[expected]`）
  - 存在 Arrange/Act/Assert 结构
  - 无 `sleep()`、带时间值的 `await` 或随机种子
  - 无外部 API 或文件 I/O 调用
  - 无内联魔法数字（使用 `tests/unit/combat/fixtures/` 中的常量）

**Input:** `/test-evidence-review tests/unit/combat/`

**Expected behavior:**
1. Skill 从 `coding-standards.md` 读取测试标准
2. Skill 读取测试文件；检查所有 5 项标准
3. 所有检查通过：命名、结构、确定性、隔离、无硬编码数据
4. 判定为 PASS

**Assertions:**
- [ ] 检查并报告 5 项测试标准中的每一项
- [ ] 标准满足时所有检查显示 PASS
- [ ] 判定为 PASS
- [ ] 不写入任何文件

---

### Case 2: Fail — 检测到时间依赖

**Fixture:**
- `tests/unit/ui/hud_update_test.gd` 包含：
  ```gdscript
  await get_tree().create_timer(1.0).timeout
  assert_eq(label.text, "Ready")
  ```
- 使用 1 秒的实时等待而非 mock 或基于信号的断言

**Input:** `/test-evidence-review tests/unit/ui/hud_update_test.gd`

**Expected behavior:**
1. Skill 读取测试文件
2. Skill 检测到实时等待（`create_timer(1.0)`）— 非确定性时间依赖
3. Skill 将其标记为 FAIL 级别结果
4. 判定为 FAIL
5. Skill 建议用基于信号的断言或 mock 替换计时器

**Assertions:**
- [ ] 实时等待使用被检测为非确定性时间依赖
- [ ] 结果分类为 FAIL 严重级别（阻塞 — 违反确定性标准）
- [ ] 判定为 FAIL
- [ ] 修复建议引用基于信号或 mock 的方法
- [ ] Skill 不修改测试文件

---

### Case 3: Fail — 测试直接调用外部 API

**Fixture:**
- `tests/unit/networking/auth_test.gd` 包含：
  ```gdscript
  var result = HTTPRequest.new().request("https://api.example.com/auth")
  ```
- 直接 HTTP 调用外部 API，无 mock

**Input:** `/test-evidence-review tests/unit/networking/auth_test.gd`

**Expected behavior:**
1. Skill 读取测试文件
2. Skill 检测到直接外部 API 调用（HTTPRequest 到实时 URL）
3. Skill 将其标记为 FAIL 级别结果 — 违反隔离标准
4. 判定为 FAIL
5. Skill 建议注入 mock HTTP 客户端

**Assertions:**
- [ ] 直接外部 API 调用被检测和标记
- [ ] 结果分类为 FAIL 严重级别（违反隔离标准）
- [ ] 判定为 FAIL
- [ ] 修复建议引用使用 mock HTTP 客户端的依赖注入
- [ ] Skill 不修改测试文件

---

### Case 4: Edge Case — 未找到测试文件

**Fixture:**
- 用户调用 `/test-evidence-review tests/unit/audio/`
- `tests/unit/audio/` 目录不存在

**Input:** `/test-evidence-review tests/unit/audio/`

**Expected behavior:**
1. Skill 尝试读取 `tests/unit/audio/` 中的文件 — 未找到
2. Skill 输出："No test files found at `tests/unit/audio/` — run `/test-setup` to scaffold test directories"
3. 不输出判定

**Assertions:**
- [ ] 路径不存在时 Skill 不崩溃
- [ ] 消息中命名了尝试的路径
- [ ] 输出推荐 `/test-setup` 用于搭建脚手架
- [ ] 没有审查内容时不输出判定

---

### Case 5: Gate Compliance — 无 gate；QL-TEST-COVERAGE 是单独的技能

**Fixture:**
- 测试文件有 1 个 WARNINGS 级别结果（非边界测试中的魔法数字）
- `review-mode.txt` 内容为 `full`

**Input:** `/test-evidence-review tests/unit/combat/`

**Expected behavior:**
1. Skill 审查测试；发现 1 个 WARNINGS 级别结果
2. 不触发任何 director gate（QL-TEST-COVERAGE 单独调用，不在此处）
3. 判定为 WARNINGS
4. 输出注明："For full test coverage gate, run `/gate-check` which invokes QL-TEST-COVERAGE"
5. Skill 提供可选报告写入；如果用户选择则询问 "May I write"

**Assertions:**
- [ ] 任何审查模式下都不触发 director gate
- [ ] 输出区分此 skill 与 QL-TEST-COVERAGE gate 调用
- [ ] 可选报告在写入前需要 "May I write"
- [ ] 建议性级别测试质量问题判定为 WARNINGS

---

## Protocol Compliance

- [ ] 审查测试文件前从 `coding-standards.md` 读取测试标准
- [ ] 检查命名、Arrange/Act/Assert 结构、确定性、隔离、无硬编码数据
- [ ] 不修改任何测试文件（只读技能）
- [ ] 不触发任何 director gate
- [ ] 判定为以下之一：PASS、WARNINGS、FAIL

---

## 覆盖说明

- 未显式测试 `tests/` 中所有测试文件的批量审查行为；假设逐文件应用相同检查并汇总判定。
- QL-TEST-COVERAGE director gate（检查测试覆盖率百分比）是单独的关注点，此 skill 有意不触发。
