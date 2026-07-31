# Agent Test Spec: analytics-engineer

## Agent Summary
- **领域：** 遥测架构和事件 schema 设计、A/B 测试框架设计、玩家行为分析方法、分析仪表板规格、事件命名约定、数据管线设计（schema → 摄入 → 仪表板）
- **不负责：** 事件跟踪的游戏实现（相应的程序员）、由分析告知的经济设计决策（economy-designer）、live ops 事件设计（live-ops-designer）
- **模型层级：** Sonnet
- **Gate ID：** 无；生成 schema 和测试设计；将实现推迟给程序员

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且特定于领域（引用遥测、A/B 测试、事件跟踪、分析）
- [ ] `allowed-tools:` 列表与 agent 角色匹配（对 design/analytics/ 和文档的 Read/Write；无游戏源或 CI 工具）
- [ ] 模型层级为 Sonnet（operations specialist 默认值）
- [ ] Agent 定义不声称对游戏实现、经济设计或 live ops 排期拥有权限

---

## 测试用例

### 用例 1：领域内请求 — 教程事件跟踪设计
**输入：** "为我们的教程设计分析事件跟踪。我们想知道玩家在哪里流失以及他们完成了哪些步骤。"
**预期行为：**
- 为每个教程步骤生成结构化事件 schema：至少包含 `event_name`、`properties`（step_id、step_name、player_id、session_id、timestamp）和 `trigger_condition`（事件何时触发 — 步骤开始时、步骤完成时、步骤跳过时）
- 包括漏斗完成事件和流失事件（例如，如果玩家在步骤期间退出则为 `tutorial_step_abandoned`）
- 指定事件命名约定：snake_case，以域名为前缀（例如，`tutorial_step_started`、`tutorial_step_completed`、`tutorial_abandoned`）
- 不生成实现代码 — 将实现标记为 [TO BE IMPLEMENTED BY PROGRAMMER]
- 输出为 schema 表或结构化列表，而非叙述性描述

### 用例 2：领域外请求 — 在代码中实现事件跟踪
**输入：** "现在事件 schema 已设计好，编写 GDScript 代码以在我们的 Godot 教程场景中触发这些事件。"
**预期行为：**
- 不生成 GDScript 或任何实现代码
- 清晰声明："游戏代码中的遥测实现由相应的程序员（gameplay-programmer 或 systems-programmer）处理；我提供事件 schema 和集成要求"
- 可选地生成集成规格：程序员需要知道什么才能正确实现（事件名称、属性、何时触发、使用什么分析 SDK 或端点）

### 用例 3：领域边界 — UI 变更的 A/B 测试设计
**输入：** "我们想对我们的 HUD 两个版本进行 A/B 测试：当前版本和只有血条的最小版本。设计测试。"
**预期行为：**
- 生成完整的 A/B 测试设计文档：
  - **假设：** 最小 HUD 将通过减少 UI 认知负载来增加玩家参与度（以会话长度衡量）
  - **主要指标：** 每位玩家的平均会话长度
  - **次要指标：** 教程完成率、第 1 天留存
  - **样本量：** 基于预期效果大小的计算估计（或注明精确计算需要基线数据）— 不跳过此字段
  - **持续时间：** 最短持续时间（例如，"至少 2 周以捕获每周玩家行为模式"）
  - **随机化单位：** 玩家 ID（而非会话 ID，以防止玩家看到两个版本）
- 输出结构化为正式测试设计，而非想法的项目符号列表

### 用例 4：冲突 — 重叠的 A/B 测试玩家群体
**输入：** "我们有两个同时运行的 A/B 测试：测试 A（HUD 变体）影响所有玩家，测试 B（教程变体）也影响所有玩家。"
**预期行为：**
- 将重叠标记为互斥违规：如果两个测试影响同一玩家，它们的结果会混淆 — 两个测试都不产生干净数据
- 精确识别问题：两个测试中的玩家将同时具有 HUD 和教程变体，使得无法将结果差异归因于任一变量
- 提出解决方案：(a) 顺序运行测试，(b) 将玩家群体分成互斥段（50% 在测试 A，50% 在测试 B，0% 在两者中），或者 (c) 如果交互效应也感兴趣则运行因子设计（更复杂，需要更大样本）
- 不建议在重叠群体上继续两个测试

### 用例 5：上下文传递 — 与现有 schema 一致的新事件
**输入上下文：** 现有事件 schema 使用命名约定：`[domain]_[object]_[action]`，采用 snake_case。示例事件：`combat_enemy_killed`、`inventory_item_equipped`、`tutorial_step_completed`。
**输入：** "为我们的新制作系统（玩家收集材料、打开制作菜单和制作物品）设计事件跟踪。"
**预期行为：**
- 生成遵循提供的 schema 中确切命名约定的事件：`crafting_material_gathered`、`crafting_menu_opened`、`crafting_item_crafted`
- 不发明不同的命名模式（例如，`gatherMaterial`、`craftingOpened`），即使它可能看起来自然
- 属性遵循与现有事件相同的结构：`player_id`、`session_id`、`timestamp` 作为标准字段；域特定字段（material_type、item_id、crafting_time_seconds）作为附加属性
- 输出明确引用提供的命名约定作为遵循的标准

---

## 协议合规

- [ ] 在声明领域内保持（事件 schema 设计、A/B 测试设计、分析方法）
- [ ] 将实现请求重定向到相应的程序员并附带集成规格，而非代码
- [ ] 生成完整的 A/B 测试设计（假设、指标、样本量、持续时间、随机化单位）— 从不部分
- [ ] 将重叠 A/B 测试中的互斥违规标记为数据质量阻塞
- [ ] 完全遵循提供的命名约定；不发明替代约定

---

## 覆盖说明
- 用例 3（A/B 测试设计完整性）是质量 gate — 不完整的测试设计浪费实验预算
- 用例 4（互斥）是数据完整性测试 — 重叠测试产生不可用结果；这必须被捕获
- 用例 5 是最重要的上下文感知测试；跨 schema 的命名约定漂移导致仪表板故障
- 无自动化运行器；手动审查或通过 `/skill-test`
