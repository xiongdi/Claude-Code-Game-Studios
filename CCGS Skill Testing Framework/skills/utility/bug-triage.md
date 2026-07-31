# Skill 测试规格：/bug-triage

## Skill 摘要

`/bug-triage` 读取 `production/bugs/` 中所有未关闭的 bug 报告，并生成一个
按严重级别排序的优先级分类表（CRITICAL → HIGH → MEDIUM → LOW）。
它在 Haiku 模型上运行（只读、格式化/排序任务），不写入任何文件——分类输出是对话式的。
该 skill 标记缺少复现步骤的 bug，并通过比较标题和受影响的系统来识别可能的重复项。

判定结果始终是 TRIAGED——该 skill 是建议性和信息性的。
不适用 director 关卡。输出旨在帮助 producer 或 QA 负责人
确定接下来要处理哪些 bug 的优先级。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：TRIAGED
- [ ] 不包含"可以写入吗"语言（skill 是只读的）
- [ ] 有下一步交接（例如，`/bug-report` 创建新报告，`/hotfix` 处理严重 bug）

---

## Director 关卡检查

无。`/bug-triage` 是只读的建议性 skill。不适用 director 关卡。

---

## 测试用例

### 用例 1：正常路径——5 个不同严重级别的 bug，生成排序表

**Fixture：**
- `production/bugs/` 包含 5 个 bug 报告文件：
  - bug-2026-03-10-audio-crash.md (CRITICAL)
  - bug-2026-03-12-score-overflow.md (HIGH)
  - bug-2026-03-14-ui-overlap.md (MEDIUM)
  - bug-2026-03-15-typo-tutorial.md (LOW)
  - bug-2026-03-16-vfx-flicker.md (HIGH)

**输入：** `/bug-triage`

**预期行为：**
1. Skill 读取所有 5 个 bug 报告文件
2. Skill 从每个报告中提取严重级别、标题、系统和复现状态
3. Skill 生成按以下顺序排序的分类表：CRITICAL 优先，然后 HIGH、MEDIUM、LOW
4. 同一严重级别内，bug 按日期排序（最早的在前）
5. 判定为 TRIAGED

**断言：**
- [ ] 分类表恰好有 5 行
- [ ] CRITICAL bug 出现在两个 HIGH bug 之前
- [ ] HIGH bug 出现在 MEDIUM 和 LOW bug 之前
- [ ] 判定为 TRIAGED
- [ ] 未写入任何文件

---

### 用例 2：未找到 Bug 报告——引导运行 /bug-report

**Fixture：**
- `production/bugs/` 目录存在但为空（或不存在）

**输入：** `/bug-triage`

**预期行为：**
1. Skill 扫描 `production/bugs/` 并找到无报告
2. Skill 输出："No open bug reports found in production/bugs/"
3. Skill 建议运行 `/bug-report` 创建 bug 报告
4. 不生成分类表

**断言：**
- [ ] 输出明确说明未找到 bug
- [ ] 建议 `/bug-report` 作为下一步
- [ ] Skill 不会报错——优雅处理空目录
- [ ] 判定为 TRIAGED（附带"未找到 bug"上下文）

---

### 用例 3：Bug 缺少复现步骤——标记为 NEEDS REPRO INFO

**Fixture：**
- `production/bugs/` 包含 3 个 bug 报告；其中一个的"Repro Steps"部分为空

**输入：** `/bug-triage`

**预期行为：**
1. Skill 读取所有 3 个报告
2. Skill 检测到没有复现步骤的报告
3. 该 bug 在分类表中出现并带有 `NEEDS REPRO INFO` 标签
4. 其他 bug 正常分类
5. 判定为 TRIAGED

**断言：**
- [ ] `NEEDS REPRO INFO` 标签出现在缺少复现步骤的 bug 旁边
- [ ] 被标记的 bug 仍包含在表中（未排除）
- [ ] 其他 bug 不受影响
- [ ] 判定为 TRIAGED

---

### 用例 4：可能的重复 Bug——在分类输出中标记

**Fixture：**
- `production/bugs/` 包含 2 个标题相似的 bug 报告：
  - bug-2026-03-18-player-fall-through-floor.md
  - bug-2026-03-20-player-clips-through-floor.md
  - 两者都影响 "Physics" 系统，严重级别相同

**输入：** `/bug-triage`

**预期行为：**
1. Skill 读取两个报告并检测到相似标题 + 相同系统 + 相同严重级别
2. 两个 bug 都包含在分类表中
3. 每个都标记为 `POSSIBLE DUPLICATE` 并交叉引用另一个报告
4. 不合并或删除任何 bug——标记是建议性的
5. 判定为 TRIAGED

**断言：**
- [ ] 两个 bug 都出现在表中（未合并）
- [ ] 两者都标记为 `POSSIBLE DUPLICATE`
- [ ] 每个都交叉引用另一个（通过文件名或标题）
- [ ] 判定为 TRIAGED

---

### 用例 5：Director 关卡检查——无关卡；分类是建议性的

**Fixture：**
- `production/bugs/` 包含任意数量的报告

**输入：** `/bug-triage`

**预期行为：**
1. Skill 生成分类表
2. 不派生任何 director agent
3. 输出中不出现 gate ID
4. 不调用写入工具

**断言：**
- [ ] 未调用 director 关卡
- [ ] 未调用写入工具
- [ ] 不出现 gate 跳过消息
- [ ] 判定为 TRIAGED，无任何关卡检查

---

## 协议合规性

- [ ] 生成表之前读取 `production/bugs/` 中的所有文件
- [ ] 按严重级别排序（CRITICAL → HIGH → MEDIUM → LOW）
- [ ] 标记缺少复现步骤的 bug
- [ ] 按标题/系统相似性标记可能的重复项
- [ ] 不写入任何文件
- [ ] 所有情况下判定均为 TRIAGED（即使为空）

---

## 覆盖说明

- bug 报告格式错误（完全缺少严重级别字段）的情况未进行 fixture 测试；
  skill 会将其标记为 `UNKNOWN SEVERITY` 并排在表末尾。
- 状态转换（将 bug 标记为已解决）超出此 skill 范围——
  bug-triage 是只读的。
- 重复检测启发式方法（标题相似性 + 相同系统）是近似值；
  精确匹配逻辑在 skill 主体中定义。
