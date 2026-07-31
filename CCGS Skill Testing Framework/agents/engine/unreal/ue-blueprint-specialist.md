# Agent 测试规格：ue-blueprint-specialist

## Agent 摘要
- **领域**：Blueprint 架构、Blueprint/C++ 边界、Blueprint 图质量、Blueprint 性能优化、Blueprint Function Library 设计
- **不负责**：C++ 实现（engine-programmer 或 gameplay-programmer）、美术资产或着色器、UI/UX 流程设计（ux-designer）
- **模型层级**：Sonnet
- **Gate ID**：无；将跨领域裁决推迟给 unreal-specialist 或 lead-programmer

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域特定（引用 Blueprint 架构和优化）
- [ ] `allowed-tools:` 列表与 agent 角色匹配（Blueprint 项目文件的 Read；无服务器或部署工具）
- [ ] 模型层级为 Sonnet（专业人员默认值）
- [ ] Agent 定义不声称对 C++ 实现决策有权限

---

## 测试用例

### 用例 1：领域内请求 — Blueprint 图性能审查
**输入**："审查我们的 AI 行为 Blueprint。它有每帧运行的基于 tick 的逻辑，同时检查 30 个 NPC 的视线。"
**预期行为**：
- 识别 tick 繁重逻辑是性能问题
- 建议从 EventTick 切换到事件驱动模式（感知系统事件、定时器或降低间隔的轮询）
- 标记同时视线检查的每个 NPC 成本
- 建议替代方案：AIPerception 组件事件、交错 tick 组，或如果测量到 Blueprint 开销显著则将系统移至 C++
- 输出结构化：问题识别、影响估计、替代方案列出

### 用例 2：领域外请求 — C++ 实现
**输入**："编写此能力冷却系统的 C++ 实现。"
**预期行为**：
- 不生成 C++ 实现代码
- 提供冷却逻辑的 Blueprint 等效方案（例如，如果使用 GAS，则使用 Timeline 或 GameplayEffect）
- 明确指出："C++ 实现由 engine-programmer 或 gameplay-programmer 处理；我可以展示 Blueprint 方法或描述 Blueprint 调用 C++ 的边界"
- 可选地注意冷却复杂度何时需要 C++ 后端

### 用例 3：领域边界 — Blueprint 中不安全的原始指针访问
**输入**："我们的 Blueprint 调用 GetOwner()，然后立即访问结果上的组件而不检查是否有效。"
**预期行为**：
- 将此标记为运行时崩溃风险：GetOwner() 在某些生命周期状态下可以返回 null
- 提供正确的 Blueprint 模式：在任何属性/组件访问之前使用 IsValid() 节点
- 注意 Blueprint 的 null 检查在 Actor 派生引用上不是可选的
- 不会在不解释原始代码为何不安全的情况下静默修复代码

### 用例 4：Blueprint 图复杂度 — 准备 Function Library 重构
**输入**："我们的主 GameMode Blueprint 在单个图中有 600+ 节点，8 个地方有重复的伤害计算逻辑。"
**预期行为：**
- 将此诊断为可维护性和可测试性问题
- 建议将重复逻辑提取到 Blueprint Function Library（BFL）
- 描述如何构建 BFL：计算的纯函数，从任何 Blueprint 静态调用
- 注意如果伤害逻辑对性能敏感或与 C++ 共享，可能是迁移到 unreal-specialist 审查的候选
- 输出是具体的重构计划，而非模糊建议

### 用例 5：上下文传递 — Blueprint 复杂度预算
**输入上下文**：项目约定指定每个 Blueprint 事件图最多 100 个节点，超过后强制进行 Function Library 提取。
**输入**："这是我们的库存 Blueprint 图 [显示 150 个节点]。它准备好发布了吗？"
**预期行为**：
- 引用所述 150 节点数量与项目约定中 100 节点预算的对比
- 标记该图超出复杂度阈值
- 不会按原样批准它
- 生成候选子图列表以进行 Function Library 提取，使主图在预算内

---

## 协议合规

- [ ] 保持在声明领域内（Blueprint 架构、性能、图质量）
- [ ] 将 C++ 实现请求重定向到 engine-programmer 或 gameplay-programmer
- [ ] 返回结构化发现（问题/影响/替代方案格式），而非自由格式意见
- [ ] 主动执行 Blueprint 安全模式（null 检查、IsValid）
- [ ] 在评估图复杂度时引用项目约定

---

## 覆盖说明
- 用例 3（空指针安全）是安全关键测试 — 这是发布崩溃的常见来源
- 用例 5 要求项目约定包括规定的节点预算；如果未配置，agent 应注明缺失并建议设置一个
- 无自动化运行器；手动或通过 `/skill-test` 审查
