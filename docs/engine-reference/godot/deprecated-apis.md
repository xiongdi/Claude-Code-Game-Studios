# Godot — 已弃用 API

Last verified: 2026-02-12

如果 agent 建议使用 "Deprecated" 列中的任何 API，必须替换为 "Use Instead" 列中的内容。

## 节点与类

| Deprecated | Use Instead | Since | Notes |
|------------|-------------|-------|-------|
| `TileMap` | `TileMapLayer` | 4.3 | 每层一个节点，而非多层节点 |
| `VisibilityNotifier2D` | `VisibleOnScreenNotifier2D` | 4.0 | 为清晰而重命名 |
| `VisibilityNotifier3D` | `VisibleOnScreenNotifier3D` | 4.0 | 为清晰而重命名 |
| `YSort` | `Node2D.y_sort_enabled` | 4.0 | Node2D 上的属性，而非独立节点 |
| `Navigation2D` / `Navigation3D` | `NavigationServer2D` / `NavigationServer3D` | 4.0 | 基于 Server 的 API |
| `EditorSceneFormatImporterFBX` | `EditorSceneFormatImporterFBX2GLTF` | 4.3 | 重命名 |

## 方法与属性

| Deprecated | Use Instead | Since | Notes |
|------------|-------------|-------|-------|
| `yield()` | `await signal` | 4.0 | GDScript 2.0 协程语法 |
| `connect("signal", obj, "method")` | `signal.connect(callable)` | 4.0 | 基于 Callable 的连接 |
| `instance()` | `instantiate()` | 4.0 | 重命名 |
| `PackedScene.instance()` | `PackedScene.instantiate()` | 4.0 | 重命名 |
| `get_world()` | `get_world_3d()` | 4.0 | 显式 2D/3D 拆分 |
| `OS.get_ticks_msec()` | `Time.get_ticks_msec()` | 4.0 | 优先使用 Time singleton |
| `duplicate()` for nested resources | `duplicate_deep()` | 4.5 | 显式深拷贝控制 |
| `Skeleton3D` signal `bone_pose_updated` | `skeleton_updated` | 4.3 | 重命名 |
| `AnimationPlayer.method_call_mode` | `AnimationMixer.callback_mode_method` | 4.3 | 移到基类 |
| `AnimationPlayer.playback_active` | `AnimationMixer.active` | 4.3 | 移到基类 |

## 模式（不仅仅是 API）

| Deprecated Pattern | Use Instead | Why |
|--------------------|-------------|-----|
| 基于字符串的 `connect()` | 类型化 signal 连接 | 类型安全，便于重构 |
| 在 `_process()` 中使用 `$NodePath` | `@onready var` 缓存引用 | 性能：每帧路径查找 |
| 无类型的 `Array` / `Dictionary` | `Array[Type]`、类型化变量 | GDScript 编译器优化 |
| Shader 参数中使用 `Texture2D` | `Texture` 基类型 | 4.4 中变更 |
| 手动后处理 viewport 链 | `Compositor` + `CompositorEffect` | 结构化后处理（4.3+） |
| 新项目使用 GodotPhysics3D | Jolt Physics 3D | 4.6 起为默认值；更好的稳定性 |
