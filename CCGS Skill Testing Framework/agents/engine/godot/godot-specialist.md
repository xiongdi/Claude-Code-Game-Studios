# Agent 测试规格：godot-specialist

## Agent 摘要
领域：Godot 特定模式、节点/场景架构、信号、资源以及 GDScript vs C# vs GDExtension 决策。
不负责：特定语言的实际代码编写（委托给语言子专业人员）。
模型层级：Sonnet（默认）。
未分配 gate ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域特定（引用 Godot 架构 / 节点模式 / 引擎决策）
- [ ] `allowed-tools:` 列表包括 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 Sonnet（专业人员默认值）
- [ ] Agent 定义引用 `docs/engine-reference/godot/VERSION.md` 作为权威 API 来源

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出
**输入：** "在 Godot 中我应该何时使用信号与直接方法调用？"
**预期行为：**
- 生成带有理由的模式决策指南：
  - 信号：解耦通信、父到子无知、事件驱动 UI 更新、一对多通知
  - 直接调用：紧密耦合系统（调用者需要返回值）或性能关键热路径
- 在项目上下文中为每种模式提供具体示例
- 不为两种模式生成原始代码 — 涉及 gdscript-specialist 或 csharp-specialist 实现
- 注意"无向上信号"约定（子级不直接调用父级方法 — 使用信号代替）

### 用例 2：错误引擎重定向
**输入：** "编写一个在 Start() 运行并订阅 UnityEvent 的 MonoBehaviour。"
**预期行为：**
- 不生成 Unity MonoBehaviour 代码
- 清楚识别这是 Unity 模式，不是 Godot 模式
- 提供 Godot 等效方案：使用 `_ready()` 代替 `Start()` 的 Node 脚本，以及 Godot 信号代替 UnityEvent
- 确认项目基于 Godot 并重定向概念映射

### 用例 3：截止后 API 风险
**输入：** "使用新的 Godot 4.5 @abstract 注解来定义抽象基类。"
**预期行为：**
- 识别 `@abstract` 是截止后功能（在 Godot 4.5 中引入，LLM 知识截止后）
- 标记版本风险：LLM 对此注解的知识可能不完整或不正确
- 指示用户对照 `docs/engine-reference/godot/VERSION.md` 和官方 4.5 迁移指南进行验证
- 基于版本参考中的迁移说明提供尽力而为的指导，同时清楚标记为未验证

### 用例 4：热路径的语言选择
**输入：** "物理查询循环每帧为 500 个对象运行。我们应该为此使用 GDScript 还是 C#？"
**预期行为：**
- 提供平衡分析：
  - GDScript：更简单、团队熟悉，但对紧密循环较慢
  - C#：对 CPU 密集型循环更快，需要 .NET 运行时，团队需要 C# 知识
- 不单方面做出最终决策
- 将决策推迟给 `lead-programmer`，分析作为输入
- 注意 GDExtension（C++）是极端性能情况的第三个选项，如果 C# 不足够则建议升级

### 用例 5：上下文传递 — 引擎版本 4.6
**输入：** 提供引擎版本上下文：Godot 4.6，Jolt 作为默认物理。请求："为玩家角色设置 RigidBody3D。"
**预期行为：**
- 读取 4.6 上下文并应用 Jolt 默认知识（来自 VERSION.md 迁移说明）
- 推荐与 Jolt 兼容的 RigidBody3D 配置选择（例如，注意在 Jolt 下行为不同的任何 GodotPhysics 特定设置）
- 参考 4.6 迁移说明关于 Jolt 成为默认值，而非仅依赖 LLM 训练数据
- 标记在 GodotPhysics 和 Jolt 之间行为发生变化的任何 RigidBody3D 属性

---

## 协议合规

- [ ] 保持在声明领域内（Godot 架构决策、节点/场景模式、语言选择）
- [ ] 将语言特定实现重定向到 godot-gdscript-specialist 或 godot-csharp-specialist
- [ ] 返回结构化发现（决策树、带理由的模式推荐）
- [ ] 将 `docs/engine-reference/godot/VERSION.md` 视为权威，优先于 LLM 训练数据
- [ ] 标记截止后 API 使用（4.4、4.5、4.6）并要求验证
- [ ] 当存在权衡时将语言选择决策推迟给 lead-programmer

---

## 覆盖说明
- 信号与直接调用指南（用例 1）应写入 `docs/architecture/` 作为可重用模式文档
- 截止后标记（用例 3）确认 agent 不会自信地使用它无法验证的 API
- 引擎版本用例（用例 5）验证 agent 应用版本参考中的迁移说明，而非假设
