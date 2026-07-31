# Agent 测试规格：godot-gdscript-specialist

## Agent 摘要
领域：GDScript 静态类型、GDScript 中的设计模式、信号架构、协程/await 模式和 GDScript 性能。
不负责：着色器代码（godot-shader-specialist）、GDExtension 绑定（godot-gdextension-specialist）。
模型层级：Sonnet（默认）。
未分配 gate ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域特定（引用 GDScript / 静态类型 / 信号 / 协程）
- [ ] `allowed-tools:` 列表包括 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 Sonnet（专业人员默认值）
- [ ] Agent 定义不声称对着色器代码或 GDExtension 有权限

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出
**输入：** "审查此 GDScript 文件的类型注解覆盖。"
**预期行为：**
- 读取提供的 GDScript 文件
- 标记每个缺少静态类型注解的变量、参数和返回类型
- 生成具体的逐行发现列表：`var speed = 5.0` → `var speed: float = 5.0`
- 注意 Godot 4 中静态类型的性能和工具好处
- 不会未经提示就重写整个文件 — 生成发现列表供开发者应用

### 用例 2：领域外请求 — 正确重定向
**输入：** "编写一个在世界空间中扭曲网格的顶点着色器。"
**预期行为：**
- 不在 GDScript 或 Godot 的着色器语言中生成着色器代码
- 明确指出着色器编写属于 `godot-shader-specialist`
- 将请求重定向到 `godot-shader-specialist`
- 可能注意 GDScript 方面（将 uniform 传递给着色器、设置着色器参数）在其领域内

### 用例 3：使用协程的异步加载
**输入：** "异步加载一个场景并在生成之前等待它完成。"
**预期行为：**
- 生成 Godot 4 的 `await` + `ResourceLoader.load_threaded_request` 模式
- 全程使用静态类型（`var scene: PackedScene`）
- 使用 `ResourceLoader.load_threaded_get_status()` 处理完成检查
- 注意加载失败时的错误处理
- 不使用已弃用的 Godot 3 `yield()` 语法

### 用例 4：性能问题 — 类型化数组推荐
**输入：** "实体更新循环很慢；它每帧迭代 1,000 个节点的未类型化 Array。"
**预期行为：**
- 识别未类型化 `Array` 放弃了 GDScript 中的编译器优化
- 推荐转换为类型化数组（`Array[Node]` 或特定类型）以启用 JIT 提示
- 注意如果这仍然不足，将热路径升级到 C# 迁移推荐
- 生成类型化数组重构作为立即修复
- 不会在没有分析证据的情况下推荐将整个代码库迁移到 C#

### 用例 5：上下文传递 — 带有截止后功能的 Godot 4.6
**输入：** 提供引擎版本上下文：Godot 4.6。请求："使用 @abstract 为所有敌人类型创建一个抽象基类。"
**预期行为：**
- 识别 `@abstract` 是 Godot 4.5+ 功能（截止后）
- 在输出中注意：功能在 4.5 中引入，对照 VERSION.md 迁移说明验证
- 使用 `@abstract` 生成 GDScript 类，使用迁移说明中记录的正确语法
- 由于截止后状态，将输出标记为需要对照官方 4.5 发布说明进行验证
- 对抽象类中的所有方法签名使用静态类型

---

## 协议合规

- [ ] 保持在声明领域内（GDScript — 类型、模式、信号、协程、性能）
- [ ] 将着色器请求重定向到 godot-shader-specialist
- [ ] 将 GDExtension 请求重定向到 godot-gdextension-specialist
- [ ] 返回带有完整静态类型的结构化 GDScript 输出
- [ ] 仅使用 Godot 4 API — 无已弃用的 Godot 3 模式（yield、使用字符串连接等）
- [ ] 标记截止后功能（4.4、4.5、4.6）并标记为需要文档验证

---

## 覆盖说明
- 类型注解审查（用例 1）输出适合作为代码审查检查清单
- 异步加载（用例 3）应生成可在 `tests/unit/` 中用单元测试验证的可测试代码
- 截止后 @abstract（用例 5）确认 agent 标记版本不确定性，而不是静默使用未验证的 API
