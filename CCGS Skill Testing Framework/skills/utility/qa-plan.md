# Skill 测试规格：/qa-plan

## Skill 摘要

`/qa-plan` 为功能或 sprint 里程碑生成结构化的 QA 测试计划。它读取指定 sprint 的 story 文件，从每个 story 中提取验收标准，交叉引用 `coding-standards.md` 中的测试标准以分配适当的测试类型（unit、integration、visual、UI 或 config/data），并生成一个优先排序的 QA 计划文档。

该 skill 在持久化输出前会询问 "May I write to `production/qa/qa-plan-sprint-NNN.md`?"。如果发现同一 sprint 的现有测试计划，该 skill 会提供更新而不是替换。当计划写入后裁决为 COMPLETE。不使用 director gate——gate 级别的 story 准备状态由 `/story-readiness` 处理。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含裁决关键词：COMPLETE
- [ ] 在写入计划前包含 "May I write" 协作协议语言
- [ ] 有下一步交接（例如，`/smoke-check` 或 `/story-readiness`）

---

## Director 关卡检查

无。`/qa-plan` 是一个计划工具。Story 准备状态的 gate 是独立的。

---

## 测试用例

### 用例 1：正常路径——包含 4 个 story 的 sprint 生成完整测试计划

**Fixture：**
- `production/sprints/sprint-003.md` 列出 4 个具有已定义验收标准的 story
- Story 跨越类型：1 个 logic（公式）、1 个 integration、1 个 visual、1 个 UI
- `coding-standards.md` 存在并包含测试证据表

**输入：** `/qa-plan sprint-003`

**预期行为：**
1. Skill 读取 sprint-003.md 并识别 4 个 story
2. Skill 读取每个 story 的验收标准
3. Skill 按照 coding-standards.md 表分配测试类型：
   - Logic story → Unit test (BLOCKING)
   - Integration story → Integration test (BLOCKING)
   - Visual story → Screenshot + lead sign-off (ADVISORY)
   - UI story → Manual walkthrough doc (ADVISORY)
4. Skill 起草包含逐 story 测试类型分解的 QA 计划
5. Skill 询问 "May I write to `production/qa/qa-plan-sprint-003.md`?"
6. 批准后写入文件；裁决为 COMPLETE

**断言：**
- [ ] 所有 4 个 story 都包含在计划中
- [ ] 测试类型按照 coding-standards.md 分配（不是猜测）
- [ ] 每个 story 都注明了 gate 级别（BLOCKING vs ADVISORY）
- [ ] "May I write" 使用正确的文件路径
- [ ] 裁决为 COMPLETE

---

### 用例 2：无验收标准的 Story——标记为 UNTESTABLE

**Fixture：**
- `production/sprints/sprint-004.md` 列出 3 个 story；一个 story 的验收标准部分为空

**输入：** `/qa-plan sprint-004`

**预期行为：**
1. Skill 读取所有 3 个 story
2. Skill 检测到没有 AC 的 story
3. Story 在计划中被标记为 `UNTESTABLE — Acceptance Criteria required`
4. 其他 2 个 story 获得正常的测试类型分配
5. 计划写入时标记了 UNTESTABLE story；裁决为 COMPLETE

**断言：**
- [ ] 没有 AC 的 story 出现 UNTESTABLE 标签
- [ ] 计划未被阻塞——其他 story 仍被计划
- [ ] 输出建议为标记的 story 添加 AC（下一步）
- [ ] 裁决为 COMPLETE（计划仍被生成）

---

### 用例 3：发现现有测试计划——提供更新而不是替换

**Fixture：**
- `production/qa/qa-plan-sprint-003.md` 已存在（来自上次运行）
- Sprint-003 自上次计划以来新增了 2 个 story

**输入：** `/qa-plan sprint-003`

**预期行为：**
1. Skill 读取 sprint-003.md 并检测到 2 个不在现有计划中的 story
2. Skill 报告："Existing QA plan found for sprint-003 — offering to update"
3. Skill 展示 2 个新 story 及其建议的测试分配
4. Skill 询问 "May I update `production/qa/qa-plan-sprint-003.md`?"（不是覆盖）
5. 批准后写入更新后的计划

**断言：**
- [ ] Skill 检测到现有计划文件
- [ ] 使用 "update" 语言（不是 "overwrite"）
- [ ] 仅提议添加新 story——保留现有条目
- [ ] 裁决为 COMPLETE

---

### 用例 4：未找到 Sprint 的 Story——附带指导的错误

**Fixture：**
- `production/sprints/sprint-007.md` 不存在
- 没有其他匹配 sprint-007 的 sprint 文件

**输入：** `/qa-plan sprint-007`

**预期行为：**
1. Skill 尝试读取 sprint-007.md——文件未找到
2. Skill 输出："No sprint file found for sprint-007"
3. Skill 建议运行 `/sprint-plan` 先创建 sprint
4. 不写入计划；不询问 "May I write"

**断言：**
- [ ] 错误消息命名缺失的 sprint 文件
- [ ] 建议 `/sprint-plan` 作为修复步骤
- [ ] 不调用写入工具
- [ ] 裁决不是 COMPLETE（错误状态）

---

### 用例 5：Director 关卡检查——无 gate；QA 计划是实用工具

**Fixture：**
- 具有有效 story 和 AC 的 sprint

**输入：** `/qa-plan sprint-003`

**预期行为：**
1. Skill 生成并写入 QA 计划
2. 不派生 director agent
3. 输出中不出现 gate ID

**断言：**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] Skill 无需任何 gate 检查即可达到 COMPLETE

---

## 协议合规性

- [ ] 在分配测试类型前读取 coding-standards.md 测试证据表
- [ ] 按 story 类型分配 BLOCKING 或 ADVISORY gate 级别
- [ ] 将没有 AC 的 story 标记为 UNTESTABLE（不静默跳过）
- [ ] 检测现有计划并提供更新路径
- [ ] 在创建或更新计划文件前询问 "May I write"
- [ ] 计划写入后裁决为 COMPLETE

---

## 覆盖说明

- `coding-standards.md` 缺失（skill 无法分配测试类型）的情况未进行 fixture 测试；行为将遵循 BLOCKED 模式并附带恢复标准文件的说明。
- 多 sprint 计划（跨越 2 个 sprint）未测试；该 skill 设计为一次处理一个 sprint。
- Config/data story 类型（平衡调优 → smoke check）遵循与用例 1 中其他类型相同的分配模式，未单独测试。
