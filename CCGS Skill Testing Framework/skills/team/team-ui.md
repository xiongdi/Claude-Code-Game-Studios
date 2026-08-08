# 技能测试规格: /team-ui

## 技能摘要

协调 UI 团队完成单个 UI 功能的完整 UX 流程。协调 ux-designer、ui-programmer、art-director、引擎 UI 专家和 accessibility-specialist 五个结构化阶段：Context Gathering + UX Spec（Phase 1a/1b）→ UX Review Gate（Phase 1c）→ Visual Design（Phase 2）→ Implementation（Phase 3）→ Review in parallel（Phase 4）→ Polish（Phase 5）。在每个阶段转换时使用 `AskUserQuestion`。将所有文件写入委托给 sub-agent 和 sub-skill（`/ux-design`、`ui-programmer`）。生成包含 COMPLETE / BLOCKED 裁决的摘要报告，并交接给 `/ux-review`、`/code-review`、`/team-polish`。

---

## 静态断言（结构性）

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题（Phase 1a 到 Phase 5 全部存在）
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含"可以写入吗"或"File Write Protocol"语言——写入委托给 sub-agent 和 sub-skill，编排器不直接写入文件
- [ ] 末尾有下一步交接（引用 `/ux-review`、`/code-review`、`/team-polish`）
- [ ] Error Recovery Protocol 章节存在，包含全部四个恢复步骤
- [ ] 在阶段转换时使用 `AskUserQuestion` 以在继续前获取用户批准
- [ ] Phase 4 明确标记为并行（ux-designer、art-director、accessibility-specialist）
- [ ] UX Review Gate（Phase 1c）定义为阻塞门控——skill 在无 APPROVED 判定时不得进入 Phase 2
- [ ] Team Composition 列出全部五个角色（ux-designer、ui-programmer、art-director、引擎 UI 专家、accessibility-specialist）
- [ ] 引用交互模式库（`design/ux/interaction-patterns.md`）——ui-programmer 必须使用现有模式
- [ ] Phase 1a 在设计开始前读取 `design/accessibility-requirements.md`

---

## 测试用例

### 用例 1：正常路径——从 UX spec 到 polish 的完整管线成功

**Fixture：**
- `design/gdd/game-concept.md` 存在，有目标平台和预期受众
- `design/player-journey.md` 存在
- `design/ux/interaction-patterns.md` 存在，有相关模式
- `design/accessibility-requirements.md` 存在，有提交层级（如 Enhanced）
- 引擎 UI 专家在 `.claude/docs/technical-preferences.md` 中配置

**输入：** `/team-ui inventory screen`

**预期行为：**
1. Phase 1a——编排器读取 game-concept.md、player-journey.md、相关 GDD UI 部分、interaction-patterns.md、accessibility-requirements.md；为 ux-designer 总结简报
2. Phase 1b——调用 `/ux-design inventory-screen`（或直接派生 ux-designer）；使用 `ux-spec.md` 模板生成 `design/ux/inventory-screen.md`；`AskUserQuestion` 在审查前确认 spec
3. Phase 1c——调用 `/ux-review design/ux/inventory-screen.md`；返回 APPROVED；门控通过，进入 Phase 2
4. Phase 2——派生 art-director；审查完整 UX spec（不仅是线框图）；应用视觉处理；验证色彩对比度；生成带资源清单的视觉设计 spec；`AskUserQuestion` 在 Phase 3 前确认
5. Phase 3——首先派生引擎 UI 专家（从 technical-preferences.md 读取）；为 ui-programmer 生成实现说明；派生 ui-programmer，附带 UX spec + 视觉 spec + 引擎说明；生成实现；如果引入新模式则更新 interaction-patterns.md
6. Phase 4——并行派生 ux-designer、art-director、accessibility-specialist；三者在 Phase 5 前全部返回结果
7. Phase 5——处理审查反馈；验证动画可跳过；通过音频事件系统确认 UI 声音；interaction-patterns.md 最终检查；判定：COMPLETE
8. 摘要报告：UX spec APPROVED、视觉设计 COMPLETE、实现 COMPLETE、无障碍 COMPLIANT、支持所有输入方法、模式库已更新、判定：COMPLETE

**断言：**
- [ ] Phase 1a 在向 ux-designer 简报前读取全部五个来源
- [ ] Phase 2 前检查 UX Review Gate——Phase 2 在 APPROVED 前不开始
- [ ] Phase 2 中的 art-director 审查完整 spec，不只是线框图图像
- [ ] Phase 3 中引擎 UI 专家在 ui-programmer 之前派生
- [ ] Phase 4 agent 同时启动（ux-designer、art-director、accessibility-specialist）
- [ ] 所有文件写入委托给 sub-agent 和 sub-skill
- [ ] 最终摘要报告中有判定 COMPLETE
- [ ] 下一步包括 `/ux-review`、`/code-review`、`/team-polish`

---

### 用例 2：UX Review Gate——Spec 审查失败；skill 在实现前停止

**Fixture：**
- `design/ux/inventory-screen.md` 由 Phase 1b 产出
- `/ux-review` 返回判定 NEEDS REVISION，标记具体问题（例如，gamepad 导航流程不完整、对比度低于最低标准）

**输入：** `/team-ui inventory screen`

**预期行为：**
1. Phase 1a + 1b 完成——产出 UX spec
2. Phase 1c——`/ux-review design/ux/inventory-screen.md` 返回 NEEDS REVISION
3. Skill 不进入 Phase 2
4. 呈现 `AskUserQuestion`，展示标记的具体问题和选项：
   - (a) 返回 ux-designer 解决问题并重新审查
   - (b) 接受风险并继续进入 Phase 2（有意识决策）
5. 如果用户选择 (a)：ux-designer 修订 spec，重新运行 `/ux-review`；循环继续直到 APPROVED 或用户覆盖
6. 如果用户选择 (b)：skill 继续，在最终报告中明确注明 NEEDS REVISION
7. Skill 不静默继续通过门控

**断言：**
- [ ] Phase 2 在 UX 审查判定为 NEEDS REVISION 时不开始
- [ ] `AskUserQuestion` 在提供选项前展示标记的具体问题
- [ ] 用户必须做出有意识的选择来覆盖——skill 不假设覆盖
- [ ] 如果用户接受风险，NEEDS REVISION 问题记录在最终报告中
- [ ] 提供修订和重新审查循环（不只是一次性失败）
- [ ] Skill 在审查失败时不丢弃已产出的 UX spec

---

### 用例 3：无参数——显示使用指引

**Fixture：**
- 任何项目状态

**输入：** `/team-ui`（无参数）

**预期行为：**
1. Skill 检测到未提供参数
2. 输出使用消息解释所需参数（UI 功能描述）
3. 提供示例调用：`/team-ui [UI feature description]`
4. Skill 退出，不派生任何 subagent 或读取任何项目文件

**断言：**
- [ ] Skill 在未提供参数时不派生任何 subagent
- [ ] 使用消息包含 frontmatter 中的 argument-hint 格式
- [ ] 显示至少一个有效调用的示例
- [ ] 失败前不读取任何 UX spec 文件或 GDD
- [ ] 不显示判定（管线从未启动）

---

### 用例 4：无障碍并行审查——Phase 4 同时运行三个流

**Fixture：**
- `design/ux/inventory-screen.md` 存在（APPROVED）
- 视觉设计 spec 完成
- 实现完成
- `design/accessibility-requirements.md` 提交层级：Enhanced

**输入：** `/team-ui inventory screen`（从 Phase 3 完成恢复）

**预期行为：**
1. Phase 4 在确认实现完成后开始
2. 同时发出三个 Task 调用：ux-designer、art-director、accessibility-specialist
3. 每个流独立运行：
   - ux-designer：验证实现与线框图匹配、测试纯键盘和纯 gamepad 导航、检查无障碍功能是否正常
   - art-director：在最低和最高支持分辨率下验证与 art bible 的视觉一致性
   - accessibility-specialist：根据 `design/accessibility-requirements.md` 中的 Enhanced 无障碍层级进行审计；任何违规标记为阻塞项
4. Skill 在进入 Phase 5 前等待所有三个结果
5. `AskUserQuestion` 在 Phase 5 开始前展示所有三个审查结果

**断言：**
- [ ] 所有三个 Task 调用在任何结果等待前发出（并行，非顺序）
- [ ] Phase 5 在所有三个 Phase 4 agent 返回前不开始
- [ ] Accessibility-specialist 明确读取 `design/accessibility-requirements.md` 获取提交层级
- [ ] 无障碍违规标记为 BLOCKING（不只是建议）
- [ ] `AskUserQuestion` 在 Phase 5 批准前一起展示所有三个审查流的结果
- [ ] 没有 Phase 4 agent 的输出被用作另一个 Phase 4 agent 的输入

---

### 用例 5：缺失交互模式库——Skill 注明缺口而不是编造模式

**Fixture：**
- `design/ux/interaction-patterns.md` 不存在
- 所有其他必需文件存在

**输入：** `/team-ui settings menu`

**预期行为：**
1. Phase 1a——编排器尝试读取 `design/ux/interaction-patterns.md`；文件未找到
2. Skill 浮现缺口："interaction-patterns.md 不存在——没有现有模式可重用"
3. 呈现 `AskUserQuestion`，选项：
   - (a) 先运行 `/ux-design patterns` 建立模式库，然后继续
   - (b) 在没有模式库的情况下继续——ux-designer 将在创建时记录新模式
4. Skill 不编造或从其他来源假设模式
5. 如果用户选择 (b)：ui-programmer 被明确指示将所有创建的模式视为新的，并在完成后将每个添加到新的 `design/ux/interaction-patterns.md`
6. 最终报告注明 interaction-patterns.md 已创建（或如果用户跳过则仍缺失）

**断言：**
- [ ] Skill 不静默忽略缺失的模式库
- [ ] Skill 不通过从功能名称或 GDD 猜测来编造模式
- [ ] `AskUserQuestion` 提供"先创建模式库"选项（引用 `/ux-design patterns`）
- [ ] 如果用户在没有库的情况下继续，ui-programmer 被告知将所有模式视为新的
- [ ] 最终报告记录模式库状态（已创建/缺失/已更新）
- [ ] Skill 不完全失败——缺口被注明并给用户选择

---

## 协议合规性

- [ ] `AskUserQuestion` 用于每个阶段转换——用户在管线推进前批准
- [ ] UX Review Gate（Phase 1c）是阻塞性的——Phase 2 在没有 APPROVED 或明确用户覆盖时不能开始
- [ ] 所有文件写入委托给 sub-agent 和 sub-skill——编排器不直接调用 Write 或 Edit
- [ ] Phase 4 agent 按 skill 规格并行启动
- [ ] 遵循 Error Recovery Protocol：浮现 → 评估 → 提供选项 → 部分报告
- [ ] 即使 agent 被 BLOCKED 也始终生成部分报告
- [ ] 判定为 COMPLETE / BLOCKED 之一
- [ ] 末尾有下一步：`/ux-review`、`/code-review`、`/team-polish`

---

## 覆盖说明

- HUD 特定路径（`/ux-design hud` + `hud-design.md` 模板 + Phase 5 中的视觉预算检查）
  未在此单独测试；它共享相同的阶段结构但使用不同的模板。
- interaction-patterns.md 的"原地更新"路径（实现期间添加新模式）
  由用例 1 Step 5 隐式练习——带有已知新模式的专门 fixture 将
  加强覆盖。
- 引擎 UI 专家不可用（无引擎配置）——skill 规格说明"如无引擎
  配置则跳过"；此路径在用例 1 中断言但未给出专门 fixture。
- NEEDS REVISION 接受风险覆盖（用例 2 选项 b）要求覆盖在报告中
  明确记录；这被断言但未进一步测试下游影响。
