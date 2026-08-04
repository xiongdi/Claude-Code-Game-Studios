# 技能测试规格: /regression-suite

## 技能摘要

`/regression-suite` 将测试覆盖率映射到 GDD 需求：它读取当前 sprint（或指定 epic）中 story 文件的验收标准，然后扫描 `tests/` 中对应的测试文件并检查每个 AC 是否有匹配的断言。它生成覆盖率报告，识别哪些 AC 完全覆盖、部分覆盖或未测试，以及哪些测试文件没有匹配的 AC（孤立测试）。

该 skill 可以在 "May I write" 请求后将覆盖率报告写入 `production/qa/`。不适用 director gate。裁定：FULL COVERAGE（所有 AC 都有测试）、GAPS FOUND（某些 AC 未测试）或 CRITICAL GAPS（关键优先级的 AC 无测试）。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：FULL COVERAGE、GAPS FOUND、CRITICAL GAPS
- [ ] 包含 "May I write" 语言（skill 可能写入覆盖率报告）
- [ ] 具有下一步交接说明（例如，`/test-setup` 如果框架缺失，`/qa-plan` 如果计划缺失）

---

## 导演门控检查

无。`/regression-suite` 是 QA 分析实用工具。不适用 director gate。

---

## 测试用例

### Case 1: Full Coverage——Sprint 中的所有 AC 都有对应测试

**Fixture:**
- `production/sprints/sprint-004.md` 列出 3 个 story，每个有 2 个 AC（共 6 个）
- `tests/unit/` 和 `tests/integration/` 包含与所有 6 个 AC 匹配的测试文件
  （通过系统名称和场景描述）

**Input:** `/regression-suite sprint-004`

**Expected behavior:**
1. Skill 读取 sprint-004 story 中的所有 6 个 AC
2. Skill 扫描测试文件并将每个 AC 匹配到至少一个测试断言
3. 所有 6 个 AC 都有覆盖
4. Skill 生成覆盖率报告："6/6 ACs covered"
5. Skill 询问 "May I write to `production/qa/regression-sprint-004.md`?"
6. 批准后写入文件；裁定为 FULL COVERAGE

**Assertions:**
- [ ] 所有 6 个 AC 都出现在覆盖率报告中
- [ ] 每个 AC 标记为已覆盖，并引用匹配的测试文件
- [ ] 裁定为 FULL COVERAGE
- [ ] 在写入报告前询问 "May I write"

---

### Case 2: Gaps Found——3 个 AC 无测试

**Fixture:**
- Sprint 有 5 个 story，共 8 个 AC
- 测试存在于 8 个 AC 中的 5 个；3 个 AC 无对应测试文件或断言

**Input:** `/regression-suite`

**Expected behavior:**
1. Skill 读取所有 8 个 AC
2. Skill 扫描测试——5 个匹配，3 个不匹配
3. 覆盖率报告按 story 和 AC 文本列出 3 个未测试的 AC
4. Skill 询问 "May I write to `production/qa/regression-[sprint]-[date].md`?"
5. 报告写入；裁定为 GAPS FOUND

**Assertions:**
- [ ] 3 个未测试的 AC 在报告中按名称列出
- [ ] 匹配的 AC 也显示（不仅是差距）
- [ ] 裁定为 GAPS FOUND（不是 FULL COVERAGE）
- [ ] 报告在 "May I write" 批准后写入

---

### Case 3: Critical AC Untested——CRITICAL GAPS 裁定，突出标记

**Fixture:**
- Sprint 有 4 个 story；一个 story 为 Priority: Critical，有 2 个 AC
- 关键优先级 AC 之一无测试

**Input:** `/regression-suite`

**Expected behavior:**
1. Skill 读取所有 story 和 AC，注意哪些 story 为关键优先级
2. Skill 扫描测试——关键 AC 无匹配
3. 报告突出标记："CRITICAL GAP: [AC text] — no test found (Critical priority story)"
4. Skill 建议阻塞 story 完成直到添加测试
5. 裁定为 CRITICAL GAPS

**Assertions:**
- [ ] 裁定为 CRITICAL GAPS（不是 GAPS FOUND）
- [ ] 关键优先级 AC 比普通差距更突出地标记
- [ ] 包含阻塞 story 完成的建议
- [ ] 非关键差距（如果存在）也列出

---

### Case 4: Orphan Tests——测试文件无匹配的 AC

**Fixture:**
- `tests/unit/save_system_test.gd` 存在，包含对当前 story AC 列表中
  不存在场景的断言
- 当前 sprint story 不引用保存系统

**Input:** `/regression-suite`

**Expected behavior:**
1. Skill 扫描测试并交叉引用 AC
2. `save_system_test.gd` 断言与任何当前 AC 不匹配
3. 测试文件在覆盖率报告中标记为 ORPHAN TEST
4. 报告注明："Orphan tests may belong to a past or future sprint, or AC was renamed"
5. 裁定为 FULL COVERAGE 或 GAPS FOUND，取决于整体 AC 覆盖率
   （孤立测试不影响裁定，它们是建议性的）

**Assertions:**
- [ ] 孤立测试在报告中标记
- [ ] 孤立标记包含文件名和建议（过去 sprint / 重命名的 AC）
- [ ] 孤立测试本身不导致 GAPS FOUND 裁定
- [ ] 整体裁定仅反映 AC 覆盖率

---

### Case 5: Director Gate Check——无 gate；regression-suite 是 QA 实用工具

**Fixture:**
- 具有 story 和测试文件的 sprint

**Input:** `/regression-suite`

**Expected behavior:**
1. Skill 生成覆盖率报告并写入
2. 不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 不调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 裁定为 FULL COVERAGE、GAPS FOUND 或 CRITICAL GAPS——无 gate 裁定

---

## 协议合规

- [ ] 在扫描测试前从 sprint 文件读取 story AC
- [ ] 通过系统名称和场景匹配 AC 到测试（不是仅文件名）
- [ ] 将关键优先级未测试的 AC 标记为 CRITICAL GAPS
- [ ] 标记孤立测试（存在于 tests/ 但无 AC 匹配）
- [ ] 在持久化覆盖率报告前询问 "May I write"
- [ ] 裁定为 FULL COVERAGE、GAPS FOUND 或 CRITICAL GAPS

---

## 覆盖说明

- 将 AC 匹配到测试的启发式方法（通过系统名称 + 场景关键词）是近似的；
  精确定义在 skill 正文中。
- 集成测试覆盖率的映射方式与单元测试覆盖率相同；
  两者在裁定上没有区别。
- 此 skill 不运行测试——它将 AC 文本映射到测试断言。
  测试执行由 CI 管道处理。
