# Skill 测试规格：/sprint-status

## Skill 摘要

`/sprint-status` 是 Haiku 层级的只读 skill，它读取当前活动 sprint 文件和
会话状态以生成简明 sprint 健康摘要。它按状态报告 story 数量
（Complete / In Progress / Blocked / Not Started）并发出三个 sprint 健康判定之一：
ON TRACK、AT RISK 或 BLOCKED。它从不写入文件，也不调用任何 director 关卡。
它专为会话期间的快速、低成本状态检查而设计。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题或编号检查部分
- [ ] 包含判定关键词：ON TRACK、AT RISK、BLOCKED
- [ ] 不需要"可以写入吗"语言（只读 skill）
- [ ] 有下一步交接（根据判定结果做什么）

---

## Director 关卡检查

无。`/sprint-status` 是只读报告 skill；不调用关卡。

---

## 测试用例

### 用例 1：正常路径——混合 sprint，AT RISK 并命名阻塞项

**Fixture：**
- `production/sprints/sprint-004.md` 存在（活动 sprint，在 `active.md` 中链接）
- Sprint 包含 6 个 story：
  - 3 个 `Status: Complete`
  - 2 个 `Status: In Progress`
  - 1 个 `Status: Blocked`（阻塞项："Waiting on physics ADR acceptance"）
- Sprint 结束日期还有 2 天

**输入：** `/sprint-status`

**预期行为：**
1. Skill 读取 `production/session-state/active.md` 以找到活动 sprint 引用
2. Skill 读取 `production/sprints/sprint-004.md`
3. Skill 按状态统计 story：3 Complete、2 In Progress、1 Blocked
4. Skill 检测到 Blocked story 和临近的截止日期
5. Skill 输出 AT RISK 判定，明确命名阻塞项

**断言：**
- [ ] 输出包含按状态分类的 story 数量明细
- [ ] 输出命名具体的阻塞 story 及其阻塞原因
- [ ] 当有任何 story 为 Blocked 时判定为 AT RISK（不是 BLOCKED，不是 ON TRACK）
- [ ] Skill 不写入任何文件

---

### 用例 2：所有 Story 完成——Sprint COMPLETE 判定

**Fixture：**
- `production/sprints/sprint-004.md` 存在
- 所有 5 个 story 的 `Status: Complete`

**输入：** `/sprint-status`

**预期行为：**
1. Skill 读取 sprint 文件——所有 story 均为 Complete
2. Skill 输出 ON TRACK 判定或 SPRINT COMPLETE 标签
3. Skill 建议运行 `/milestone-review` 或 `/sprint-plan` 作为下一步

**断言：**
- [ ] 当所有 story 均为 Complete 时判定为 ON TRACK 或 SPRINT COMPLETE
- [ ] 输出注明 sprint 已全部完成
- [ ] 下一步建议引用 `/milestone-review` 或 `/sprint-plan`
- [ ] 未写入任何文件

---

### 用例 3：无活动 Sprint 文件——引导运行 /sprint-plan

**Fixture：**
- `production/session-state/active.md` 未引用活动 sprint
- `production/sprints/` 目录为空或不存在

**输入：** `/sprint-status`

**预期行为：**
1. Skill 读取 `active.md`——未找到活动 sprint 引用
2. Skill 检查 `production/sprints/`——未找到文件
3. Skill 输出信息性消息：未检测到活动 sprint
4. Skill 建议运行 `/sprint-plan` 创建一个

**断言：**
- [ ] 当无 sprint 文件存在时 skill 不会报错或崩溃
- [ ] 输出清楚说明未找到活动 sprint
- [ ] 输出推荐 `/sprint-plan` 作为下一步操作
- [ ] 不发出判定关键词（无 sprint 可评估）

---

### 用例 4：边缘情况——过期的 In Progress Story（被标记）

**Fixture：**
- `production/sprints/sprint-004.md` 存在
- 一个 story 的 `Status: In Progress`，在 `active.md` 中有注释：
  `Last updated: 2026-03-30`（距今天会话日期超过 2 天）
- 无 story 为 Blocked

**输入：** `/sprint-status`

**预期行为：**
1. Skill 读取 sprint 文件和会话状态
2. Skill 检测到 story 已 In Progress >2 天未更新
3. Skill 在输出中将 story 标记为"stale"
4. 判定为 AT RISK（过期的进行中 story 表明存在隐藏阻塞项）

**断言：**
- [ ] Skill 将 story 的"last updated"元数据与会话日期进行比较
- [ ] 过期的 In Progress story 在输出中按名称标记
- [ ] 当检测到过期 story 时判定为 AT RISK，不是 ON TRACK
- [ ] 输出不将"stale"与"Blocked"混淆——标签是区分的

---

### 用例 5：关卡合规性——只读；不调用关卡

**Fixture：**
- `production/sprints/sprint-004.md` 存在，有 4 个 story（2 Complete、2 In Progress）
- `production/session-state/review-mode.txt` 包含 `full`

**输入：** `/sprint-status`

**预期行为：**
1. Skill 读取 sprint 并生成状态摘要
2. Skill 无论审查模式如何都不调用任何 director 关卡
3. 输出是带有 ON TRACK、AT RISK 或 BLOCKED 判定的纯状态报告
4. Skill 不提示用户批准或请求写入任何文件

**断言：**
- [ ] 在任何审查模式下都不调用 director 关卡
- [ ] 输出不包含任何"可以写入吗"提示
- [ ] Skill 无需用户交互即完成并返回判定
- [ ] 审查模式文件被此 skill 忽略（或确认无关）

---

## 协议合规性

- [ ] 不使用 Write 或 Edit 工具（只读 skill）
- [ ] 发出判定前展示 story 数量明细
- [ ] 不请求批准
- [ ] 以基于判定的推荐下一步结束
- [ ] 在 Haiku 模型层级运行（快速、低成本）

---

## 覆盖说明

- 多个 sprint 同时活动的情况未测试；
  skill 读取 `active.md` 引用的任何 sprint。
- 部分 sprint 完成百分比未明确验证；
  按状态计数输出隐含了它们。
- `solo` 模式审查模式变体未单独测试；
  用例 5 中的关卡行为平等适用于所有模式。
