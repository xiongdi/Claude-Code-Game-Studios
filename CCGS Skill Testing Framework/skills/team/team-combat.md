# 技能测试规格: /team-combat

## 技能摘要

协调完整的战斗团队端到端流程，完成单个战斗功能。协调 game-designer、gameplay-programmer、ai-programmer、technical-artist、sound-designer、主要引擎专家和 qa-tester 六个结构化阶段：Design → Architecture（含引擎专家验证）→ Implementation（并行）→ Integration → Validation → Sign-off。在每个阶段转换时使用 `AskUserQuestion`。将所有文件写入委托给 sub-agent。生成包含 COMPLETE / NEEDS WORK / BLOCKED 裁决的摘要报告，并交接给 `/code-review`、`/balance-check`、`/team-polish`。

---

## 静态断言（结构性）

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题（Phase 1 到 Phase 6 全部存在）
- [ ] 包含判定关键词：COMPLETE、NEEDS WORK、BLOCKED
- [ ] 包含"可以写入吗"或"File Write Protocol"语言——写入委托给 sub-agent，编排器不直接写入文件
- [ ] 末尾有下一步交接（引用 `/code-review`、`/balance-check`、`/team-polish`）
- [ ] Error Recovery Protocol 章节存在，包含全部四个恢复步骤
- [ ] 在阶段转换时使用 `AskUserQuestion` 以在继续前获取用户批准
- [ ] Phase 3 明确标记为并行（gameplay-programmer、ai-programmer、technical-artist、sound-designer）
- [ ] Phase 2 包括派生主要引擎专家（从 `.claude/docs/technical-preferences.md` 读取）
- [ ] Team Composition 列出全部七个角色（game-designer、gameplay-programmer、ai-programmer、technical-artist、sound-designer、引擎专家、qa-tester）

---

## 测试用例

### 用例 1：正常路径——所有 agent 成功，完整管线运行至完成

**Fixture：**
- `design/gdd/game-concept.md` 存在且已填充
- 引擎在 `.claude/docs/technical-preferences.md` 中配置（Engine Specialists 部分已填写）
- 请求的战斗功能无现有 GDD

**输入：** `/team-combat parry and riposte system`

**预期行为：**
1. Phase 1——派生 game-designer；产出 `design/gdd/parry-riposte.md`，涵盖全部 8 个必需章节（overview、player fantasy、rules、formulas、edge cases、dependencies、tuning knobs、acceptance criteria）；请求用户批准设计文档
2. Phase 2——派生 gameplay-programmer + ai-programmer；产出架构草图，包含类结构、接口和文件列表；然后派生主要引擎专家验证惯用法；引擎专家输出被纳入；`AskUserQuestion` 在 Phase 3 开始前展示架构选项
3. Phase 3——并行派生 gameplay-programmer、ai-programmer、technical-artist、sound-designer；四者在 Phase 4 开始前全部返回输出
4. Phase 4——集成将所有 Phase 3 输出连接在一起；验证 tuning knobs 是数据驱动的；`AskUserQuestion` 在 Phase 5 前确认集成
5. Phase 5——派生 qa-tester；从验收标准编写测试用例；验证边缘情况；对照预算检查性能影响
6. Phase 6——产出摘要报告：设计 COMPLETE、所有团队成员 COMPLETE、列出测试用例、判定：COMPLETE
7. 列出下一步：`/code-review`、`/balance-check`、`/team-polish`

**断言：**
- [ ] `AskUserQuestion` 在每个阶段门控调用（至少在 Phase 3 前和 Phase 5 前）
- [ ] Phase 3 agent 同时启动——gameplay-programmer、ai-programmer、technical-artist、sound-designer 之间无顺序依赖
- [ ] 引擎专家在 Phase 2 中 Phase 3 开始前运行（输出纳入架构）
- [ ] 所有文件写入委托给 sub-agent（编排器从不直接调用 Write/Edit）
- [ ] 最终报告中有判定 COMPLETE
- [ ] 下一步包括 `/code-review`、`/balance-check`、`/team-polish`
- [ ] 设计文档涵盖全部 8 个必需 GDD 章节

---

### 用例 2：阻塞的 Agent——一个 subagent 在管线中途返回 BLOCKED

**Fixture：**
- `design/gdd/parry-riposte.md` 存在（Phase 1 已完成）
- ai-programmer agent 返回 BLOCKED，因为不存在 AI 系统架构 ADR（ADR 状态为 Proposed）

**输入：** `/team-combat parry and riposte system`

**预期行为：**
1. Phase 1——找到设计文档；game-designer 确认其有效；阶段批准
2. Phase 2——gameplay-programmer 完成架构草图；ai-programmer 返回 BLOCKED："AI 行为系统的 ADR 是 Proposed——在 ADR 被 Accepted 前无法实现"
3. 触发 Error Recovery Protocol："ai-programmer: BLOCKED——AI 行为 ADR 是 Proposed"
4. 呈现 `AskUserQuestion`，选项：(a) 跳过 ai-programmer 并注明缺口；(b) 用更窄范围重试；(c) 在此停止并先运行 `/architecture-decision`
5. 如果用户选择 (a)：Phase 3 仅使用 gameplay-programmer、technical-artist、sound-designer 继续；ai-programmer 缺口在部分报告中注明
6. 生成最终报告：记录部分实现，ai-programmer 部分标记为 BLOCKED，整体判定：BLOCKED

**断言：**
- [ ] BLOCKED 浮现消息在任何依赖阶段继续前出现
- [ ] `AskUserQuestion` 至少提供三个选项：跳过/重试/停止
- [ ] 生成部分报告——已完成 agent 的工作不丢弃
- [ ] 整体判定为 BLOCKED（不是 COMPLETE），当任何 agent 未解决时
- [ ] 阻塞原因引用 ADR 并建议 `/architecture-decision`
- [ ] 编排器不静默继续通过阻塞的依赖

---

### 用例 3：无参数——显示清晰的使用指引

**Fixture：**
- 任何项目状态

**输入：** `/team-combat`（无参数）

**预期行为：**
1. Skill 检测到未提供参数
2. 输出使用消息解释所需参数（战斗功能描述）
3. 提供示例调用：`/team-combat [combat feature description]`
4. Skill 退出，不派生任何 subagent

**断言：**
- [ ] Skill 在未提供参数时不派生任何 subagent
- [ ] 使用消息包含 frontmatter 中的 argument-hint 格式
- [ ] 错误消息包含至少一个有效调用的示例
- [ ] 除检测缺失参数所需外不读取其他文件
- [ ] 不显示判定（管线从未运行）

---

### 用例 4：并行阶段验证——Phase 3 agent 同时运行

**Fixture：**
- `design/gdd/parry-riposte.md` 存在且完整
- 架构草图已批准
- 引擎专家已验证架构

**输入：** `/team-combat parry and riposte system`（从 Phase 2 完成恢复）

**预期行为：**
1. Phase 3 在架构批准后开始
2. 所有四个 Task 调用——gameplay-programmer、ai-programmer、technical-artist、sound-designer——在任何结果等待前发出
3. Skill 在进入 Phase 4 前等待所有四个 agent 完成
4. 如果任何单个 agent 提前完成，skill 在所有四个返回前不开始 Phase 4

**断言：**
- [ ] 四个 Task 调用在单批次中发出（它们之间无顺序等待）
- [ ] Phase 4 在所有四个 Phase 3 agent 返回结果前不开始
- [ ] Skill 不将一个 Phase 3 agent 的输出作为另一个 Phase 3 agent 的输入（它们是独立的）
- [ ] 所有四个 Phase 3 agent 的结果在 Phase 4 集成步骤中被引用

---

### 用例 5：架构阶段引擎路由——引擎专家接收正确上下文

**Fixture：**
- `.claude/docs/technical-preferences.md` 的 Engine Specialists 部分已填充（例如，Primary: godot-specialist）
- gameplay-programmer 产出的架构草图可用
- 引擎版本在 `docs/engine-reference/godot/VERSION.md` 中固定

**输入：** `/team-combat parry and riposte system`

**预期行为：**
1. Phase 2——gameplay-programmer 产出架构草图
2. Skill 读取 `.claude/docs/technical-preferences.md` 的 Engine Specialists 部分以识别主要引擎专家 agent 类型
3. 引擎专家被派生，附带：架构草图、GDD 路径、`VERSION.md` 中的引擎版本，以及检查废弃 API 的明确指示
4. 引擎专家输出（惯用法说明、废弃 API 警告、原生系统建议）返回给编排器
5. 编排器在将 Phase 2 结果展示给用户前将引擎说明纳入架构
6. `AskUserQuestion` 将引擎专家的说明与架构草图一起包含

**断言：**
- [ ] 引擎专家 agent 类型从 `.claude/docs/technical-preferences.md` 读取——不是硬编码
- [ ] 引擎专家提示包含架构草图和 GDD 路径
- [ ] 引擎专家对照固定的引擎版本检查废弃 API
- [ ] 引擎专家输出在 Phase 3 开始前被纳入（不跳过或单独附加）
- [ ] 如果未配置引擎，引擎专家步骤被跳过并在报告中添加注释

---

## 协议合规性

- [ ] `AskUserQuestion` 用于每个阶段转换——用户在管线推进前批准
- [ ] 所有文件写入通过 Task 委托给 sub-agent——编排器不直接调用 Write 或 Edit
- [ ] 遵循 Error Recovery Protocol：浮现 → 评估 → 提供选项 → 部分报告
- [ ] Phase 3 agent 按 skill 规格并行启动
- [ ] 即使 agent 被 BLOCKED 也始终生成部分报告
- [ ] 判定为 COMPLETE / NEEDS WORK / BLOCKED 之一
- [ ] 输出末尾有下一步：`/code-review`、`/balance-check`、`/team-polish`

---

## 覆盖说明

- NEEDS WORK 判定路径（qa-tester 在 Phase 5 中发现失败）未在此单独测试；
  它遵循与用例 2 相同的错误恢复和部分报告协议。
- "用更窄范围重试"错误恢复选项在断言中列出，但其完整
  递归行为（通过 `/create-stories` 拆分）由 `/create-stories` 规格覆盖。
- Phase 4 集成逻辑（连接 gameplay、AI、VFX、音频）由
  正常路径用例隐式验证；专门的集成测试需要 fixture 代码文件。
- 引擎专家不可用（无引擎配置）部分由用例 5
  断言覆盖——未配置引擎状态的专门 fixture 将加强覆盖。
