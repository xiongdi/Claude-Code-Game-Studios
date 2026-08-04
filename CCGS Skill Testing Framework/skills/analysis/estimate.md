# 技能测试规格: /estimate

## 技能摘要

`/estimate` 使用相对大小量表（S / M / L / XL）估算任务或 story 的工作量，基于 story 复杂度、验收标准数量和过去 sprint 文件中的历史 sprint 速度。估算结果为建议性，从不自动写入。不触发任何 director gate。判定结果是工作量范围，而非 pass/fail — 每次运行都会产生估算。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含大小标签：S、M、L、XL（此 skill 的"判定"等价物）
- [ ] 不要求 "May I write" 语言（仅建议性输出）
- [ ] 具有下一步交接说明（如何在 sprint 规划中使用估算）

---

## 导演门控检查

无。估算是建议性信息技能；不触发任何 gate。

---

## 测试用例

### Case 1: Happy Path — 技术栈清晰明确的 story

**Fixture:**
- `production/epics/combat/story-hitbox-detection.md` 存在，包含：
  - 4 个清晰的验收标准
  - ADR 引用（Accepted 状态）
  - story 正文中无 "unknown" 或 "TBD" 语言
- `production/sprints/sprint-003.md` 到 `sprint-005.md` 存在，包含速度数据
- 技术栈为 GDScript（团队根据 sprint 历史充分了解）

**Input:** `/estimate production/epics/combat/story-hitbox-detection.md`

**Expected behavior:**
1. Skill 读取 story 文件 — 评估清晰度、AC 数量、技术栈
2. Skill 读取 sprint 历史以确定平均速度
3. Skill 输出估算：M（1–2 天）及理由
4. 不写入任何文件

**Assertions:**
- [ ] 对于技术清晰、范围明确的 story，估算为 M
- [ ] 理由引用 AC 数量、技术栈熟悉度和速度数据
- [ ] 估算以范围呈现（如 "1–2 天"），而非单一数值
- [ ] 不写入任何文件

---

### Case 2: High Uncertainty — 未知系统，尚无 ADR

**Fixture:**
- `production/epics/online/story-lobby-matchmaking.md` 存在，包含：
  - 2 个模糊的验收标准（使用 "should" 和 "TBD"）
  - 无 ADR 引用 — matchmaking 架构尚未决定
  - 引用新子系统（"online/matchmaking"），无现有源文件

**Input:** `/estimate production/epics/online/story-lobby-matchmaking.md`

**Expected behavior:**
1. Skill 读取 story — 发现模糊 AC、无 ADR、无现有源文件
2. Skill 标记多个不确定性因素
3. 估算为 L–XL，附带明确风险说明："Estimate range is wide due to architectural unknowns"
4. Skill 建议开发前先创建 ADR

**Assertions:**
- [ ] 存在重大未知时估算为 L 或 XL（非 S 或 M）
- [ ] 风险说明解释导致范围宽的具体未知因素
- [ ] 输出建议先解决架构问题
- [ ] 不写入任何文件

---

### Case 3: No Sprint Velocity Data — 使用保守默认值

**Fixture:**
- story 文件存在且定义明确
- `production/sprints/` 为空 — 无历史 sprint

**Input:** `/estimate production/epics/core/story-save-load.md`

**Expected behavior:**
1. Skill 读取 story — 评估复杂度
2. Skill 尝试读取 sprint 速度数据 — 未找到
3. Skill 注明："No sprint history found — using conservative defaults for velocity"
4. 使用默认假设计算估算（如 1 story point = 1 天）
5. 不写入任何文件

**Assertions:**
- [ ] 没有 sprint 历史时 Skill 不出错
- [ ] 输出明确注明正在使用保守默认值
- [ ] 仍然产生估算（不被缺失速度数据阻塞）
- [ ] 保守默认值产生更高（非更低）的估算范围

---

### Case 4: Multiple Stories — 每个单独估算加 sprint 总计

**Fixture:**
- 用户提供 sprint 文件：`production/sprints/sprint-007.md`，包含 4 个 story
- sprint 历史存在（3 个先前 sprint）

**Input:** `/estimate production/sprints/sprint-007.md`

**Expected behavior:**
1. Skill 读取 sprint 文件 — 识别 4 个 story
2. Skill 单独估算每个 story：S、M、M、L
3. Skill 计算 sprint 总计：约 6–8 story points
4. Skill 先呈现每个 story 的估算，然后是 sprint 总计
5. 不写入任何文件

**Assertions:**
- [ ] 每个 story 获得自己的估算标签
- [ ] sprint 总计在单独估算后呈现
- [ ] 总计是从单独范围派生的总和范围
- [ ] Skill 处理 sprint 文件（不仅是单个 story 文件）作为输入

---

### Case 5: Gate Compliance — 无 gate；估算为信息性

**Fixture:**
- story 文件存在，复杂度中等
- `review-mode.txt` 内容为 `full`

**Input:** `/estimate production/epics/core/story-item-pickup.md`

**Expected behavior:**
1. Skill 读取 story 和 sprint 历史；计算估算
2. 任何审查模式下都不触发 director gate
3. 估算仅作为建议性输出呈现
4. Skill 注明："Use this estimate in /sprint-plan when selecting stories for the next sprint"

**Assertions:**
- [ ] 无论审查模式如何，不触发任何 director gate
- [ ] 输出纯信息性 — 无批准或写入提示
- [ ] 下一步建议引用 `/sprint-plan`
- [ ] 估算不随审查模式变化

---

## Protocol Compliance

- [ ] 估算前读取 story 文件
- [ ] 可用时读取 sprint 速度历史
- [ ] 产生工作量范围（S/M/L/XL），而非单一数字
- [ ] 不写入任何文件
- [ ] 不触发任何 director gate
- [ ] 始终产生估算（从不被缺失数据阻塞；使用默认值代替）

---

## 覆盖说明

- 此 skill 不产生 PASS/FAIL 判定；此处的"判定"是工作量范围本身。测试断言关注范围的准确性和理由的质量，而非二元结果。
- 团队特定的速度校准（"M" 对团队意味着什么）是此处未测试的实现细节；它通过 sprint 历史配置。
