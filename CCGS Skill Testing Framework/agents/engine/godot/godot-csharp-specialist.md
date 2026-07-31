# Agent 测试规格：godot-csharp-specialist

## Agent 摘要
领域：Godot 4 中的 C# 模式、应用于 Godot 的 .NET 习惯用法、[Export] 属性使用、信号委托和 async/await 模式。
不负责：GDScript 代码（gdscript-specialist）、GDExtension C/C++ 绑定（gdextension-specialist）。
模型层级：Sonnet（默认）。
未分配 gate ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域特定（引用 Godot 4 中的 C# / .NET 模式 / 信号委托）
- [ ] `allowed-tools:` 列表包括 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 Sonnet（专业人员默认值）
- [ ] Agent 定义不声称对 GDScript 或 GDExtension 代码有权限

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出
**输入：** "创建一个带有验证的敌人生命值导出属性，将其限制在 1 到 1000 之间。"
**预期行为：**
- 生成带有 `[Export]` 属性的 C# 属性
- 使用带有属性 getter/setter 的后备字段，在 setter 中限制值
- 不使用没有验证的原始 `[Export]` 公共字段
- 遵循 Godot 4 C# 命名约定（属性使用 PascalCase，字段使用下划线前缀的私有字段）
- 根据编码标准在属性上包含 XML 文档注释

### 用例 2：领域外请求 — 正确重定向
**输入：** "用 GDScript 重写这个敌人生命值系统。"
**预期行为：**
- 不生成 GDScript 代码
- 明确指出 GDScript 编写属于 `godot-gdscript-specialist`
- 将请求重定向到 `godot-gdscript-specialist`
- 可能注意 C# 接口可以被描述，以便 gdscript-specialist 知道预期的 API 形状

### 用例 3：异步信号等待
**输入：** "使用 C# async 等待动画完成后再转换游戏状态。"
**预期行为：**
- 生成使用 `ToSignal()` 等待 Godot 信号的适当 `async Task` 模式
- 使用 `await ToSignal(animationPlayer, AnimationPlayer.SignalName.AnimationFinished)`
- 不使用 `Thread.Sleep()` 或 `Task.Delay()` 作为轮询替代
- 注意调用方法必须是 `async`，并且 fire-and-forget `async void` 仅对事件处理程序可接受
- 如果动画可能无法触发，处理取消或超时

### 用例 4：线程模型冲突
**输入：** "此 C# 代码从后台 Task 线程访问 Godot Node 以更新其位置。"
**预期行为：**
- 将此标记为竞态条件风险：Godot 节点不是线程安全的，只能从主线程访问
- 不批准或实现多线程节点访问模式
- 提供正确模式：使用 `CallDeferred()`、`Callable.From().CallDeferred()` 或通过线程安全队列封送回主线程
- 解释 Godot 主线程要求与 .NET 线程无关类型之间的区别

### 用例 5：上下文传递 — Godot 4.6 API 正确性
**输入：** 引擎版本上下文：Godot 4.6。请求："使用新的类型化信号委托模式连接信号。"
**预期行为：**
- 生成使用 Godot 4 C# 中引入的类型化委托模式的 C# 信号连接（类型化信号上的 `+=` 运算符）
- 检查 4.6 上下文以确认 4.4、4.5 或 4.6 中信号委托 API 没有破坏性更改
- 不使用旧的基于字符串的 `Connect("signal_name", callable)` 模式（在 Godot 4 C# 中已弃用）
- 生成与项目固定的 4.6 版本兼容的代码，如 VERSION.md 中记录的

---

## 协议合规

- [ ] 保持在声明领域内（Godot 4 中的 C# — 模式、导出、信号、async）
- [ ] 将 GDScript 请求重定向到 godot-gdscript-specialist
- [ ] 将 GDExtension 请求重定向到 godot-gdextension-specialist
- [ ] 返回遵循 Godot 4 约定的 C# 代码（不是 Unity MonoBehaviour 模式）
- [ ] 将多线程 Godot 节点访问标记为不安全并提供正确模式
- [ ] 使用类型化信号委托 — 不是已弃用的基于字符串的 Connect() 调用
- [ ] 在生成代码之前检查引擎版本参考中的 API 更改

---

## 覆盖说明
- 带验证的导出属性（用例 1）应有一个验证限制行为的单元测试
- 线程冲突（用例 4）是安全关键的：agent 必须识别并修复此问题而不需要提示
- 异步信号（用例 3）验证 agent 在 Godot 的单线程约束内正确应用 .NET 习惯用法
