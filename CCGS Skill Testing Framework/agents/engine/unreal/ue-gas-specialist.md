# Agent 测试规格：ue-gas-specialist

## Agent 摘要
- **领域**：Gameplay Ability System (GAS) — 能力（UGameplayAbility）、游戏效果（UGameplayEffect）、属性集（UAttributeSet）、游戏标签、能力任务（UAbilityTask）、能力规格（FGameplayAbilitySpec）、GAS 预测和延迟补偿
- **不负责**：能力状态的 UI 显示（ue-umg-specialist）、超出内置 GAS 预测的 GAS 数据网络复制（ue-replication-specialist）、能力反馈的美术或 VFX（vfx-artist）
- **模型层级**：Sonnet
- **Gate ID**：无；将跨领域调用推迟给适当的专业人员

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域特定（引用 GAS、能力、GameplayEffects、AttributeSets）
- [ ] `allowed-tools:` 列表与 agent 角色匹配（GAS 源文件的 Read/Write；无部署或服务器工具）
- [ ] 模型层级为 Sonnet（专业人员默认值）
- [ ] Agent 定义不声称对 UI 实现或底层网络序列化有权限

---

## 测试用例

### 用例 1：领域内请求 — 带冷却的冲刺能力
**输入**："实现一个将玩家向前移动 500 单位并有 1.5 秒冷却的冲刺能力。"
**预期行为**：
- 生成 GAS AbilitySpec 结构或大纲：带有 ActivateAbility 逻辑的 UGameplayAbility 子类、用于移动的能力任务（例如 AbilityTask_ApplyRootMotionMoveToForce 或自定义根运动）以及用于冷却的 UGameplayEffect
- 冷却 GameplayEffect 使用 Duration 策略，持续 1.5 秒，并使用 GameplayTag 阻止重新激活
- 标签按照层次约定清晰命名（例如 Ability.Dash、Cooldown.Ability.Dash）
- 输出包括能力类大纲和 GameplayEffect 定义

### 用例 2：领域外请求 — GAS 状态复制
**输入**："如何将玩家的能力冷却状态复制到所有客户端，以便 UI 正确更新？"
**预期行为**：
- 澄清 GAS 通过 AbilitySystemComponent 的复制模式内置了 AbilitySpecs 和 GameplayEffects 的复制
- 解释三种 ASC 复制模式（Full、Mixed、Minimal）以及何时使用每种模式
- 对于超出 GAS 内置功能的自定义复制需求，明确指出："对于 GAS 数据的自定义网络序列化，请与 ue-replication-specialist 协调"
- 不会在没有标记领域边界的情况下尝试编写 GAS 自身系统之外的自定义复制代码

### 用例 3：领域边界 — 错误的 GameplayTag 层次结构
**输入**："我们有一个应用名为 'Stunned' 的标签的能力，另一个检查 'Status.Stunned' 的标签。它们不匹配。"
**预期行为**：
- 识别根本原因：标签名称必须精确或使用通过 TagContainer 查询的层次匹配
- 标记命名不一致：'Stunned' 是根级标签；'Status.Stunned' 是 'Status' 下的子标签 — 这些是不同的标签
- 推荐项目标签命名约定：所有状态效果在 Status.* 下，所有能力在 Ability.* 下
- 提供修复：将应用的标签重命名为 'Status.Stunned' 或将查询更新为匹配 'Stunned'
- 指出标签定义应该存在的位置（DefaultGameplayTags.ini 或 DataTable）

### 用例 4：冲突 — 两个能力之间的属性集冲突
**输入**："我们的 Shield 能力和 Armor 能力都修改 'DefenseValue' 属性。它们以非预期的方式叠加 — 两者都激活后，防御值远超最大值。"
**预期行为**：
- 将此识别为 GameplayEffect 叠加和量级计算问题
- 提出使用 Execution Calculations（UGameplayEffectExecutionCalculation）或 Modifier Aggregators 来限制组合结果的解决方案
- 或者推荐使用 Gameplay Effect 叠加策略（Aggregate、None）来防止非预期的加法叠加
- 生成具体解决方案：Execution Calculation 类大纲或 Modifier Op 的更改（对上限使用 Override 而非 Additive）
- 不会提议移除其中一个能力作为解决方案

### 用例 5：上下文传递 — 针对现有属性集进行设计
**输入上下文**：项目有一个现有的 AttributeSet，属性包括：Health、MaxHealth、Stamina、MaxStamina、Defense、AttackPower。
**输入**："设计一个 Berserker 能力，当 Health 降至 30% 以下时，AttackPower 增加 50%。"
**预期行为**：
- 使用现有的 Health、MaxHealth 和 AttackPower 属性 — 不发明新属性
- 设计一个 Passive GameplayAbility（或触发的 Effect），在 Health 变化时触发，通过 GameplayEffectExecutionCalculation 或基于属性的量级检查 Health/MaxHealth 比率
- 使用 Gameplay Cue 或 Gameplay Tag 来追踪 Berserker 激活状态
- 引用提供的 AttributeSet 中的实际属性名称（AttackPower，而非 "Damage" 或 "Strength"）

---

## 协议合规

- [ ] 保持在声明领域内（GAS：能力、效果、属性、标签、能力任务）
- [ ] 将自定义复制请求重定向到 ue-replication-specialist，并清楚解释边界
- [ ] 返回结构化发现（能力大纲 + GameplayEffect 定义）而非模糊描述
- [ ] 主动执行标签层次命名约定
- [ ] 仅使用提供的上下文中存在的属性和标签；不发明新的而不注明

---

## 覆盖说明
- 用例 3（标签层次结构）是微妙 bug 的频繁来源；每当标签命名约定更改时进行测试
- 用例 4 需要了解 GAS 叠加策略 — 如果 GAS 集成深度发生变化，请验证此用例
- 用例 5 是最重要的上下文感知测试；失败意味着 agent 忽略项目状态
- 无自动化运行器；手动或通过 `/skill-test` 审查
