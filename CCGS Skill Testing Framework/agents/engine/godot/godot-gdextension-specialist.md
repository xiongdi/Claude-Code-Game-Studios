# Agent 测试规格：godot-gdextension-specialist

## Agent 摘要
领域：GDExtension API、godot-cpp C++ 绑定、godot-rust 绑定、原生库集成和原生性能优化。
不负责：GDScript 代码（gdscript-specialist）、着色器代码（godot-shader-specialist）。
模型层级：Sonnet（默认）。
未分配 gate ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域特定（引用 GDExtension / godot-cpp / 原生绑定）
- [ ] `allowed-tools:` 列表包括 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 Sonnet（专业人员默认值）
- [ ] Agent 定义不声称对 GDScript 或着色器编写有权限

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出
**输入：** "通过 GDExtension 将一个 C++ 刚体物理模拟库暴露给 GDScript。"
**预期行为：**
- 生成使用 godot-cpp 的 GDExtension 绑定模式：
  - 从 `godot::Object` 或适当的 Godot 基类继承的类
  - `GDCLASS` 宏注册
  - 将物理 API 暴露给 GDScript 的 `_bind_methods()` 实现
  - `GDExtension` 入口点（`gdextension_init`）设置
- 注意所需的 `.gdextension` 清单文件格式
- 不生成 GDScript 使用代码（那属于 gdscript-specialist）

### 用例 2：领域外重定向
**输入：** "编写从用例 1 调用物理模拟的 GDScript。"
**预期行为：**
- 不生成 GDScript 代码
- 明确指出 GDScript 编写属于 `godot-gdscript-specialist`
- 重定向到 `godot-gdscript-specialist`
- 可能描述 GDScript 应该调用的 API 表面（方法名称、参数类型）作为交接规格

### 用例 3：ABI 兼容性风险 — 小版本更新
**输入：** "我们正在从 Godot 4.5 升级到 4.6。我们现有的 GDExtension 还能工作吗？"
**预期行为：**
- 标记 ABI 兼容性关注点：GDExtension 二进制文件可能跨小版本不 ABI 兼容
- 指示检查 4.5→4.6 迁移指南中的 GDExtension API 更改
- 建议针对 4.6 godot-cpp 头文件重新编译扩展，而不是假设二进制兼容性
- 注意 `.gdextension` 清单可能需要 `compatibility_minimum` 版本更新
- 提供重新编译检查清单

### 用例 4：内存管理 — Godot 对象的 RAII
**输入：** "我们应该如何管理在 C++ GDExtension 代码中创建的 Godot 对象的生命周期？"
**预期行为：**
- 生成 Godot 对象在 GDExtension 中的基于 RAII 的生命周期模式：
  - `Ref<T>` 用于引用计数对象（当 Ref 超出作用域时自动释放）
  - `memnew()` / `memdelete()` 用于非引用计数对象
  - 警告：不要对 Godot 对象使用 `new`/`delete` — 未定义行为
- 注意对象所有权规则：谁负责释放添加到场景树的节点
- 提供一个管理在 C++ 中创建的 `CollisionShape3D` 的具体示例

### 用例 5：上下文传递 — Godot 4.6 GDExtension API 检查
**输入：** 引擎版本上下文：Godot 4.6（从 4.5 升级）。请求："检查从 4.5 到 4.6 是否有任何 GDExtension API 更改。"
**预期行为：**
- 参考 VERSION.md 验证来源列表中的 4.5→4.6 迁移指南
- 报告 4.6 版本中记录的任何 GDExtension API 更改
- 如果 4.6 中没有记录 GDExtension 的破坏性更改，则明确说明此情况，并附带对照官方 changelog 验证的注意事项
- 标记 Windows 上的 D3D12 默认值（4.6 更改）可能与 GDExtension 渲染代码相关
- 提供升级后要验证的内容检查清单

---

## 协议合规

- [ ] 保持在声明领域内（GDExtension、godot-cpp、godot-rust、原生绑定）
- [ ] 将 GDScript 编写重定向到 godot-gdscript-specialist
- [ ] 将着色器编写重定向到 godot-shader-specialist
- [ ] 返回结构化输出（绑定模式、RAII 示例、ABI 检查清单）
- [ ] 在小版本升级时标记 ABI 兼容性风险 — 永远不假设二进制兼容性
- [ ] 使用 Godot 特定内存管理（`memnew`/`memdelete`、`Ref<T>`），而非原始 C++ new/delete
- [ ] 在确认兼容性之前检查引擎版本参考中的 GDExtension API 更改

---

## 覆盖说明
- 绑定模式（用例 1）应包括一个冒烟测试，验证扩展可从 GDScript 加载和调用方法
- ABI 风险（用例 3）是关键升级路径 — agent 绝不能批准发布未经验证的扩展二进制文件
- 内存管理（用例 4）验证 agent 应用 Godot 特定模式，而非通用 C++ RAII
