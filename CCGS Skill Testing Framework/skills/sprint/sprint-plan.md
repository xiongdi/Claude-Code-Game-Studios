# Skill 测试规格：/sprint-plan

## Skill 摘要

`/sprint-plan` 读取当前里程碑文件和积压 story，然后
生成一个新的编号 sprint，story 按实现层和优先级分数排序。
在 full 模式下，PR-SPRINT director 关卡在 sprint 草稿编译后运行
（producer 审查计划）。在 lean 和 solo 模式下，关卡被跳过。
该 skill 在持久化前询问"可以写入 `production/sprints/sprint-NNN.md` 吗？"。
判定结果：COMPLETE（sprint 已生成并写入）或 BLOCKED（因缺少数据或关卡失败无法继续）。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含"可以写入吗"语言（skill 写入 sprint 文件）
- [ ] 有下一步交接（sprint 写入后做什么）

---

## Director 关卡检查

| Gate ID   | 触发条件             | 模式守卫                  |
|-----------|----------------------|---------------------------|
| PR-SPRINT | sprint 草稿构建后    | full only（非 lean/solo）  |

---

## 测试用例

### 用例 1：正常路径——有 story 的积压生成 sprint

**Fixture：**
- `production/milestones/milestone-02.md` 存在，容量为 `10 story points`
- 积压包含跨 2 个 epic 的 5 个未开始 story，优先级混合
- `production/session-state/review-mode.txt` 包含 `full`
- 下一个 sprint 编号是 `003`（sprint 001 和 002 已存在）

**输入：** `/sprint-plan`

**预期行为：**
1. Skill 读取当前里程碑以获取容量和目标
2. Skill 读取积压中所有未开始的 story；按层 + 优先级排序
3. Skill 起草 sprint-003，story 在容量范围内
4. Skill 在调用关卡前向用户展示草稿
5. Skill 调用 PR-SPRINT 关卡（full 模式）；producer 批准
6. Skill 询问"可以写入 `production/sprints/sprint-003.md` 吗？"
7. 用户批准；文件写入

**断言：**
- [ ] Story 在优先级之前按实现层排序
- [ ] Sprint 草稿在任何写入或关卡调用前展示
- [ ] PR-SPRINT 关卡在 full 模式下草稿就绪后调用
- [ ] Skill 在写入 sprint 文件前询问"可以写入吗"
- [ ] 写入文件路径匹配 `production/sprints/sprint-003.md`
- [ ] 成功写入后判定为 COMPLETE

---

### 用例 2：阻塞路径——积压为空

**Fixture：**
- `production/milestones/milestone-02.md` 存在
- 任何 epic 积压中无未开始的 story

**输入：** `/sprint-plan`

**预期行为：**
1. Skill 读取积压——未发现未开始的 story
2. Skill 输出 "No unstarted stories in backlog"
3. Skill 建议运行 `/create-stories` 填充积压
4. 不调用关卡；不写入文件

**断言：**
- [ ] 判定为 BLOCKED
- [ ] 输出包含 "No unstarted stories" 或等效消息
- [ ] 输出推荐 `/create-stories`
- [ ] PR-SPRINT 关卡不调用
- [ ] 不调用写入工具

---

### 用例 3：关卡返回 CONCERNS——Sprint 超载，写入前修订

**Fixture：**
- 积压有 8 个 story 共 16 个点；里程碑容量为 10 个点
- `review-mode.txt` 包含 `full`

**输入：** `/sprint-plan`

**预期行为：**
1. Skill 起草包含所有 8 个 story 的 sprint（超出容量）
2. PR-SPRINT 关卡运行；producer 返回 CONCERNS：sprint 超载
3. Skill 向用户展示关切并询问要延期哪些 story
4. 用户选择 3 个 story 延期；sprint 修订为 5 个 story / 10 个点
5. Skill 询问"可以写入吗"附带修订后的 sprint；批准后写入

**断言：**
- [ ] 来自 PR-SPRINT 关卡的 CONCERNS 在任何写入前展示给用户
- [ ] Skill 允许在关卡反馈后修订 sprint
- [ ] 写入的是修订后的 sprint（不是原始版本）
- [ ] 修订和写入后判定为 COMPLETE

---

### 用例 4：Lean 模式——PR-SPRINT 关卡被跳过

**Fixture：**
- 积压有 4 个 story；里程碑容量为 8 个点
- `review-mode.txt` 包含 `lean`

**输入：** `/sprint-plan`

**预期行为：**
1. Skill 读取审查模式——确定为 `lean`
2. Skill 起草 sprint 并向用户展示
3. PR-SPRINT 关卡被跳过；输出注明"[PR-SPRINT] skipped——Lean mode"
4. Skill 请求用户直接批准 sprint
5. 用户批准；sprint 文件写入

**断言：**
- [ ] PR-SPRINT 关卡在 lean 模式下不调用
- [ ] 跳过在输出中明确注明
- [ ] 写入前仍需要用户批准（gate 跳过 ≠ 批准跳过）
- [ ] 写入后判定为 COMPLETE

---

### 用例 5：边缘情况——先前的 sprint 仍有未关闭 story

**Fixture：**
- `production/sprints/sprint-002.md` 存在，有 2 个 story 仍为 `Status: In Progress`
- 积压有 5 个新的未开始 story
- `review-mode.txt` 包含 `full`

**输入：** `/sprint-plan`

**预期行为：**
1. Skill 读取 sprint-002 并检测到 2 个未关闭（进行中）的 story
2. Skill 标记："Sprint 002 has 2 open stories——confirm carry-over before planning sprint 003"
3. Skill 向用户展示选择：延期 story、延期它们、或取消
4. 用户确认延期；延期的 story 以 `[CARRY]` 标签前置到新 sprint
5. Sprint 草稿构建；PR-SPRINT 关卡运行；sprint 在批准后写入

**断言：**
- [ ] Skill 检查最近的 sprint 文件中的未关闭 story
- [ ] 在继续 sprint 计划前要求用户确认延期
- [ ] 延期的 story 在新 sprint 草稿中以区分标签出现
- [ ] Skill 不忽略先前 sprint 中的未关闭 story

---

## 协议合规性

- [ ] 在调用 PR-SPRINT 关卡或请求写入前展示 sprint 草稿
- [ ] 写入 sprint 文件前始终询问"可以写入吗"
- [ ] PR-SPRINT 关卡仅在 full 模式下运行
- [ ] 跳过消息出现在 lean 和 solo 模式输出中
- [ ] 判定在 skill 输出末尾明确说明

---

## 覆盖说明

- 无里程碑文件存在的情况未明确测试；行为
  遵循 BLOCKED 模式，建议运行 `/gate-check` 进行
  里程碑推进。
- Solo 模式行为等同于 lean（gate 跳过，用户批准
  需要）且未单独测试。
- 并行 story 选择算法不在此处测试；这些是
  sprint-plan subagent 的单元问题。
