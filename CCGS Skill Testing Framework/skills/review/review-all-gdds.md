# Skill Test Spec: /review-all-gdds

## Skill Summary

`/review-all-gdds` 是一个 Opus 层级的 skill，对 `design/gdd/` 中的所有文件执行整体跨 GDD 审查。它并行运行两个互补的审查阶段：Phase 1 检查一致性（矛盾、公式不匹配、过时引用、竞争所有权），Phase 2 检查设计理论（主导策略、pillar drift、认知超载、经济失衡）。由于两个阶段是独立的，它们同时派生以节省时间。该 skill 产生 CONSISTENT / MINOR ISSUES / MAJOR ISSUES 的裁定，并且是只读的——未经用户明确批准不写入任何文件。

该 skill 本身就是管道中的整体审查 gate。它在各个 GDD 完成后、架构工作开始前调用。它不派生任何 director gate agent（它本身就是 director 级别的审查）。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥5 个 phase 标题（复杂多阶段 skill）
- [ ] 包含裁定关键词：CONSISTENT、MINOR ISSUES、MAJOR ISSUES
- [ ] 不要求 "May I write" 语言（只读 skill）
- [ ] 末尾有下一步交接说明
- [ ] 记录并行阶段派生（Phase 1 和 Phase 2 是独立的）

---

## Director Gate Checks

无 director gate——此 skill 不派生任何 director gate agent。它本身就是整体审查；委托给 director gate 会产生循环依赖。

---

## Test Cases

### Case 1: Happy Path——无矛盾的清洁 GDD 集

**Fixture:**
- `design/gdd/` 包含 ≥3 个系统 GDD
- 所有 GDD 内部一致：无公式矛盾、无竞争所有权、无过时引用
- 所有 GDD 与 `design/gdd/game-pillars.md` 中定义的 pillar 对齐

**Input:** `/review-all-gdds`

**Expected behavior:**
1. Skill 读取 `design/gdd/` 中的所有 GDD 文件
2. Phase 1（一致性扫描）和 Phase 2（设计理论检查）并行派生
3. Phase 1 未发现矛盾、公式不匹配、所有权矛盾
4. Phase 2 未发现 pillar drift、主导策略、认知超载
5. Skill 输出具有 0 个阻塞问题的结构化发现表
6. 裁定：CONSISTENT

**Assertions:**
- [ ] 两个审查阶段并行派生（非顺序）
- [ ] 输出包含发现表（即使为空——显示 "No issues found"）
- [ ] 当未发现矛盾时，裁定为 CONSISTENT
- [ ] Skill 未经用户批准不写入任何文件
- [ ] 存在到 `/architecture-review` 或 `/create-architecture` 的下一步交接说明

---

### Case 2: Failure Path——两个 GDD 之间的矛盾规则

**Fixture:**
- GDD-A 定义了下限值（例如 "minimum [output] is [N]"）
- GDD-B 陈述了绕过该下限的机制（例如 "[mechanic] can reduce [output] to 0"）
- 两个 GDD 在其他方面完整且有效

**Input:** `/review-all-gdds`

**Expected behavior:**
1. Phase 1（一致性扫描）检测到 GDD-A 和 GDD-B 之间的矛盾
2. 矛盾报告包含：两个文件名、具体矛盾规则和严重性 HIGH
3. 裁定：MAJOR ISSUES
4. 交接说明指导用户在继续前解决矛盾并重新运行

**Assertions:**
- [ ] 裁定为 MAJOR ISSUES（不是 CONSISTENT 或 MINOR ISSUES）
- [ ] 矛盾条目中命名了两个 GDD 文件名
- [ ] 具体矛盾规则被引用或描述（不是模糊的 "conflict found"）
- [ ] 问题被分类为严重性 HIGH（阻塞性）
- [ ] Skill 不自动解决矛盾

---

### Case 3: Partial Path——单个 GDD 具有孤立依赖引用

**Fixture:**
- GDD-A 在其 Dependencies 章节中列出了指向 "system-B" 的依赖
- `design/gdd/` 中不存在 system-B 的 GDD
- 其他所有 GDD 一致

**Input:** `/review-all-gdds`

**Expected behavior:**
1. Phase 1 检测到 GDD-A 中的孤立依赖引用
2. 问题报告为：DEPENDENCY GAP——GDD-A 引用了没有 GDD 的 system-B
3. 未发现其他矛盾
4. 裁定：MINOR ISSUES（依赖差距是建议性的，本身不是阻塞性的）

**Assertions:**
- [ ] 裁定为 MINOR ISSUES（不是 MAJOR ISSUES），针对单个孤立引用
- [ ] 报告了具体的 GDD 文件名和缺失的依赖名称
- [ ] Skill 建议运行 `/design-system system-B` 来解决差距
- [ ] Skill 不跳过或静默忽略缺失的依赖

---

### Case 4: Edge Case——未找到 GDD 文件

**Fixture:**
- `design/gdd/` 目录为空或不存在
- 不存在 GDD 文件

**Input:** `/review-all-gdds`

**Expected behavior:**
1. Skill 尝试读取 `design/gdd/` 中的文件
2. 未找到文件——skill 输出带有指导的错误
3. Skill 建议在重新运行前运行 `/brainstorm` 和 `/design-system`
4. Skill 不产生裁定（CONSISTENT / MINOR ISSUES / MAJOR ISSUES）

**Assertions:**
- [ ] Skill 在未找到 GDD 时输出明确的错误消息
- [ ] 当目录为空时不产生裁定
- [ ] Skill 推荐正确的下一步操作（`/brainstorm` 或 `/design-system`）
- [ ] Skill 不崩溃或不产生部分报告

---

### Case 5: Director Gate——无论审查模式如何都不派生 gate

**Fixture:**
- `design/gdd/` 包含 ≥2 个一致的系统 GDD
- `production/session-state/review-mode.txt` 存在，内容为 `full`

**Input:** `/review-all-gdds`

**Expected behavior:**
1. Skill 读取所有 GDD 并运行两个审查阶段
2. Skill 不读取 `review-mode.txt`
3. Skill 不派生任何 director gate agent（CD-、TD-、PR-、AD- 前缀）
4. Skill 完成并正常输出其裁定
5. 审查模式设置对此 skill 的行为无影响

**Assertions:**
- [ ] 在任何时候都不派生 director gate agent
- [ ] Skill 不读取 `production/session-state/review-mode.txt`
- [ ] 输出不包含任何 "Gate: [GATE-ID]" 或被跳过的 gate 条目
- [ ] 无论审查模式如何，skill 都产生裁定
- [ ] R4 指标：此 skill 的 gate 计数在所有模式下 = 0

---

## Protocol Compliance

- [ ] Phase 1（一致性）和 Phase 2（设计理论）并行派生——非顺序
- [ ] 未经 "May I write" 批准不写入任何文件
- [ ] 在任何写入请求前显示发现表
- [ ] 裁定严格为以下之一：CONSISTENT、MINOR ISSUES、MAJOR ISSUES
- [ ] 以适当的交接说明结束：MAJOR ISSUES → 修复并重新运行；MINOR ISSUES → 可继续但需注意；CONSISTENT → `/create-architecture`

---

## Coverage Notes

- 经济平衡分析（source/sink 循环）需要跨 GDD 资源数据——由 Case 2 结构性覆盖
  （矛盾检测模式相同）。
- 设计理论阶段（Phase 2）检查包括主导策略检测和认知超载，不单独进行 fixture 测试——
  它们遵循与一致性检查相同的模式，并通过 pillar drift 案例结构进行验证。
- `since-last-review` 范围模式不在此测试——这是一个运行时问题。
