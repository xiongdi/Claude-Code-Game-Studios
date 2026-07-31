# Godot — 破坏性变更

Last verified: 2026-02-12

Godot 各版本之间的变更，重点关注 LLM 截止时间之后的变更（4.4+）。

## 4.5 → 4.6（2026 年 1 月 — 截止时间后，高风险）

| Subsystem | Change | Details |
|-----------|--------|---------|
| Physics | Jolt 现在是默认的 3D 物理引擎 | 新项目自动使用 Jolt。现有项目保留其设置。部分 HingeJoint3D 属性（如 `damp`）仅在 GodotPhysics 下有效。 |
| Rendering | Glow 在 tonemapping 之前处理 | 之前是在 tonemapping 之后。带 glow 的场景外观会发生变化。请在 WorldEnvironment 中调整 intensity/blend。 |
| Rendering | Windows 上默认使用 D3D12 | 之前是 Vulkan。为了更好的驱动兼容性。 |
| Rendering | AgX tonemapper 新增控制参数 | 新增白点和对比度参数。 |
| Core | Quaternion 初始化为单位四元数 | 之前为零。不太可能影响大多数代码，但技术上属于破坏性变更。 |
| UI | 双焦点系统 | 鼠标/触摸焦点现在与键盘/手柄焦点分离。视觉反馈因输入方式而异。 |
| Animation | IK 系统全面恢复 | CCDIK、FABRIK、Jacobian IK、Spline IK、TwoBoneIK，通过 SkeletonModifier3D 节点实现。 |
| Editor | 新的 "Modern" 主题默认 | 灰度替代蓝色调。恢复方式：Editor Settings → Interface → Theme → Style: Classic |
| Editor | "Select Mode" 快捷键变更 | 新的 "Select Mode"（v 键）防止意外变换。旧模式重命名为 "Transform Mode"（q 键）。 |
| 2D | TileMapLayer 场景 tile 旋转 | 场景 tile 现在可以像 atlas tile 一样旋转。 |
| Localization | CSV 复数形式支持 | 复数不再需要 Gettext。新增 context 列。 |
| C# | 自动字符串提取 | 翻译字符串从 C# 代码中自动提取。 |
| Plugins | 新的 EditorDock 类 | 专门用于 plugin dock 的容器，支持布局控制。 |

## 4.4 → 4.5（2025 年末 — 截止时间后，高风险）

| Subsystem | Change | Details |
|-----------|--------|---------|
| GDScript | 新增 variadic arguments | 函数可接受 `...` 任意参数 — 新语言特性 |
| GDScript | `@abstract` 装饰器 | 抽象类和方法现在可被强制实施 |
| GDScript | Script backtracing | 即使在 Release 构建中也可获得详细的调用栈 |
| Rendering | Stencil buffer 支持 | 高级视觉效果的新能力 |
| Rendering | SMAA 1x 抗锯齿 | 新的后处理 AA 选项 |
| Rendering | Shader Baker | 预编译 shader — 据报道在某些演示中启动速度快 20 倍 |
| Rendering | Bent normal maps、specular occlusion | 新的材质特性 |
| Accessibility | 屏幕阅读器支持 | Control 节点通过 AccessKit 与辅助工具配合工作 |
| Editor | 实时翻译预览 | 在编辑器中测试不同语言的 GUI 布局 |
| Physics | 3D interpolation 重构 | 从 RenderingServer 移到 SceneTree。API 不变但内部实现不同。 |
| Animation | BoneConstraint3D | 新增：AimModifier3D、CopyTransformModifier3D、ConvertTransformModifier3D |
| Resources | 新增 `duplicate_deep()` | 嵌套资源深拷贝的新显式方法 |
| Navigation | 专用 2D navigation server | 不再是 3D navigation 的代理；2D 游戏的导出体积更小 |
| UI | FoldableContainer 节点 | 用于可折叠 UI 部分的手风琴式容器 |
| UI | 递归 Control 行为 | 跨整个节点层级禁用鼠标/焦点交互 |
| Platform | visionOS 导出支持 | 新的平台目标 |
| Platform | SDL3 手柄驱动 | 将手柄处理委托给 SDL 库 |
| Platform | Android 16KB page 支持 | 针对 Android 15+ 的 Google Play 所必需 |

## 4.3 → 4.4（2025 年中 — 接近截止时间，需验证）

| Subsystem | Change | Details |
|-----------|--------|---------|
| Core | `FileAccess.store_*` 返回 `bool` | 之前是 `void`。方法：`store_8`、`store_16`、`store_32`、`store_64`、`store_buffer`、`store_csv_line`、`store_double`、`store_float`、`store_half`、`store_line`、`store_pascal_string`、`store_real`、`store_string`、`store_var` |
| Core | `OS.execute_with_pipe` | 新增可选的 `blocking` 参数 |
| Core | `RegEx.compile/create_from_string` | 新增可选的 `show_error` 参数 |
| Rendering | `RenderingDevice.draw_list_begin` | 移除了许多参数；新增 `breadcrumb` 参数 |
| Rendering | Shader 纹理类型 | 参数/返回类型从 `Texture2D` 改为 `Texture` |
| Particles | `.restart()` 方法 | 新增可选的 `keep_seed` 参数（CPU/GPU 2D/3D） |
| GUI | `RichTextLabel.push_meta` | 新增可选的 `tooltip` 参数 |
| GUI | `GraphEdit.connect_node` | 新增可选的 `keep_alive` 参数 |

## 4.2 → 4.3（在训练数据中 — 低风险）

| Subsystem | Change | Details |
|-----------|--------|---------|
| Animation | `Skeleton3D.add_bone` 返回 `int32` | 之前是 `void` |
| Animation | `bone_pose_updated` signal | 被 `skeleton_updated` 替代 |
| TileMap | `TileMapLayer` 替代 `TileMap` | 每层一个节点，而非多层单节点 |
| Navigation | `NavigationRegion2D` | 移除了 `avoidance_layers`、`constrain_avoidance` 属性 |
| Editor | `EditorSceneFormatImporterFBX` | 重命名为 `EditorSceneFormatImporterFBX2GLTF` |
| Animation | AnimationMixer 基类 | AnimationPlayer 和 AnimationTree 现在继承自 AnimationMixer |
