# Skill Test Spec: /scope-check

## Skill Summary

`/scope-check` 是 Haiku 层级的只读技能，用于分析功能、sprint 或 story 的范围蔓延风险。它读取 sprint 和 story 文件，并与活动里程碑目标进行比较。设计用于规划前或规划期间的快速、低成本检查。不触发任何 director gate。不写入任何文件。判定结果：ON SCOPE、CONCERNS 或 SCOPE CREEP DETECTED。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：ON SCOPE、CONCERNS、SCOPE CREEP DETECTED
- [ ] 不要求 "May I write" 语言（只读技能）
- [ ] 具有下一步交接说明（基于判定如何处理）

---

## Director Gate Checks

无。范围检查是只读建议性技能；不触发任何 gate。

---

## Test Cases

### Case 1: Happy Path — Sprint story 与里程碑目标对齐

**Fixture:**
- `production/milestones/milestone-03.md` 列出 3 个目标：战斗系统、敌人 AI、关卡加载
- `production/sprints/sprint-006.md` 包含 5 个 story，全部标记到 3 个目标之一
- `production/session-state/active.md` 引用 milestone-03 为活动里程碑

**Input:** `/scope-check`

**Expected behavior:**
1. Skill 从 milestone-03 读取活动里程碑目标
2. Skill 读取 sprint-006 story 并对照里程碑目标检查每个
3. 所有 5 个 story 映射到 3 个目标之一
4. Skill 输出映射表：story → 里程碑目标
5. 判定为 ON SCOPE

**Assertions:**
- [ ] 输出中将每个 story 映射到里程碑目标
- [ ] 所有 story 映射到里程碑目标时判定为 ON SCOPE
- [ ] 不写入任何文件
- [ ] Skill 不修改 sprint 或里程碑文件

---

### Case 2: Scope Creep Detected — 引入里程碑外系统的 story

**Fixture:**
- `production/milestones/milestone-03.md` 目标：战斗、敌人 AI、关卡加载
- `production/sprints/sprint-006.md` 包含 5 个 story：
  - 3 个 story 映射到里程碑目标
  - 2 个 story 引用 "online leaderboard" 和 "achievement system"（不在 milestone-03 中）

**Input:** `/scope-check`

**Expected behavior:**
1. Skill 读取里程碑目标和 sprint story
2. Skill 识别 2 个没有匹配里程碑目标的 story
3. Skill 命名范围外的 story："Online Leaderboard Feature"、"Achievement System Setup"
4. 判定为 SCOPE CREEP DETECTED

**Assertions:**
- [ ] 范围外的 story 在输出中明确命名
- [ ] 任何 story 无里程碑目标匹配时判定为 SCOPE CREEP DETECTED
- [ ] Skill 不自动移除 story — 结果为建议性
- [ ] 输出建议将范围外的 story 推迟到后续里程碑

---

### Case 3: No Milestone Defined — CONCERNS；无法验证范围

**Fixture:**
- `production/session-state/active.md` 无里程碑引用
- `production/milestones/` 目录存在但为空
- `production/sprints/sprint-006.md` 有 4 个 story

**Input:** `/scope-check`

**Expected behavior:**
1. Skill 读取 active.md — 未找到里程碑引用
2. Skill 检查 `production/milestones/` — 未找到里程碑文件
3. Skill 输出："No active milestone defined — scope cannot be validated"
4. 判定为 CONCERNS

**Assertions:**
- [ ] 未定义里程碑时 Skill 不出错
- [ ] 输出明确说明范围验证需要里程碑引用
- [ ] 无数据时判定为 CONCERNS（非 ON SCOPE 或 SCOPE CREEP DETECTED）
- [ ] 输出建议运行 `/milestone-review` 或创建里程碑

---

### Case 4: Single Story Check — 对照其父 epic 评估

**Fixture:**
- 用户定位单个 story：`production/epics/combat/story-parry-timing.md`
- Story 引用父 epic：`epic-combat.md`
- `production/epics/combat/epic-combat.md` 范围："melee combat mechanics"
- Story 标题："Implement parry timing window" — 匹配 epic 范围

**Input:** `/scope-check production/epics/combat/story-parry-timing.md`

**Expected behavior:**
1. Skill 读取指定的 story 文件
2. Skill 读取父 epic 获取范围定义
3. Skill 对照 epic 范围评估 story — "parry timing" 匹配 "melee combat"
4. 判定为 ON SCOPE

**Assertions:**
- [ ] 接受单文件参数（story 路径，非 sprint）
- [ ] Skill 读取 story 文件中引用的父 epic
- [ ] 在单 story 模式下 story 对照 epic 范围（非里程碑范围）评估
- [ ] story 匹配 epic 范围时判定为 ON SCOPE

---

### Case 5: Gate Compliance — 无 gate；PR 可单独咨询

**Fixture:**
- Sprint 有 2 个 SCOPE CREEP story 和 3 个 ON SCOPE story
- `review-mode.txt` 内容为 `full`

**Input:** `/scope-check`

**Expected behavior:**
1. Skill 读取里程碑和 sprint；识别 2 个范围蔓延项
2. 无论审查模式如何，不触发任何 director gate
3. Skill 以 SCOPE CREEP DETECTED 判定呈现结果
4. 输出注明："Consider raising scope concerns with the Producer before sprint begins"
5. Skill 结束时不写入任何文件

**Assertions:**
- [ ] 任何审查模式下都不触发 director gate
- [ ] 建议（非强制）咨询 Producer
- [ ] 不写入任何文件
- [ ] 判定为 SCOPE CREEP DETECTED

---

## Protocol Compliance

- [ ] 分析前读取里程碑目标和 sprint/story 文件
- [ ] 将每个 story 映射到里程碑目标（或标记为未映射）
- [ ] 不写入任何文件
- [ ] 不触发任何 director gate
- [ ] 在 Haiku 模型层级运行（快速、低成本）
- [ ] 判定为以下之一：ON SCOPE、CONCERNS、SCOPE CREEP DETECTED

---

## Coverage Notes

- sprint 文件本身不存在的情况未测试；skill 会输出 CONCERNS 判定并附带缺失 sprint 数据的消息。
- 部分范围重叠（story 触及里程碑目标但也引入新范围）未显式测试；实现可能将其分类为 CONCERNS 而非 SCOPE CREEP DETECTED。
