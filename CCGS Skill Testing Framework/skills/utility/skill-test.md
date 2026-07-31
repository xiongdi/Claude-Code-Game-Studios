# Skill Test Spec: /skill-test

## Skill Summary

`/skill-test` 验证技能文件的结构正确性、行为合规性和类别评分。它以三种模式运行：

- **static**：检查单个技能文件的结构需求（frontmatter 字段、phase 标题、裁决关键词、"May I write" 语言、下一步交接），不需要 fixture。生成每个检查的 PASS/FAIL 表。
- **spec**：从 `tests/skills/` 读取测试规格文件，针对每个测试用例断言评估技能，生成逐案裁决。
- **audit**：生成 `.claude/skills/` 中所有技能和 `.claude/agents/` 中所有 agent 的覆盖率表，显示哪些有规格文件，哪些没有。

额外的 **category** 模式读取技能类别的质量评分标准（例如，gate 技能），并根据评分标准标准对技能进行评分。裁决系统因模式而异。

---

## Static Assertions (Structural)

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决：COMPLIANT、NON-COMPLIANT、WARNINGS（static 模式）；PASS、FAIL、PARTIAL（spec 模式）；COMPLETE（audit 模式）
- [ ] 不包含 "May I write" 语言（技能在所有模式下都是只读的）
- [ ] 具有下一步交接（例如，`/skill-improve` 以修复发现的问题）

---

## Director Gate Checks

无。`/skill-test` 是一个元工具技能。不适用 director gate。

---

## Test Cases

### Case 1: Static Mode — Well-formed skill, all 7 checks pass, COMPLIANT

**Fixture:**
- `.claude/skills/brainstorm/SKILL.md` 存在且格式良好：
  - 具有所有必需的 frontmatter 字段
  - 具有 ≥2 个 phase 标题
  - 具有裁决关键词
  - 具有 "May I write" 语言
  - 具有下一步交接
  - 记录 director gate
  - 记录 gate 模式行为（lean/solo 跳过）

**Input:** `/skill-test static brainstorm`

**Expected behavior:**
1. Skill 读取 `.claude/skills/brainstorm/SKILL.md`
2. Skill 运行所有 7 个结构检查
3. 所有 7 个检查通过
4. Skill 输出一个 PASS/FAIL 表，所有 7 个检查标记为 PASS
5. 裁决为 COMPLIANT

**Assertions:**
- [ ] 报告恰好 7 个结构检查
- [ ] 所有 7 个都标记为 PASS
- [ ] 裁决为 COMPLIANT
- [ ] 不写入任何文件

---

### Case 2: Static Mode — Skill Missing "May I Write" Despite Write Tool in allowed-tools

**Fixture:**
- `.claude/skills/some-skill/SKILL.md` 的 `allowed-tools` frontmatter 中有 `Write`
- 技能正文中没有 "May I write" 或 "May I update" 语言

**Input:** `/skill-test static some-skill`

**Expected behavior:**
1. Skill 读取 `some-skill/SKILL.md`
2. 检查 4（协作写入协议）失败：allowed-tools 中有 `Write` 但未找到 "May I write" 语言
3. 所有其他检查可能通过
4. 裁决为 NON-COMPLIANT，检查 4 为失败断言
5. 输出将检查 4 列为 FAIL 并附带说明

**Assertions:**
- [ ] 检查 4 标记为 FAIL
- [ ] 说明识别了具体的不匹配（有 Write 工具但无 "May I write" 语言）
- [ ] 裁决为 NON-COMPLIANT
- [ ] 显示其他通过的检查（不仅是失败）

---

### Case 3: Spec Mode — gate-check Skill Evaluated Against Spec

**Fixture:**
- `tests/skills/gate-check.md` 存在并包含 5 个测试用例
- `.claude/skills/gate-check/SKILL.md` 存在

**Input:** `/skill-test spec gate-check`

**Expected behavior:**
1. Skill 读取技能文件和规格文件
2. Skill 针对技能行为评估 5 个测试用例断言中的每一个
3. 对于每个用例：如果技能行为与规格断言匹配则为 PASS，否则为 FAIL
4. Skill 生成逐案结果表
5. 总体裁决：PASS（全部 5 个）、PARTIAL（部分）或 FAIL（大多数失败）

**Assertions:**
- [ ] 评估规格中的所有 5 个测试用例
- [ ] 每个用例都有单独的 PASS/FAIL 结果
- [ ] 总体裁决基于用例结果为 PASS、PARTIAL 或 FAIL
- [ ] 不写入任何文件

---

### Case 4: Audit Mode — Coverage Table of All Skills and Agents

**Fixture:**
- `.claude/skills/` 包含 72+ 个技能目录
- `.claude/agents/` 包含 49+ 个 agent 文件
- `tests/skills/` 包含子集技能的规格文件

**Input:** `/skill-test audit`

**Expected behavior:**
1. Skill 枚举 `.claude/skills/` 中的所有技能和 `.claude/agents/` 中的所有 agent
2. Skill 检查 `tests/skills/` 中每个对应的规格文件
3. Skill 生成覆盖率表：
   - 列出每个技能/agent
   - "Has Spec" 列：YES 或 NO
   - 摘要："X of Y skills have specs; A of B agents have specs"
4. 裁决为 COMPLETE

**Assertions:**
- [ ] 枚举所有技能目录（不仅是样本）
- [ ] "Has Spec" 列对每个条目准确
- [ ] 摘要计数正确
- [ ] 裁决为 COMPLETE

---

### Case 5: Category Mode — Gate Skill Evaluated Against Quality Rubric

**Fixture:**
- `tests/skills/quality-rubric.md` 存在并包含 "Gate Skills" 部分，定义标准 G1-G5（例如，G1：具有模式守卫，G2：具有裁决表等）
- `.claude/skills/gate-check/SKILL.md` 是一个 gate 技能

**Input:** `/skill-test category gate-check`

**Expected behavior:**
1. Skill 读取 `quality-rubric.md` 并识别 Gate Skills 部分
2. Skill 针对标准 G1-G5 评估 `gate-check/SKILL.md`
3. 每个标准评分：PASS、PARTIAL 或 FAIL
4. 计算总体类别分数（例如，4/5 标准通过）
5. 裁决为 COMPLIANT（全部通过）、WARNINGS（部分）或 NON-COMPLIANT（失败）

**Assertions:**
- [ ] 评估来自 quality-rubric.md 的所有 gate 标准（G1-G5）
- [ ] 每个标准都有单独的分数
- [ ] 总体裁决反映分数分布
- [ ] 不写入任何文件

---

## Protocol Compliance

- [ ] Static 模式恰好检查 7 个结构断言
- [ ] Spec 模式单独评估规格文件中的每个测试用例
- [ ] Audit 模式覆盖所有技能 AND agent（不仅是一个类别）
- [ ] Category 模式读取 quality-rubric.md 获取标准（不是硬编码）
- [ ] 在任何模式下都不写入任何文件
- [ ] 发现问题时建议 `/skill-improve` 作为下一步

---

## Coverage Notes

- skill-test 技能是自引用的（它可以测试自己）。为避免测试设计中的无限递归，skill-test 自身 SKILL.md 的 static 模式用例未单独进行 fixture 测试。
- 具体的 7 个结构检查在技能正文中定义；只有检查 4（May I write）在此单独测试，因为它具有最细致的逻辑。
- Audit 模式计数是近似值——技能和代理的准确数量会随着系统增长而变化；断言使用 "all" 而不是固定计数。
