# Skill 测试规格：/release-checklist

## Skill 摘要

`/release-checklist` 生成一份内部发布就绪检查清单，涵盖：
sprint story 完成度、未关闭 bug 的严重级别、QA 签署状态、构建稳定性
以及 changelog 就绪情况。这是一个内部关卡——不是平台/商店清单
（那是 `/launch-checklist`）。当存在先前的发布检查清单时，它会展示
已解决和新引入问题的增量对比。

该 skill 在"可以写入吗"询问后，将检查清单报告写入 `production/releases/release-checklist-[date].md`。
不适用 director 关卡——`/gate-check` 处理正式的阶段关卡逻辑。判定结果：RELEASE READY、RELEASE BLOCKED 或 CONCERNS。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：RELEASE READY、RELEASE BLOCKED、CONCERNS
- [ ] 在写入报告前包含"可以写入吗"协作协议语言
- [ ] 有下一步交接（例如，外部用 `/launch-checklist`，阶段用 `/gate-check`）

---

## Director 关卡检查

无。`/release-checklist` 是内部审计工具。正式的阶段推进
由 `/gate-check` 管理。

---

## 测试用例

### 用例 1：正常路径——所有 Sprint Story 完成，QA 通过，RELEASE READY

**Fixture：**
- `production/sprints/sprint-008.md`——所有 story 均为 `Status: Done`
- `production/bugs/` 中没有严重级别为 HIGH 或 CRITICAL 的未关闭 bug
- `production/qa/qa-plan-sprint-008.md` 有 QA 签署注释
- 存在此版本的 changelog 条目
- `production/stage.txt` 包含 `Polish`

**输入：** `/release-checklist`

**预期行为：**
1. Skill 读取 sprint-008：所有 story 均为 Done
2. Skill 读取 bug：没有 HIGH 或 CRITICAL 级别的未关闭 bug
3. Skill 确认 QA plan 有签署
4. Skill 确认 changelog 条目存在
5. 所有检查通过；skill 询问"可以写入
   `production/releases/release-checklist-2026-04-06.md` 吗？"
6. 报告已写入；判定为 RELEASE READY

**断言：**
- [ ] 评估了全部 4 个检查类别（story、bug、QA、changelog）
- [ ] 所有项目都带有 PASS 标记
- [ ] 判定为 RELEASE READY
- [ ] 写入前询问了"可以写入吗"

---

### 用例 2：存在 HIGH 严重级别 Bug——RELEASE BLOCKED

**Fixture：**
- 所有 sprint story 均为 Done
- `production/bugs/` 包含 2 个严重级别为 HIGH 的未关闭 bug

**输入：** `/release-checklist`

**预期行为：**
1. Skill 读取 sprint——story 完成
2. Skill 读取 bug——2 个 HIGH 严重级别 bug 未关闭
3. Skill 报告："RELEASE BLOCKED——必须解决 2 个未关闭的 HIGH 严重级别 bug"
4. 两个 bug 文件名都列在报告中
5. 判定为 RELEASE BLOCKED

**断言：**
- [ ] 判定为 RELEASE BLOCKED（不是 CONCERNS）
- [ ] 两个 bug 文件名都被明确列出
- [ ] Skill 明确 HIGH 严重级别 bug 是阻塞性的（不是建议性的）

---

### 用例 3：Changelog 未生成——CONCERNS

**Fixture：**
- 所有 story 均为 Done，无 HIGH/CRITICAL bug
- 未找到当前版本/sprint 的 changelog 条目

**输入：** `/release-checklist`

**预期行为：**
1. Skill 检查所有项目
2. Changelog 检查失败：未找到 changelog 条目
3. Skill 报告："CONCERNS——未为此发布生成 Changelog"
4. Skill 建议运行 `/changelog` 来生成
5. 判定为 CONCERNS（建议性的——不是硬性阻塞）

**断言：**
- [ ] 判定为 CONCERNS（不是 RELEASE BLOCKED——changelog 是建议性的）
- [ ] 建议 `/changelog` 作为修复方案
- [ ] 报告中显示其他通过的检查
- [ ] 缺失的 changelog 被描述为建议性的，不是阻塞性的

---

### 用例 4：存在先前的发布检查清单——与上次发布的增量对比

**Fixture：**
- `production/releases/release-checklist-2026-03-20.md` 存在
- 先前：1 个 story 未完成，1 个 HIGH bug 未关闭
- 当前：所有 story 均为 Done，HIGH bug 已解决，但出现了 1 个 MEDIUM bug

**输入：** `/release-checklist`

**预期行为：**
1. Skill 找到先前的检查清单并加载
2. 生成新检查清单并比较：
   - 新解决："Story [X]——之前开放，现在 Done"
   - 新解决："HIGH bug [filename]——之前开放，现在已关闭"
   - 新项目："出现 1 个 MEDIUM bug（建议性）"
3. 增量部分突出显示所有变更
4. 判定为 CONCERNS（MEDIUM bug 是建议性的，不是阻塞性的）

**断言：**
- [ ] 报告中出现增量部分，包含已解决和新项目
- [ ] 注意到了先前检查清单中新解决的项目
- [ ] 突出显示了先前检查清单中没有的新项目
- [ ] 判定反映当前状态（不是先前状态）

---

### 用例 5：Director 关卡检查——无关卡；release-checklist 是内部审计

**Fixture：**
- 有 story 和 bug 报告的活动 sprint

**输入：** `/release-checklist`

**预期行为：**
1. Skill 运行完整检查清单并写入报告
2. 不派生任何 director agent
3. 输出中不出现 gate ID

**断言：**
- [ ] 未调用 director 关卡
- [ ] 不出现 gate 跳过消息
- [ ] 判定为 RELEASE READY、RELEASE BLOCKED 或 CONCERNS——无 gate 判定

---

## 协议合规性

- [ ] 检查 sprint story 完成状态
- [ ] 检查未关闭 bug 的严重级别（CRITICAL/HIGH = BLOCKED；MEDIUM/LOW = CONCERNS）
- [ ] 检查 QA plan 签署状态
- [ ] 检查 changelog 是否存在
- [ ] 存在先前检查清单时与之比较
- [ ] 写入报告前询问"可以写入吗"
- [ ] 判定为 RELEASE READY、RELEASE BLOCKED 或 CONCERNS

---

## 覆盖说明

- 构建稳定性验证（无 CI 运行失败）被列为检查类别，
  但依赖外部 CI 系统状态；如果未配置 CI 集成，skill 将其标注为 MANUAL CHECK。
- CRITICAL bug 始终导致 RELEASE BLOCKED，无论其他项目如何；
  这等同于用例 2 中的 HIGH 严重级别情况。
- `Status: In Review`（不是 Done）的 story 被视为未完成，
  并导致 RELEASE BLOCKED；此边缘情况遵循与 HIGH bug 情况相同的模式。
