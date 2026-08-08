# 技能测试规格: /brainstorm

## 技能摘要

`/brainstorm` 促进引导式的游戏概念构思。它展示 2-4 个带有优缺点对比的概念选项，让用户选择并完善一个概念，并生成一个结构化的 `design/gdd/game-concept.md` 文档。该技能是协作式的——它在提出选项前会提出问题，并迭代直到用户批准一个概念方向。

在 `full` 审查模式下，概念草稿完成后会并行派生四个 director gate：CD-PILLARS（creative-director）、AD-CONCEPT-VISUAL（art-director）、TD-FEASIBILITY（technical-director）和 PR-SCOPE（producer）。在 `lean` 模式下，所有 4 个内联 gate 都被跳过（lean 模式仅运行 PHASE-GATE，而 brainstorm 没有）。在 `solo` 模式下，所有 gate 都被跳过。该技能在写入 `design/gdd/game-concept.md` 前会询问 "May I write"。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：APPROVED、REJECTED、CONCERNS
- [ ] 对 game-concept.md 包含 "May I write" 协作协议语言
- [ ] 在末尾具有下一步交接（`/map-systems`）
- [ ] 记录 full 模式下的 4 个 director gate：CD-PILLARS、AD-CONCEPT-VISUAL、TD-FEASIBILITY、PR-SCOPE
- [ ] 记录 lean 和 solo 模式下所有 4 个 gate 都被跳过

---

## 导演门控检查

在 `full` 模式下：CD-PILLARS、AD-CONCEPT-VISUAL、TD-FEASIBILITY 和 PR-SCOPE
在用户批准概念草稿后并行派生。

在 `lean` 模式下：所有 4 个内联 gate 都被跳过（brainstorm 没有 PHASE-GATE，
所以 lean 模式跳过一切）。输出注明所有 4 个为："[GATE-ID] skipped — lean mode"。

在 `solo` 模式下：所有 4 个 gate 都被跳过。输出注明所有 4 个为："[GATE-ID] skipped — solo mode"。

---

## 测试用例

### Case 1: Happy Path — Full mode, 3 concepts, user picks one, all 4 directors approve

**Fixture:**
- 不存在现有的 `design/gdd/game-concept.md`
- `production/session-state/review-mode.txt` 包含 `full`

**Input:** `/brainstorm`

**Expected behavior:**
1. Skill 向用户询问关于类型、范围和目标感受的问题
2. Skill 展示 3 个概念选项，每个都有优缺点
3. 用户选择一个概念
4. Skill 将选定的概念详细阐述为结构化草稿
5. 所有 4 个 director gate 并行派生：CD-PILLARS、AD-CONCEPT-VISUAL、TD-FEASIBILITY、PR-SCOPE
6. 所有 4 个返回 APPROVED
7. Skill 询问 "May I write `design/gdd/game-concept.md`?"
8. 批准后写入概念

**Assertions:**
- [ ] 恰好展示 3 个概念选项（不是 1 个，不是 5+）
- [ ] 所有 4 个 director gate 并行派生（不是顺序）
- [ ] 所有 4 个 gate 在 "May I write" 询问之前完成
- [ ] 在写入前询问 "May I write `design/gdd/game-concept.md`?"
- [ ] 未经用户批准不写入概念文件
- [ ] 存在到 `/map-systems` 的下一步交接

---

### Case 2: Failure Path — CD-PILLARS returns REJECT

**Fixture:**
- 概念草稿已完成
- `production/session-state/review-mode.txt` 包含 `full`
- CD-PILLARS gate 返回 REJECT："The concept has no identifiable creative pillar"

**Input:** `/brainstorm`

**Expected behavior:**
1. CD-PILLARS gate 返回 REJECT 并附带具体反馈
2. Skill 向用户呈现拒绝
3. 概念不写入文件
4. 用户被询问：重新思考概念方向，或覆盖拒绝
5. 如果重新思考：skill 返回到概念选项阶段

**Assertions:**
- [ ] CD-PILLARS 返回 REJECT 时不写入概念
- [ ] 拒绝反馈逐字显示给用户
- [ ] 用户提供重新思考或覆盖的选项
- [ ] 如果用户选择重新思考，skill 返回到概念构思阶段

---

### Case 3: Lean Mode — All 4 gates skipped; concept written after user confirms

**Fixture:**
- 不存在现有游戏概念
- `production/session-state/review-mode.txt` 包含 `lean`

**Input:** `/brainstorm`

**Expected behavior:**
1. 展示概念选项且用户选择一个
2. 概念被详细阐述为结构化草稿
3. 所有 4 个 director gate 被跳过——每个注明："[GATE-ID] skipped — lean mode"
4. Skill 询问用户确认概念是否准备好写入
5. 确认后询问 "May I write `design/gdd/game-concept.md`?"
6. 批准后写入概念

**Assertions:**
- [ ] 出现所有 4 个 gate 跳过注释："CD-PILLARS skipped — lean mode"、"AD-CONCEPT-VISUAL skipped — lean mode"、"TD-FEASIBILITY skipped — lean mode"、"PR-SCOPE skipped — lean mode"
- [ ] 概念仅在用户确认后写入（lean 模式下不需要 director 批准）
- [ ] 在写入前仍询问 "May I write"

---

### Case 4: Solo Mode — All gates skipped; concept written with only user approval

**Fixture:**
- 不存在现有游戏概念
- `production/session-state/review-mode.txt` 包含 `solo`

**Input:** `/brainstorm`

**Expected behavior:**
1. 展示概念选项且用户选择一个
2. 向用户展示概念草稿
3. 所有 4 个 director gate 被跳过——每个注明 "solo mode"
4. 询问 "May I write `design/gdd/game-concept.md`?"
5. 用户批准后写入概念

**Assertions:**
- [ ] 所有 4 个跳过注释出现并带有 "solo mode" 标签
- [ ] 不派生 director agent
- [ ] 概念仅凭用户批准写入
- [ ] 此技能的其他行为与 lean 模式等效

---

### Case 5: Director Gate — PR-SCOPE returns CONCERNS (scope too large)

**Fixture:**
- 概念草稿已完成
- `production/session-state/review-mode.txt` 包含 `full`
- PR-SCOPE gate 返回 CONCERNS："The concept scope would require 18+ months for a solo developer"

**Input:** `/brainstorm`

**Expected behavior:**
1. PR-SCOPE gate 返回 CONCERNS 并附带具体的范围反馈
2. Skill 向用户呈现范围顾虑
3. 范围顾虑在写入前被记录在概念草稿中
4. 用户被询问：缩小范围、接受顾虑并记录，或重新思考
5. 如果接受顾虑：概念写入并嵌入 "Scope Risk" 注释

**Assertions:**
- [ ] PR-SCOPE 顾虑在 "May I write" 询问前显示给用户
- [ ] Skill 在未呈现范围顾虑前不写入概念
- [ ] 如果用户接受：范围顾虑被记录在概念文件中
- [ ] Skill 不会因 PR-SCOPE CONCERNS 自动拒绝概念（用户决定）

---

## 协议合规

- [ ] 在用户提交前展示 2-4 个带有优缺点对比的概念选项
- [ ] 用户在 director gate 被调用前确认概念方向
- [ ] 在 full 模式下所有 4 个 director gate 并行派生
- [ ] 在 lean AND solo 模式下所有 4 个 gate 被跳过——每个按名称注明
- [ ] 在写入前询问 "May I write `design/gdd/game-concept.md`?"
- [ ] 以到 `/map-systems` 的下一步交接结束

---

## 覆盖说明

- AD-CONCEPT-VISUAL gate（art director 可行性）与其他 3 个 gate 分在一组并行派生——未单独进行 fixture 测试。
- 迭代概念完善循环（用户拒绝所有选项，skill 生成新选项）未进行 fixture 测试——它遵循与选项选择阶段相同的模式。
- game-concept.md 文档结构（必需部分）在技能正文中定义，不在测试断言中重新枚举。
