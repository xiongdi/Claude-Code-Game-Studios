# Skill 测试规格：/smoke-check

## Skill 摘要

`/smoke-check` 是实现和 QA 交接之间的 gate。它检测测试环境，运行自动化测试套件（通过 Bash），根据 sprint story 扫描测试覆盖率，并使用 `AskUserQuestion` 与开发者批量验证手动 smoke 检查。它在用户明确批准后将报告写入 `production/qa/smoke-[date].md`。

裁定：PASS（测试通过，所有 smoke 检查通过，无缺失测试证据）、PASS WITH WARNINGS（测试通过或 NOT RUN，所有关键检查通过，但存在建议性差距如缺失测试覆盖率）或 FAIL（任何自动化测试失败或任何 Batch 1/Batch 2 smoke 检查返回 FAIL）。

不适用 director gate。该 skill 不调用任何 director agent。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含裁定关键词：PASS、PASS WITH WARNINGS、FAIL
- [ ] 在写入报告前包含 "May I write" 协作协议语言
- [ ] 有下一步交接说明（例如，FAIL 时 `/bug-report`，PASS 时 QA 交接指导）

---

## Director 关卡检查

无。`/smoke-check` 是 QA 前实用 skill。不适用 director gate。

---

## 测试用例

### 用例 1：正常路径——自动化测试通过，手动项目确认，PASS

**Fixture：**
- `tests/` 目录存在，带有 GDUnit4 运行器脚本
- 引擎从 `technical-preferences.md` 检测为 Godot
- `production/qa/qa-plan-sprint-005.md` 存在
- 自动化测试运行器报告 12 个测试，12 个通过，0 个失败
- 开发者确认所有 Batch 1 和 Batch 2 smoke 检查为 PASS
- 所有 sprint story 都有匹配的测试文件（无 MISSING 覆盖率）

**输入：** `/smoke-check`

**预期行为：**
1. Skill 检测测试目录和引擎，注明找到 QA 计划
2. 通过 Bash 运行 `godot --headless --script tests/gdunit4_runner.gd`
3. 解析输出：12/12 通过
4. 扫描测试覆盖率——所有 story COVERED 或 EXPECTED
5. 对 Batch 1（核心稳定性）和 Batch 2（sprint 机制）使用 `AskUserQuestion`
6. 开发者对所有项目选择 PASS
7. 报告组装：自动化测试 PASS，所有 smoke 检查 PASS，无 MISSING 覆盖率
8. 询问 "May I write this smoke check report to `production/qa/smoke-[date].md`?"
9. 批准后写入报告
10. 交付裁定：PASS

**断言：**
- [ ] 自动化测试运行器通过 Bash 调用
- [ ] 对手动 smoke 检查批次使用 `AskUserQuestion`
- [ ] 在写入报告文件前询问 "May I write"
- [ ] 报告写入 `production/qa/smoke-[date].md`
- [ ] 裁定为 PASS

---

### 用例 2：失败路径——自动化测试失败，FAIL 裁定

**Fixture：**
- `tests/` 目录存在，引擎为 Godot
- 自动化测试运行器报告运行了 10 个测试：8 个通过，2 个失败
  - 失败的测试：`test_health_clamp_at_zero`、`test_damage_calculation_negative`
- QA 计划存在

**输入：** `/smoke-check`

**预期行为：**
1. Skill 通过 Bash 运行自动化测试
2. 解析输出——检测到 2 个失败
3. 记录失败测试名称
4. 继续通过手动 smoke 检查批次
5. 报告将自动化测试显示为 FAIL，列出失败测试名称
6. 询问写入报告；批准后写入
7. 交付 FAIL 裁定，消息为："The smoke check failed. Do not hand off to
   QA until these failures are resolved." 列出失败测试并建议修复后重新运行 `/smoke-check`

**断言：**
- [ ] 失败测试名称在报告中列出
- [ ] 裁定为 FAIL
- [ ] 裁定后消息指导开发者在 QA 交接前修复失败
- [ ] 修复后建议重新运行 `/smoke-check`

---

### 用例 3：手动确认——使用 AskUserQuestion，PASS WITH WARNINGS

**Fixture：**
- `tests/` 目录存在，引擎为 Godot
- 自动化测试运行器报告所有测试通过（8/8）
- 一个 Logic story 无匹配测试文件（MISSING 覆盖率）
- 开发者确认所有 Batch 1 和 Batch 2 smoke 检查为 PASS

**输入：** `/smoke-check`

**预期行为：**
1. 自动化测试 PASS
2. 覆盖率扫描发现 1 个 MISSING 条目，针对一个 Logic story
3. 对 Batch 1 和 Batch 2 使用 `AskUserQuestion`——开发者确认所有 PASS
4. 报告展示：自动化测试 PASS，手动检查全部 PASS，1 个 MISSING 覆盖率条目
5. 裁定为 PASS WITH WARNINGS——构建已准备好进行 QA，但 MISSING 条目必须在 `/story-done` 关闭受影响的 story 前解决
6. 询问写入报告；批准后写入

**断言：**
- [ ] 对手动 smoke 检查批次使用 `AskUserQuestion`（不是内联文本提示）
- [ ] MISSING 测试覆盖率条目出现在报告中
- [ ] 裁定为 PASS WITH WARNINGS（不是 PASS，不是 FAIL）
- [ ] 建议说明解释 MISSING 条目必须在 `/story-done` 前解决
- [ ] 报告文件写入 `production/qa/smoke-[date].md`

---

### 用例 4：无测试目录——Skill 停止并给出指导

**Fixture：**
- `tests/` 目录不存在
- 引擎配置为 Godot

**输入：** `/smoke-check`

**预期行为：**
1. Phase 1 检查 `tests/` 目录——未找到
2. Skill 输出："No test directory found at `tests/`. Run `/test-setup` to
   scaffold the testing infrastructure, or create the directory manually if
   tests live elsewhere."
3. Skill 停止——不运行自动化测试，不进行手动 smoke 检查，不写入报告

**断言：**
- [ ] 错误消息引用缺失的 `tests/` 目录
- [ ] `/setup-engine` 被建议为修复步骤
- [ ] Skill 在此消息后停止（不运行后续阶段）
- [ ] 不写入报告文件

---

### 用例 5：Director 关卡检查——无 gate；smoke-check 是 QA 前检查实用工具

**Fixture：**
- 有效的测试设置，自动化测试通过，手动 smoke 检查确认

**输入：** `/smoke-check`

**预期行为：**
1. Skill 运行所有阶段并产生 PASS 或 PASS WITH WARNINGS 裁定
2. 在任何时候都不派生 director agent
3. 输出中不出现 gate ID（CD-*、TD-*、AD-*、PR-*）
4. 不调用 `/gate-check`

**断言：**
- [ ] 不调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 裁定为 PASS、PASS WITH WARNINGS 或 FAIL——不涉及 gate 裁定

---

## 协议合规性

- [ ] 对所有手动 smoke 检查批次（Batch 1、Batch 2、Batch 3）使用 `AskUserQuestion`
- [ ] 在询问任何手动问题前通过 Bash 运行自动化测试
- [ ] 在创建报告文件前询问 "May I write"——永远不经批准不写入
- [ ] 裁定词汇严格为 PASS / PASS WITH WARNINGS / FAIL——无其他裁定
- [ ] FAIL 由自动化测试失败或 Batch 1/Batch 2 FAIL 响应触发
- [ ] PASS WITH WARNINGS 在存在 MISSING 测试覆盖率但无关键失败时触发
- [ ] NOT RUN（引擎二进制不可用）记录为警告，不是 FAIL
- [ ] 在任何时候都不调用 director gate

---

## 覆盖说明

- `quick` 参数（跳过 Phase 3 覆盖率扫描和 Batch 3）不单独进行 fixture 测试；
  它遵循与用例 1 相同的模式，输出中带有覆盖率跳过说明。
- `--platform` 参数添加特定于平台的 `AskUserQuestion` 批次和每个平台的裁定表；
  不在此单独测试。
- 引擎二进制不在 PATH 上的情况（NOT RUN）遵循 PASS WITH WARNINGS 模式，
  并由上述协议合规断言覆盖。
