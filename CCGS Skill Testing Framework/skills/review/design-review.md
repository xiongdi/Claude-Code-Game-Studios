# Skill Test Spec: /design-review

## Skill Summary

`/design-review` 读取游戏设计文档（GDD）并根据项目的 8 章节设计标准（Overview、Player Fantasy、Detailed Rules、Formulas、Edge Cases、Dependencies、Tuning Knobs、Acceptance Criteria）对其进行评估。它检查内部一致性、可实现性和跨系统矛盾。它产生 APPROVED、NEEDS REVISION 或 MAJOR REVISION NEEDED 的裁定。它是一个只读 skill（无文件写入），作为 `context: fork` subagent 运行。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题或编号步骤
- [ ] 包含裁定关键词：APPROVED、NEEDS REVISION、MAJOR REVISION NEEDED
- [ ] 不要求 "May I write" 语言（只读 skill——`allowed-tools` 排除 Write/Edit）
- [ ] 输出格式已记录（审查模板在 skill 正文中显示）

---

## Test Cases

### Case 1: Happy Path——完整的 GDD，所有 8 个章节存在

**Fixture:**
- `design/gdd/light-manipulation.md` 存在（使用 `_fixtures/minimal-game-concept.md`
  作为替身——表示具有所有必需内容的完整文档）
- 所有 8 个必需章节都填写了实质性内容
- Formulas 章节包含至少一个具有已定义变量的公式
- Acceptance Criteria 章节包含至少 3 个可测试标准

**Input:** `/design-review design/gdd/light-manipulation.md`

**Expected behavior:**
1. Skill 完整读取目标文档
2. Skill 读取 CLAUDE.md 获取项目上下文和标准
3. Skill 评估所有 8 个必需章节（存在/缺失检查）
4. Skill 检查内部一致性（公式与描述的行为匹配）
5. Skill 检查可实现性（规则足够精确可编码）
6. Skill 输出具有逐章节状态的结构化审查
7. Skill 输出 APPROVED 裁定

**Assertions:**
- [ ] Skill 在产生任何输出前读取目标文件
- [ ] 输出包含显示 X/8 章节存在的 "Completeness" 章节
- [ ] 输出包含 "Internal Consistency" 章节
- [ ] 输出包含 "Implementability" 章节
- [ ] 输出以裁定行结束：APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED
- [ ] 当所有 8 个章节存在且一致时给出 APPROVED 裁定

---

### Case 2: Failure Path——不完整的 GDD（4/8 章节）

**Fixture:**
- `design/gdd/light-manipulation.md` 存在，内容来自
  `tests/skills/_fixtures/incomplete-gdd.md`（8 个章节中 4 个已填写；
  Formulas、Edge Cases、Tuning Knobs、Acceptance Criteria 缺失）

**Input:** `/design-review design/gdd/light-manipulation.md`

**Expected behavior:**
1. Skill 读取文档
2. Skill 识别 4 个缺失章节
3. Skill 输出 "Completeness: 4/8 sections present"
4. Skill 具体列出哪 4 个章节缺失
5. Skill 输出 MAJOR REVISION NEEDED 裁定（不是 APPROVED 或 NEEDS REVISION）

**Assertions:**
- [ ] 输出在完整性章节显示 "4/8"（不是更高数字）
- [ ] 输出明确命名每个缺失章节（Formulas、Edge Cases、Tuning Knobs、Acceptance Criteria）
- [ ] 当 ≥3 个章节缺失时，裁定为 MAJOR REVISION NEEDED（不是 APPROVED 或 NEEDS REVISION）
- [ ] 输出不暗示文档已准备好实现
- [ ] Skill 不写入任何文件（只读执行）

---

### Case 3: Partial Path——7/8 章节，轻微不一致

**Fixture:**
- GDD 除 Formulas 外具有所有章节
- 描述的行为提及数值但未定义公式
- Acceptance Criteria 存在但模糊（"feels good" 而非可测量的）

**Input:** `/design-review design/gdd/[document].md`

**Expected behavior:**
1. Skill 识别缺失的 Formulas 章节
2. Skill 将模糊的验收标准标记为可实现性问题
3. Skill 输出 NEEDS REVISION 裁定（不是 APPROVED，不是 MAJOR REVISION NEEDED）
4. Skill 为每个问题提供具体修复说明

**Assertions:**
- [ ] 对于 7/8 有问题的情况，裁定为 NEEDS REVISION（不是 APPROVED，不是 MAJOR REVISION NEEDED）
- [ ] 输出具体识别缺失的 Formulas 章节
- [ ] 输出将模糊的验收标准标记为可实现性差距
- [ ] 每个标记的问题都有具体、可操作的修复说明

---

### Case 4: Edge Case——文件未找到

**Fixture:**
- 提供的路径在项目中不存在

**Input:** `/design-review design/gdd/nonexistent.md`

**Expected behavior:**
1. Skill 尝试读取文件
2. 文件未找到
3. Skill 输出错误消息，命名缺失的文件
4. Skill 建议检查路径或列出 `design/gdd/` 中的文件
5. Skill 不产生裁定

**Assertions:**
- [ ] Skill 在文件未找到时输出明确错误
- [ ] Skill 在文件缺失时不输出 APPROVED、NEEDS REVISION 或 MAJOR REVISION NEEDED
- [ ] Skill 建议纠正操作（检查路径，列出可用 GDD）

---

### Case 5: Director Gate——无论审查模式如何都不派生 gate

**Fixture:**
- `design/gdd/light-manipulation.md` 存在，具有所有 8 个章节
- `production/session-state/review-mode.txt` 存在，内容为 `full`（最宽松的模式）

**Input:** `/design-review design/gdd/light-manipulation.md`（full 审查模式激活）

**Expected behavior:**
1. Skill 读取 GDD 文档
2. Skill 不读取 `review-mode.txt`——此 skill 没有 director gate
3. Skill 正常产生审查输出
4. 在任何时候都不派生 director gate agent
5. 裁定为 APPROVED（fixture 中所有 8 个章节都存在）

**Assertions:**
- [ ] Skill 不派生任何 director gate agent（CD-、TD-、PR-、AD- 前缀的 agent）
- [ ] Skill 不读取 `review-mode.txt` 或等效的模式文件
- [ ] `--review` 标志或 `full` 模式状态对 director 是否派生无影响
- [ ] 输出不包含任何 "Gate: [GATE-ID]" 条目
- [ ] Skill 本身就是审查——它不将审查委托给 director

---

## Protocol Compliance

- [ ] 不使用 Write 或 Edit 工具（只读 skill）
- [ ] 在任何裁定前呈现完整发现
- [ ] 在产生输出前不请求批准（无写入需要批准）
- [ ] 以推荐的下一步结束（例如，修复问题并重新运行，或继续到 `/map-systems`）

---

## Coverage Notes

- 跨系统一致性检查（skill 自身 phase 列表中的 Case 3）不在此直接测试，
  因为它需要多个 GDD 文件进行比较；这由 `/review-all-gdds` spec 覆盖。
- Skill 的 `context: fork` 行为（作为 subagent 运行）不在 spec 级别测试——
  这是手动验证的运行时行为。
- 涉及非常大 GDD 文件的性能和边缘情况不在范围内。
