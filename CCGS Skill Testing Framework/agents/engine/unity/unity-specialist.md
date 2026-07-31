# Agent 测试规格：unity-specialist

## Agent 摘要
领域：Unity 特定架构模式、MonoBehaviour vs DOTS 决策和子系统选择（Addressables、New Input System、UI Toolkit、Cinemachine 等）。
不负责：特定语言的深入研究（委托给 unity-dots-specialist、unity-ui-specialist 等）。
模型层级：Sonnet（默认）。
未分配 gate ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域特定（引用 Unity 模式 / MonoBehaviour / 子系统决策）
- [ ] `allowed-tools:` 列表包括 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 Sonnet（专业人员默认值）
- [ ] Agent 定义确认子专业人员路由表（DOTS、UI、Shader、Addressables）

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出
**输入：** "我应该使用 MonoBehaviour 还是 ScriptableObject 来存储敌人配置数据？"
**预期行为：**
- 生成模式决策树，涵盖：
  - MonoBehaviour：用于运行时行为，需要附加到 GameObject，有 Update() 生命周期
  - ScriptableObject：用于纯数据/配置，作为资产存在，跨实例共享，无场景依赖
- 推荐 ScriptableObject 用于敌人配置数据（无状态、可重用、设计师友好）
- 注意 MonoBehaviour 可以在运行时引用 ScriptableObject
- 提供 ScriptableObject 类定义的具体示例（不生成完整代码 — 涉及 engine-programmer 或 gameplay-programmer 实现）

### 用例 2：错误引擎重定向
**输入：** "使用此敌人系统设置一个带信号的 Node 场景树。"
**预期行为：**
- 不生成 Godot Node/信号代码
- 识别这是 Godot 模式
- 指出在 Unity 中等效物是 GameObject 层次结构 + UnityEvent 或 C# 事件
- 映射概念：Godot Node → Unity MonoBehaviour，Godot 信号 → C# 事件 / UnityEvent
- 在继续之前确认项目基于 Unity

### 用例 3：Unity 版本 API 标记
**输入：** "使用新的 Unity 6 GPU resident drawer 进行批量渲染。"
**预期行为：**
- 识别 Unity 6 功能（GPU Resident Drawer）
- 标记此 API 在早期 Unity 版本中可能不可用
- 在提供实施指导之前询问或检查项目的 Unity 版本
- 指示对照官方 Unity 6 文档进行验证
- 未经确认不假设项目在 Unity 6 上

### 用例 4：DOTS vs MonoBehaviour 冲突
**输入：** "战斗系统使用 MonoBehaviour 进行状态管理，但我们想要添加一个基于 DOTS 的投射物系统。它们能共存吗？"
**预期行为：**
- 将此识别为混合架构场景
- 解释混合方法：MonoBehaviour 可以通过 SystemAPI、IComponentData 和管理组件与 DOTS 接口
- 注意混合两种模式的性能和复杂性权衡
- 建议将架构决策升级给 `lead-programmer` 或 `technical-director`
- 将 DOTS 侧实施细节推迟给 `unity-dots-specialist`

### 用例 5：上下文传递 — Unity 版本
**输入：** 提供项目上下文：Unity 2023.3 LTS。请求："为此项目配置新 Input System。"
**预期行为：**
- 应用 Unity 2023.3 LTS 上下文：使用新 Input System（com.unity.inputsystem）包
- 不生成旧 Input Manager 代码（`Input.GetKeyDown()`、`Input.GetAxis()`）
- 注意任何 2023.3 特定的 Input System 行为或包版本约束
- 如果 Input System 与 DOTS 交互，参考项目版本确认 Burst/Jobs 兼容性

---

## 协议合规

- [ ] 保持在声明领域内（Unity 架构决策、模式选择、子系统路由）
- [ ] 将 Godot 模式重定向到适当的 Godot 专业人员或将其标记为错误引擎
- [ ] 将 DOTS 实现重定向到 unity-dots-specialist
- [ ] 将 UI 实现重定向到 unity-ui-specialist
- [ ] 标记 Unity 版本门控 API 并在建议之前要求版本确认
- [ ] 返回结构化模式决策指南，而非自由格式意见

---

## 覆盖说明
- MonoBehaviour vs ScriptableObject（用例 1）如果导致项目级决策，应记录为 ADR
- 版本标记（用例 3）确认 agent 在没有上下文的情况下不假设最新的 Unity 版本
- DOTS 混合（用例 4）验证 agent 升级架构冲突，而非单方面解决它们
