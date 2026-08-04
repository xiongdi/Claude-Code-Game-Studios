# 技能测试规格: /consistency-check

## 技能摘要

`/consistency-check` 扫描 `design/gdd/` 中的所有 GDD，检查文档之间的内部冲突。它生成一个结构化的结果表，列名为：System A vs System B、Conflict Type、Severity（HIGH / MEDIUM / LOW）。冲突类型包括：公式不匹配、所有权竞争、过时引用和依赖缺口。

Skill 在分析期间为只读。没有 director gate。如果用户请求，可选择性地将一致性报告写入 `design/consistency-report-[date].md`，但 skill 会先询问 "May I write"。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：CONSISTENT、CONFLICTS FOUND、DEPENDENCY GAP
- [ ] 分析期间不要求 "May I write" 语言（只读扫描）
- [ ] 末尾具有下一步交接说明
- [ ] 记录报告写入是可选的且需要批准

---

## 导演门控检查

无 director gate — 此 skill 不生成任何 director gate agent。一致性检查是机械扫描；扫描本身不需要创意或技术负责人审查。

---

## 测试用例

### Case 1: Happy Path — 4 个 GDD 无冲突

**Fixture:**
- `design/gdd/` 恰好包含 4 个系统 GDD
- 所有 GDD 公式一致（无重叠变量有不同值）
- 没有两个 GDD 声明对同一游戏实体或机制的所有权
- 所有依赖引用都指向存在的 GDD

**Input:** `/consistency-check`

**Expected behavior:**
1. Skill 读取 `design/gdd/` 中的所有 4 个 GDD
2. 运行跨 GDD 一致性检查（公式、所有权、引用）
3. 未发现冲突
4. 输出结构化结果表，显示 0 个问题
5. 判定：CONSISTENT

**Assertions:**
- [ ] 输出前读取所有 4 个 GDD
- [ ] 结果表存在（即使为空 — 显示 "No conflicts found"）
- [ ] 无冲突时判定为 CONSISTENT
- [ ] Skill 未经用户批准不写入任何文件
- [ ] 存在下一步交接说明

---

### Case 2: Failure Path — 两个 GDD 有冲突的伤害公式

**Fixture:**
- GDD-A 定义伤害公式：`damage = attack * 1.5`
- GDD-B 对同一实体类型定义伤害公式：`damage = attack * 2.0`
- 两个 GDD 引用相同的 "attack" 变量

**Input:** `/consistency-check`

**Expected behavior:**
1. Skill 读取所有 GDD 并检测到公式不匹配
2. 结果表包含条目：GDD-A vs GDD-B | Formula Mismatch | HIGH
3. 显示具体冲突的公式（不仅是"存在公式冲突"）
4. 判定：CONFLICTS FOUND

**Assertions:**
- [ ] 判定为 CONFLICTS FOUND（非 CONSISTENT）
- [ ] 冲突条目命名两个 GDD 文件名
- [ ] 冲突类型为 "Formula Mismatch"
- [ ] 直接公式矛盾严重级别为 HIGH
- [ ] 结果表中显示两个冲突的公式
- [ ] Skill 不自动解决冲突

---

### Case 3: Partial Path — GDD 引用了没有 GDD 的系统

**Fixture:**
- GDD-A 的 Dependencies 部分列出 "system-B" 作为依赖
- `design/gdd/` 中不存在 system-B 的 GDD
- 其他 GDD 一致

**Input:** `/consistency-check`

**Expected behavior:**
1. Skill 读取所有 GDD 并检查依赖引用
2. GDD-A 对 "system-B" 的引用无法解析 — 不存在对应的 GDD
3. 结果表包含：GDD-A vs (missing) | Dependency Gap | MEDIUM
4. 判定：DEPENDENCY GAP（非 CONSISTENT，非 CONFLICTS FOUND）

**Assertions:**
- [ ] 判定为 DEPENDENCY GAP（区别于 CONSISTENT 和 CONFLICTS FOUND）
- [ ] 结果条目命名 GDD-A 和缺失的 system-B
- [ ] 未解析依赖引用严重级别为 MEDIUM
- [ ] Skill 建议运行 `/design-system system-B` 创建缺失的 GDD

---

### Case 4: Edge Case — 未找到 GDD

**Fixture:**
- `design/gdd/` 目录为空或不存在

**Input:** `/consistency-check`

**Expected behavior:**
1. Skill 尝试读取 `design/gdd/` 中的文件
2. 未找到 GDD 文件
3. Skill 输出错误："No GDDs found in `design/gdd/`. Run `/design-system` to create GDDs first."
4. 不生成结果表
5. 不输出判定

**Assertions:**
- [ ] 未找到 GDD 时 Skill 输出明确的错误消息
- [ ] 不产生判定（CONSISTENT / CONFLICTS FOUND / DEPENDENCY GAP）
- [ ] Skill 推荐正确的下一步操作（`/design-system`）
- [ ] Skill 不崩溃或不生成部分报告

---

### Case 5: Director Gate — 无 gate 生成；不读取 review-mode.txt

**Fixture:**
- `design/gdd/` 包含 ≥2 个 GDD
- `production/session-state/review-mode.txt` 存在，内容为 `full`

**Input:** `/consistency-check`

**Expected behavior:**
1. Skill 读取所有 GDD 并运行一致性扫描
2. Skill 不读取 `production/session-state/review-mode.txt`
3. 任何时候都不生成 director gate agent
4. 正常生成结果表和判定

**Assertions:**
- [ ] 不生成 director gate agent（无 CD-、TD-、PR-、AD- 前缀的 gate）
- [ ] Skill 不读取 `production/session-state/review-mode.txt`
- [ ] 输出不包含 "Gate: [GATE-ID]" 或 gate-skipped 条目
- [ ] 审查模式对此 skill 的行为无影响

---

## Protocol Compliance

- [ ] 生成结果表前读取所有 GDD
- [ ] 任何写入请求前完整显示结果表（如果请求报告）
- [ ] 判定恰好为以下之一：CONSISTENT、CONFLICTS FOUND、DEPENDENCY GAP
- [ ] 无 director gate — 不读取 review-mode.txt
- [ ] 报告写入（如果请求）受 "May I write" 批准门控
- [ ] 以适合判定的下一步交接说明结束

---

## 覆盖说明

- 此 skill 检查 GDD 之间的结构一致性。深度设计理论分析（pillar drift、主导策略）由 `/review-all-gdds` 处理。
- 公式冲突检测依赖于 GDD 之间一致的公式表示法 — 同一机制的非正式描述可能无法被检测到。
- 冲突严重级别规则（HIGH / MEDIUM / LOW）在 skill 主体中定义，此处不再列举。
