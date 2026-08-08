# 技能测试规格: /skill-improve

## 技能摘要

`/skill-improve` 对技能文件运行自动化的测试-修复-重测改进循环。它调用 `/skill-test static`（以及可选的 `/skill-test category`）建立基线分数，诊断失败的检查，向 SKILL.md 文件提出有针对性的修复，询问 "May I write the improvements to [skill path]?"，应用修复，并重新运行测试以确认改进。

如果提出的修复使技能变差（回退），则修复会被撤销（经用户确认）而不是应用。如果技能已经完美（0 失败），技能会立即退出而不做任何更改。不适用 director gate。裁决：IMPROVED（分数上升）、NO CHANGE（无改进可能或用户拒绝）或 REVERTED（修复已应用但导致回退并被撤销）。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：IMPROVED、NO CHANGE、REVERTED
- [ ] 在应用修复前包含 "May I write" 协作协议语言
- [ ] 具有下一步交接（例如，运行 `/skill-test spec` 以验证行为合规性）

---

## 导演门控检查

无。`/skill-improve` 是一个元工具技能。不适用 director gate。

---

## 测试用例

### Case 1: Happy Path — Skill With 2 Static Failures, Both Fixed, IMPROVED

**Fixture:**
- `.claude/skills/some-skill/SKILL.md` 有 2 个静态失败：
  - 检查 4：尽管 allowed-tools 中有 Write，但没有 "May I write" 语言
  - 检查 5：末尾没有下一步交接

**Input:** `/skill-improve some-skill`

**Expected behavior:**
1. Skill 运行 `/skill-test static some-skill`——基线：5/7 检查通过
2. Skill 诊断 2 个失败的检查（4 和 5）
3. Skill 提出修复：
   - 在适当的阶段添加 "May I write" 语言
   - 在末尾添加下一步交接部分
4. Skill 询问 "May I write improvements to `.claude/skills/some-skill/SKILL.md`?"
5. 应用修复；重新运行 `/skill-test static some-skill`——现在 7/7 检查通过
6. 裁决为 IMPROVED（5→7）

**Assertions:**
- [ ] 在任何更改之前建立基线分数（5/7）
- [ ] 诊断并在提出的修复中解决两个失败的检查
- [ ] 在应用修复前询问 "May I write"
- [ ] 重测确认改进（7/7）
- [ ] 裁决为 IMPROVED 并显示前后分数

---

### Case 2: Fix Causes Regression — Score Comparison Shows Regression, REVERTED

**Fixture:**
- `.claude/skills/some-skill/SKILL.md` 有 1 个静态失败（缺少交接）
- 提出的修复无意中删除了裁决关键词部分
  （引入新的失败）

**Input:** `/skill-improve some-skill`

**Expected behavior:**
1. 基线：6/7 检查通过（1 个失败：缺少交接）
2. Skill 提出修复并询问 "May I write improvements?"
3. 应用修复；运行重测
4. 重测结果：5/7（修复了交接但破坏了裁决关键词）
5. Skill 检测到回退：分数下降了
6. Skill 询问用户："Fix caused a regression (6→5). May I revert the changes?"
7. 用户确认；更改被撤销；裁决为 REVERTED

**Assertions:**
- [ ] 在最终确定前将重测分数与基线进行比较
- [ ] 分数下降时检测到回退
- [ ] 询问用户确认撤销（不是自动的）
- [ ] 用户确认后文件被撤销
- [ ] 裁决为 REVERTED

---

### Case 3: Skill With Category Assignment — Baseline Captures Both Scores

**Fixture:**
- `.claude/skills/gate-check/SKILL.md` 是一个 gate 技能，有 1 个静态失败
  和 2 个类别（G-标准）失败
- `tests/skills/quality-rubric.md` 有 Gate Skills 部分

**Input:** `/skill-improve gate-check`

**Expected behavior:**
1. Skill 为基线运行静态和类别测试：
   - 静态：6/7 检查通过
   - 类别：3/5 G-标准通过
2. 组合基线：9/12
3. Skill 诊断所有 3 个失败并提出修复
4. "May I write improvements to `.claude/skills/gate-check/SKILL.md`?"
5. 应用修复；重新运行两种测试类型
6. 重测：静态 7/7，类别 5/5 = 12/12
7. 裁决为 IMPROVED（9→12）

**Assertions:**
- [ ] 基线中捕获了静态和类别分数
- [ ] 使用组合分数进行比较（不仅是其中一种）
- [ ] 提出的修复中解决了所有 3 个失败
- [ ] 重测确认两种分数类型都有改进
- [ ] 裁决为 IMPROVED 并显示组合前后分数

---

### Case 4: Skill Already Perfect — No Improvements Needed

**Fixture:**
- `.claude/skills/brainstorm/SKILL.md` 没有静态失败
- 类别分数也是 5/5（如果适用）

**Input:** `/skill-improve brainstorm`

**Expected behavior:**
1. Skill 运行 `/skill-test static brainstorm`——7/7 检查通过
2. 如果适用类别：5/5 标准通过
3. Skill 输出："No improvements needed — brainstorm is fully compliant"
4. Skill 退出而不提出任何更改
5. 不询问 "May I write"；不修改任何文件
6. 裁决为 NO CHANGE

**Assertions:**
- [ ] Skill 在确认 0 失败后立即退出
- [ ] 显示 "No improvements needed" 消息
- [ ] 不提出任何更改
- [ ] 不询问 "May I write"
- [ ] 裁决为 NO CHANGE

---

### Case 5: Director Gate Check — No gate; skill-improve is a meta utility

**Fixture:**
- 至少有 1 个静态失败的技能

**Input:** `/skill-improve some-skill`

**Expected behavior:**
1. Skill 运行测试-修复-重测循环
2. 不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 裁决为 IMPROVED、NO CHANGE 或 REVERTED——无 gate 裁决

---

## 协议合规

- [ ] 在提出任何更改之前始终建立基线分数
- [ ] 在输出中显示前后分数比较
- [ ] 在应用任何修复前询问 "May I write"
- [ ] 通过比较重测分数与基线检测回退
- [ ] 在撤销前询问用户确认（不是自动的）
- [ ] 以 IMPROVED、NO CHANGE 或 REVERTED 裁决结束

---

## 覆盖说明

- 改进循环设计为每次调用只运行一个修复-重测周期；运行多次迭代需要重新调用 `/skill-improve`。
- 行为合规性（spec 模式测试结果）不包含在改进循环中——只有结构（静态）和类别分数是自动化的。
- 技能文件无法读取（权限错误或文件缺失）的情况未测试；这会在建立基线之前导致错误。
