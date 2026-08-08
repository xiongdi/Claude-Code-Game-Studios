# 技能测试规格: /team-qa

## 技能摘要

协调 QA 团队完成一个 7 阶段结构化测试周期。协调 qa-lead（策略、测试计划、签字报告）和 qa-tester（测试用例编写、bug 报告编写）。涵盖范围检测、Story 分类、QA 计划生成、smoke check 门控、测试用例编写、带 bug 归档的手动 QA 执行，以及最终签字报告（包含 APPROVED / APPROVED WITH CONDITIONS / NOT APPROVED 裁决）。在第 5 阶段对独立的 Story 使用并行 qa-tester 派生。

---

## 静态断言（结构性）

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含签字报告的判定关键词：APPROVED、APPROVED WITH CONDITIONS、NOT APPROVED
- [ ] 对 QA 计划和签字报告都包含"可以写入吗"语言
- [ ] 有 Error Recovery Protocol 章节
- [ ] 在阶段转换时使用 `AskUserQuestion` 以在继续前获取用户批准
- [ ] 第 4 阶段（smoke check）是硬门控：FAIL 会停止周期
- [ ] Bug 报告写入 `production/qa/bugs/`，命名格式为 `BUG-[NNN]-[short-slug].md`
- [ ] 下一步指引因判定结果而异（APPROVED / APPROVED WITH CONDITIONS / NOT APPROVED）
- [ ] 第 5 阶段中独立的 qa-tester 任务并行派生

---

## 测试用例

### 用例 1：正常路径——所有 story 通过手动 QA，APPROVED 判定

**Fixture：**
- `production/sprints/sprint-03/` 存在，有 4 个 story 文件
- Story 类型混合：1 个 Logic、1 个 Integration、2 个 Visual/Feel
- 所有 story 的验收标准都已填写
- `tests/smoke/` 包含 smoke test 列表；所有项目可验证
- `production/qa/bugs/` 中无现有 bug

**输入：** `/team-qa sprint-03`

**预期行为：**
1. 第 1 阶段：读取 `production/sprints/sprint-03/` 中的所有 story 文件；读取 `production/stage.txt`；报告"发现 4 个 story。当前阶段：[stage]。准备开始 QA 策略？"
2. 第 2 阶段：通过 Task 派生 `qa-lead`；生成策略表对所有 4 个 story 分类；无阻塞项标记；展示给用户；AskUserQuestion：用户选择"看起来不错——继续测试计划"
3. 第 3 阶段：生成 QA 计划文档；询问"可以写入 QA 计划到 `production/qa/qa-plan-sprint-03-[date].md` 吗？"；批准后写入
4. 第 4 阶段：通过 Task 派生 `qa-lead`；审查 `tests/smoke/`；返回 PASS；报告"Smoke check 通过。继续测试用例编写。"
5. 第 5 阶段：为每个 Visual/Feel 和 Integration story（2-3 个）通过 Task 派生 `qa-tester`；并行运行；按 story 分组展示测试用例；每组 AskUserQuestion；用户批准
6. 第 6 阶段：遍历每个已批准的 story；用户将所有标记为 PASS；结果摘要："Stories PASS: 4, FAIL: 0, BLOCKED: 0"
7. 第 7 阶段：通过 Task 派生 `qa-lead` 生成签字报告；报告显示所有 story PASS；无 bug 归档；判定：APPROVED；询问"可以写入此 QA 签字报告到 `production/qa/qa-signoff-sprint-03-[date].md` 吗？"；批准后写入
8. 判定：COMPLETE——QA 周期完成

**断言：**
- [ ] 第 1 阶段正确计数并报告 4 个 story 及当前阶段
- [ ] 第 2 阶段的策略表对所有 4 个 story 按正确类型分类
- [ ] QA 计划仅在"可以写入吗"批准后写入
- [ ] Smoke check PASS 允许管线继续，无需用户干预
- [ ] 第 5 阶段独立 story 的 qa-tester 任务并行发出
- [ ] 签字报告包含测试覆盖率摘要表和判定：APPROVED
- [ ] 签字报告仅在"可以写入吗"批准后写入
- [ ] 最终输出中出现判定：COMPLETE
- [ ] 下一步："运行 `/gate-check` 验证推进。"

---

### 用例 2：Smoke Check 失败——QA 周期在第 4 阶段停止

**Fixture：**
- `production/sprints/sprint-04/` 存在，有 3 个 story 文件
- `tests/smoke/` 存在，有 5 个 smoke test 项目；2 个项目无法验证（例如，build 不稳定、核心导航损坏）

**输入：** `/team-qa sprint-04`

**预期行为：**
1. 第 1-3 阶段正常完成；QA 计划已写入
2. 第 4 阶段：通过 Task 派生 `qa-lead`；smoke check 返回 FAIL；识别出两个具体失败项
3. Skill 报告："Smoke check 失败。QA 在这些问题解决前无法开始：[2 个失败项列表]。修复后重新运行 `/smoke-check`，或解决后重新运行 `/team-qa`。"
4. Skill 在第 4 阶段后立即停止——不执行第 5、6、7 阶段
5. 不生成签字报告；不发出"可以写入吗"请求

**断言：**
- [ ] Smoke check FAIL 导致管线在第 4 阶段停止——第 5、6、7 阶段不执行
- [ ] 失败列表明确展示给用户（不是模糊总结）
- [ ] Skill 推荐 `/smoke-check` 和 `/team-qa` 重跑作为修复步骤
- [ ] 不写入或不提供 QA 签字报告
- [ ] Skill 不产生 COMPLETE 判定
- [ ] 第 3 阶段已写入的任何 QA 计划保留（不删除）

---

### 用例 3：发现 Bug——Visual/Feel story 手动 QA 失败，归档 bug 报告

**Fixture：**
- `production/sprints/sprint-05/` 存在，有 2 个 story 文件：1 个 Logic（通过自动化测试）、1 个 Visual/Feel
- `tests/smoke/` smoke check 通过
- Visual/Feel story 的动画时序明显错误（验收标准未满足）
- `production/qa/bugs/` 目录存在（为空或有现有 bug）

**输入：** `/team-qa sprint-05`

**预期行为：**
1. 第 1-5 阶段正常完成；为 Visual/Feel story 编写了测试用例
2. 第 6 阶段：用户将 Visual/Feel story 标记为 FAIL；AskUserQuestion 收集失败描述："Animation plays at 2x speed — jitter visible on every loop"
3. 第 6 阶段：通过 Task 派生 `qa-tester` 编写正式 bug 报告；bug 报告写入 `production/qa/bugs/BUG-001-animation-speed-jitter.md`（或目录中已有 bug 时递增）；报告包含严重性字段
4. 结果摘要："Stories PASS: 1, FAIL: 1 — bugs filed: BUG-001"
5. 第 7 阶段：通过 Task 派生 `qa-lead` 生成签字报告；Bugs Found 表列出 BUG-001 及严重性和状态 Open；判定：NOT APPROVED（S1/S2 bug 未解决，或 FAIL 无记录变通方案）
6. 提供签字报告写入；批准后写入
7. 下一步："解决 S1/S2 bug 并在推进前重新运行 `/team-qa` 或针对性手动 QA。"

**断言：**
- [ ] 第 6 阶段的 FAIL 结果触发 AskUserQuestion 在 bug 报告写入前收集失败描述
- [ ] `qa-tester` 通过 Task 派生编写 bug 报告——编排器不直接编写
- [ ] Bug 报告遵循命名规范：`BUG-[NNN]-[short-slug].md`，位于 `production/qa/bugs/`
- [ ] Bug 报告 NNN 从目录中现有 bug 正确递增
- [ ] 第 7 阶段签字报告 Bugs Found 表包含 bug ID、story 名称、严重性和状态
- [ ] 签字报告中的判定为 NOT APPROVED
- [ ] 下一步明确提及重新运行 `/team-qa`
- [ ] 编排器仍发出判定：COMPLETE（QA 周期已完成——判定是 NOT APPROVED，但 skill 完成了其管线）

---

### 用例 4：无参数——Skill 推断活动 sprint 或询问用户

**Fixture（变体 A —— 状态文件存在）：**
- `production/session-state/active.md` 存在且包含对 `sprint-06` 的引用
- `production/sprint-status.yaml` 存在且标识 `sprint-06` 为活动状态

**Fixture（变体 B —— 状态文件缺失）：**
- `production/session-state/active.md` 不存在
- `production/sprint-status.yaml` 不存在

**输入：** `/team-qa`（无参数）

**预期行为（变体 A）：**
1. 第 1 阶段：未提供参数；读取 `production/session-state/active.md`；读取 `production/sprint-status.yaml`
2. 从两个来源检测到 `sprint-06` 为活动 sprint
3. 如同输入是 `/team-qa sprint-06` 一样继续；报告"未提供 sprint 参数——从会话状态推断为 sprint-06。发现 [N] 个 story。"

**预期行为（变体 B）：**
1. 第 1 阶段：未提供参数；尝试读取 `production/session-state/active.md`——文件缺失；尝试读取 `production/sprint-status.yaml`——文件缺失
2. 无法推断 sprint；使用 AskUserQuestion："QA 应覆盖哪个 sprint 或功能？"，提供输入 sprint 标识符或取消的选项

**断言：**
- [ ] Skill 在未提供参数时不默认使用硬编码的 sprint 名称
- [ ] Skill 在询问用户前读取 `production/session-state/active.md` 和 `production/sprint-status.yaml`（变体 A）
- [ ] 当两个状态文件都缺失时，skill 使用 AskUserQuestion 而不是猜测（变体 B）
- [ ] 推断的 sprint 在继续前报告给用户（变体 A 透明度）
- [ ] Skill 在状态文件缺失时不报错——回退到询问（变体 B）

---

### 用例 5：混合结果——部分 PASS，一个 FAIL 带 S1 bug，一个 BLOCKED

**Fixture：**
- `production/sprints/sprint-07/` 存在，有 4 个 story 文件
- Smoke check 通过
- Story A（Logic）：自动化测试通过——PASS
- Story B（UI）：手动 QA——PASS WITH NOTES（轻微文本溢出）
- Story C（Visual/Feel）：手动 QA——FAIL；测试者识别出技能激活时 S1 崩溃
- Story D（Integration）：无法测试——BLOCKED（依赖系统尚未实现）

**输入：** `/team-qa sprint-07`

**预期行为：**
1. 第 1-5 阶段进行；第 5 阶段测试用例覆盖 story B、C、D
2. 第 6 阶段：用户将 Story A 标记为隐式 PASS（自动化）；Story B：PASS WITH NOTES；Story C：FAIL；Story D：BLOCKED
3. Story C FAIL 后：派生 qa-tester 编写 bug 报告 `BUG-001-crash-ability-activation.md`，严重性为 S1
4. 展示结果摘要："Stories PASS: 1, PASS WITH NOTES: 1, FAIL: 1 — bugs filed: BUG-001 (S1), BLOCKED: 1"
5. 第 7 阶段：qa-lead 生成签字报告覆盖所有 4 个 story；BUG-001 列为 S1/Open；Story D 列为 BLOCKED；判定：NOT APPROVED
6. 签字报告在"可以写入吗"批准后写入
7. 下一步："解决 S1/S2 bug 并在推进前重新运行 `/team-qa` 或针对性手动 QA。"

**断言：**
- [ ] 所有 4 个 story 出现在第 7 阶段签字报告测试覆盖率摘要表中——无静默省略
- [ ] Story D（BLOCKED）在报告中列为 BLOCKED 状态，不静默丢弃
- [ ] S1 bug 导致判定：NOT APPROVED，无论其他 story 是否通过
- [ ] PASS WITH NOTES story 不降级为 FAIL——它们被单独追踪
- [ ] BUG-001 严重性在 Bugs Found 表中列为 S1
- [ ] 部分结果被保留——即使有失败和阻塞，仍生成签字报告
- [ ] 编排器发出判定：COMPLETE（管线已完成）；签字判定为 NOT APPROVED

---

## 协议合规性

- [ ] `AskUserQuestion` 用于第 2 阶段（策略审查）、第 5 阶段（每组测试用例批准）和第 6 阶段（每个 story 手动 QA 结果）
- [ ] 第 4 阶段 smoke check 是硬门控：FAIL 在第 4 阶段停止管线，无例外
- [ ] "可以写入吗"分别针对 QA 计划（第 3 阶段）和签字报告（第 7 阶段）询问
- [ ] Bug 报告始终由 `qa-tester` 通过 Task 编写——编排器不直接编写
- [ ] 第 5 阶段独立 story 的 qa-tester 任务尽可能并行发出
- [ ] 错误恢复：任何 BLOCKED agent 立即浮现，附带 AskUserQuestion 选项
- [ ] 始终生成部分报告——没有工作因一个 story 失败或阻塞而被丢弃
- [ ] 签字判定规则严格执行：任何 S1/S2 bug 未解决 = NOT APPROVED；无例外
- [ ] 编排器级判定：COMPLETE 与签字报告的 APPROVED/NOT APPROVED 判定是不同的

---

## 覆盖说明

- "APPROVED WITH CONDITIONS" 判定路径（S3/S4 bug、PASS WITH NOTES）由用例 5 的 PASS WITH NOTES story（Story B）隐式覆盖——如果不存在 S1/S2 bug，该用例将产生 APPROVED WITH CONDITIONS。由于判定逻辑是表驱动的，不需要专门的用例。
- `feature: [system-name]` 参数形式未单独测试——它遵循与 sprint 形式相同的第 1 阶段逻辑，使用 glob 而非目录读取。无参数推断路径（用例 4）为检测逻辑提供了足够的覆盖。
- 通过自动化测试的 Logic story 不需要手动 QA——这由用例 5（Story A）隐式验证，其中 Logic story 不接收手动 QA 阶段。
- 第 5 阶段并行 qa-tester 派生由用例 1（多个 Visual/Feel story 同时发出）隐式验证；除了静态断言检查外，不需要专门的并行性用例。
