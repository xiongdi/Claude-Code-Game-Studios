# Agent 测试规格：unreal-specialist

## Agent 摘要
- **领域**：Unreal Engine 模式和架构 — Blueprint vs C++ 决策、UE 子系统（GAS、Enhanced Input、Niagara）、UE 项目结构、插件集成和引擎级配置
- **不负责**：艺术风格和视觉方向（art-director）、服务器基础设施和部署（devops-engineer）、UI/UX 流程设计（ux-designer）
- **模型层级**：Sonnet
- **Gate ID**：无；将门控裁决推迟给 technical-director

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域特定（引用 Unreal Engine）
- [ ] `allowed-tools:` 列表与 agent 角色匹配（UE 项目文件的 Read、Write；无部署工具）
- [ ] 模型层级为 Sonnet（专业人员默认值）
- [ ] Agent 不声称在其声明领域之外有权限（无美术、无服务器基础设施）

---

## 测试用例

### 用例 1：领域内请求 — Blueprint vs C++ 决策标准
**输入**："我应该用 Blueprint 还是 C++ 实现我们的连击攻击系统？"
**预期行为**：
- 提供结构化决策标准：复杂度、重用频率、团队技能和性能要求
- 推荐 C++ 用于每帧调用或跨 5 种以上能力类型共享的系统
- 推荐 Blueprint 用于设计师可调的值和一次性逻辑
- 不知道项目上下文时不做出最终裁决 — 如果缺少上下文则提出澄清问题
- 输出结构化（标准表或项目符号列表），而非自由格式意见

### 用例 2：领域外请求 — Unity C# 代码
**输入**："给我写一个处理玩家生命值并在死亡时触发 Unity 事件的 C# MonoBehaviour。"
**预期行为**：
- 不生成 Unity C# 代码
- 明确指出："此项目使用 Unreal Engine；Unity 等效物是 UE C++ 中的 Actor Component 或 Blueprint Actor Component"
- 可选地在请求时提供 UE 等效方案
- 不重定向到 Unity 专业人员（框架中不存在）

### 用例 3：领域边界 — UE5.4 API 要求
**输入**："我需要使用 UE5.4 中引入的新 Motion Matching API。"
**预期行为**：
- 标记 UE5.4 是一个特定版本，LLM 训练覆盖可能有限
- 建议在对任何 API 建议有信心之前，交叉参考官方 Unreal 文档或项目的 engine-reference 目录
- 提供尽力而为的 API 指导，带有明确的不确定性标记（例如"对照 UE5.4 发布说明验证此内容"）
- 不会在没有警告的情况下静默生成过时或不正确的 API 签名

### 用例 4：冲突 — 核心系统中的 Blueprint 意大利面式代码
**输入**："我们的复制逻辑完全在一个深度嵌套的 Blueprint 事件图中，有 300+ 节点且没有函数。它变得无法维护。"
**预期行为**：
- 将此识别为 Blueprint 架构问题，而非小风格问题
- 建议将核心复制逻辑迁移到 C++ ActorComponent 或 GameplayAbility 系统
- 注意所需的协调：复制架构的更改必须涉及 lead-programmer
- 不会在未向用户展示重构范围的情况下单方面宣布"迁移到 C++"
- 生成具体迁移建议，而非模糊建议

### 用例 5：上下文传递 — 版本适当的 API 建议
**输入上下文**：项目 engine-reference 文件指出 Unreal Engine 5.3。
**输入**："如何为新角色设置 Enhanced Input 操作？"
**预期行为**：
- 使用 UE5.3 时代的 Enhanced Input API（InputMappingContext、UEnhancedInputComponent::BindAction）
- 不会引用 UE5.3 之后引入的 API，而不标记它们可能不可用
- 在其响应中引用项目规定的引擎版本
- 提供具体的、版本锚定的代码或 Blueprint 节点名称

---

## 协议合规

- [ ] 保持在声明领域内（Unreal 模式、Blueprint/C++、UE 子系统）
- [ ] 重定向 Unity 或其他引擎请求而不生成错误引擎代码
- [ ] 返回结构化发现（标准表、决策树、迁移计划），而非自由格式意见
- [ ] 在生成 API 建议之前明确标记版本不确定性
- [ ] 对于架构级重构与 lead-programmer 协调，而非单方面决定

---

## 覆盖说明
- Agent 行为测试不存在自动化运行器 — 这些是手动或通过 `/skill-test` 审查的
- 版本感知（用例 3、用例 5）是此 agent 的最高风险失败模式；当引擎版本更改时定期测试
- 用例 4 与 lead-programmer 的集成是协调测试，而非技术正确性测试
