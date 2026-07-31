# Skill 测试规格：/milestone-review

## Skill 摘要

`/milestone-review` 生成已完成里程碑的全面审查：
已交付的内容、速度指标、延期项目、发现的风险和 retrospective 种子。
在 full 模式下，PR-MILESTONE director 关卡在审查编译后运行
（producer 审查范围交付）。在 lean 和 solo 模式下，关卡被跳过。
该 skill 在持久化前询问"可以写入 `production/milestones/review-milestone-N.md` 吗？"。
判定结果：MILESTONE COMPLETE 或 MILESTONE INCOMPLETE。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：MILESTONE COMPLETE、MILESTONE INCOMPLETE
- [ ] 包含"可以写入吗"语言（skill 写入审查文档）
- [ ] 有下一步交接（审查写入后做什么）

---

## Director 关卡检查

| Gate ID       | 触发条件                     | 模式守卫                  |
|---------------|------------------------------|---------------------------|
| PR-MILESTONE  | 审查文档编译后               | full only（非 lean/solo）  |

---

## 测试用例

### 用例 1：正常路径——几乎完成的里程碑，有一个延期 story

**Fixture：**
- `production/milestones/milestone-03.md` 存在，有 8 个 story
- 7 个 story 的 `Status: Complete`
- 1 个 story 的 `Status: Deferred`（延期到 milestone-04）
- `review-mode.txt` 包含 `full`

**输入：** `/milestone-review milestone-03`

**预期行为：**
1. Skill 读取 `milestone-03.md` 和所有引用的 sprint 文件
2. Skill 编译：7 个已交付，1 个延期；速度；无阻塞
3. Skill 向用户展示审查草稿
4. 调用 PR-MILESTONE 关卡；producer 批准
5. Skill 询问"可以写入 `production/milestones/review-milestone-03.md` 吗？"
6. 用户批准；文件写入；判定 MILESTONE COMPLETE

**断言：**
- [ ] 延期 story 在审查中注明其目标里程碑
- [ ] 尽管有一个延期 story，判定仍为 MILESTONE COMPLETE
- [ ] PR-MILESTONE 关卡在 full 模式下草稿编译后调用
- [ ] Skill 在写入审查文件前询问"可以写入吗"
- [ ] 审查文档路径匹配 `production/milestones/review-milestone-03.md`

---

### 用例 2：阻塞的里程碑——多个阻塞 story

**Fixture：**
- `production/milestones/milestone-03.md` 存在，有 5 个 story
- 2 个 story 的 `Status: Complete`
- 3 个 story 的 `Status: Blocked`（每个 story 中列出命名阻塞项）
- `review-mode.txt` 包含 `full`

**输入：** `/milestone-review milestone-03`

**预期行为：**
1. Skill 读取里程碑和 sprint 文件
2. Skill 发现 3 个阻塞 story；编译阻塞项详情
3. 判定为 MILESTONE INCOMPLETE
4. PR-MILESTONE 关卡运行；producer 注意未解决的阻塞项
5. 审查在批准后写入，附带阻塞项列表

**断言：**
- [ ] 当有任何 story 为 Blocked 时判定为 MILESTONE INCOMPLETE
- [ ] 每个阻塞 story 的名称和阻塞原因在审查中列出
- [ ] 即使判定为 INCOMPLETE，PR-MILESTONE 关卡在 full 模式下仍被调用
- [ ] 文件写入前仍出现"可以写入吗"提示

---

### 用例 3：Full 模式——PR-MILESTONE 返回 CONCERNS

**Fixture：**
- Milestone-03 有 6 个完成 story，但 2 个不在原始范围中（sprint 中期添加）
- `review-mode.txt` 包含 `full`

**输入：** `/milestone-review milestone-03`

**预期行为：**
1. Skill 编译审查；注意 2 个范围外 story 已交付
2. 调用 PR-MILESTONE 关卡；producer 对范围蔓延返回 CONCERNS
3. Skill 向用户展示 CONCERNS 并在审查中添加"scope drift"注释
4. 用户批准修订后的审查；文件写入为 MILESTONE COMPLETE 附带注意事项

**断言：**
- [ ] 来自 PR-MILESTONE 关卡的 CONCERNS 在写入前展示给用户
- [ ] 范围蔓延在写入的审查文档中明确注明
- [ ] 判定为 MILESTONE COMPLETE（story 已交付）附带 CONCERNS 注释
- [ ] Skill 不抑制关卡反馈

---

### 用例 4：边缘情况——未找到指定里程碑的里程碑文件

**Fixture：**
- 用户调用 `/milestone-review milestone-07`
- `production/milestones/milestone-07.md` 不存在

**输入：** `/milestone-review milestone-07`

**预期行为：**
1. Skill 尝试读取 `production/milestones/milestone-07.md`
2. 文件未找到；skill 输出错误消息
3. Skill 建议检查 `production/milestones/` 中的可用里程碑
4. 不调用关卡；不写入文件

**断言：**
- [ ] 当里程碑文件缺失时 skill 不会崩溃
- [ ] 输出在错误消息中命名预期文件路径
- [ ] 输出建议检查 `production/milestones/` 获取有效的里程碑名称
- [ ] 判定为 BLOCKED（无法审查不存在的里程碑）

---

### 用例 5：Lean/Solo 模式——PR-MILESTONE 关卡被跳过

**Fixture：**
- `production/milestones/milestone-03.md` 存在，有 5 个完成 story
- `review-mode.txt` 包含 `solo`

**输入：** `/milestone-review milestone-03`

**预期行为：**
1. Skill 读取审查模式——确定为 `solo`
2. Skill 编译审查草稿
3. PR-MILESTONE 关卡被跳过；输出注明"[PR-MILESTONE] skipped——Solo mode"
4. Skill 请求用户直接批准审查
5. 用户批准；审查文件写入；判定 MILESTONE COMPLETE

**断言：**
- [ ] PR-MILESTONE 关卡在 solo（或 lean）模式下不调用
- [ ] 跳过在 skill 输出中明确注明
- [ ] 写入前仍需要用户直接批准
- [ ] 成功写入后判定为 MILESTONE COMPLETE

---

## 协议合规性

- [ ] 在调用 PR-MILESTONE 或请求写入前展示编译的审查草稿
- [ ] 写入审查文档前始终询问"可以写入吗"
- [ ] PR-MILESTONE 关卡仅在 full 模式下运行
- [ ] 跳过消息出现在 lean 和 solo 输出中
- [ ] 判定为 MILESTONE COMPLETE 或 MILESTONE INCOMPLETE，清楚说明

---

## 覆盖说明

- 里程碑有零个 story 的情况未测试；它遵循
  MILESTONE INCOMPLETE 模式，附带注释建议里程碑可能
  未被规划。
- 速度计算细节（story points vs. story count）未在此处验证；
  它们是审查编译阶段的实现细节。
