# Godot — 当前最佳实践

最后验证: 2026-02-12 | 引擎: Godot 4.6

自模型训练数据（~4.3）以来**新增或变更**的实践。
这是对 agent 内置知识的补充（而非替代）。

## GDScript（4.5+）

- **Variadic arguments**：函数可接受任意数量的参数
  ```gdscript
  func log_values(prefix: String, values: Variant...) -> void:
      for v in values:
          print(prefix, ": ", v)
  ```

- **抽象类和方法**：使用 `@abstract` 强制继承
  ```gdscript
  @abstract
  class_name BaseEnemy extends CharacterBody3D

  @abstract
  func get_attack_pattern() -> Array[Attack]:
      pass  # 子类必须重写
  ```

- **Script backtracing**：即使在 Release 构建中也可获得详细的调用栈

## Physics（4.6）

- **Jolt Physics 是新项目的默认 3D 引擎**
  - 比 GodotPhysics3D 具有更好的确定性和稳定性
  - 部分 HingeJoint3D 属性（`damp`）仅在 GodotPhysics 下有效
  - 切换方式：Project Settings → Physics → 3D → Physics Engine
  - 2D physics 未变更（仍然是 Godot Physics 2D）

## Rendering（4.6）

- **D3D12 是 Windows 上的默认后端**（之前是为了更好的驱动兼容性）
- **Glow 现在在 tonemapping 之前处理**，使用屏幕混合模式 — 现有的 glow 设置可能看起来不同
- **SSR 全面革新** — 在真实感、稳定性和性能方面有显著提升
- **AgX tonemapper** — 新的白点和对比度控制

## Rendering（4.5）

- **Shader Baker**：预编译 shader 以消除启动时的卡顿
- **SMAA 1x**：新的 AA 选项 — 比 FXAA 更锐利，比 TAA 更廉价
- **Stencil buffer**：可用于高级遮罩/传送门效果
- **Bent normal maps**：法线贴图纹理中的方向性遮蔽
- **Specular occlusion**：环境遮蔽现在影响反射

## Accessibility（4.5+）

- **屏幕阅读器支持**：Control 节点通过 AccessKit 与辅助工具集成
- **实时翻译预览**：直接在编辑器中测试不同语言的 GUI 布局
- **FoldableContainer**：用于可折叠部分的新手风琴式 UI 节点
- **递归 Control 禁用**：通过单个属性禁用整个节点层级的鼠标/焦点交互

## Animation（4.5+）

- **BoneConstraint3D**：通过修改器将骨骼绑定到其他骨骼
  - AimModifier3D、CopyTransformModifier3D、ConvertTransformModifier3D

## Animation（4.6）

- **IK 系统全面恢复**：为 3D 重新引入完整的反向运动学
  - 可用修改器：CCDIK、FABRIK、Jacobian IK、Spline IK、TwoBoneIK
  - 通过 `SkeletonModifier3D` 节点应用

## Resources（4.5+）

- **`duplicate_deep()`**：用于嵌套资源树的显式深拷贝
  - 旧的 `duplicate()` 行为保留以兼容
  - 当需要嵌套资源的逐实例副本时使用 `duplicate_deep()`

## Navigation（4.5+）

- **专用 2D navigation server**：不再通过 3D NavigationServer 代理
  - 减小纯 2D 游戏的导出二进制体积

## UI（4.6）

- **双焦点系统**：鼠标/触摸焦点现在与键盘/手柄焦点分离
  - 视觉反馈因输入方式而异
  - 设计自定义焦点行为时需考虑这一点

## Editor Workflow（4.6）

- 灵活的 dock 拖放，带蓝色轮廓预览（包括底部面板）
- 大多数面板支持浮动窗口（Debugger 除外）
- 新键盘快捷键：Alt+O（Output）、Alt+S（Shader）
- 导出变量自动生成：将资源从 FileSystem 拖入脚本编辑器
- 启用 "Live Preview" 时在 Quick Open 对话框中实时预览
- 新的 "Select Mode"（v 键）防止意外变换；旧模式重命名为 "Transform Mode"（q 键）

## 工具

- **ripgrep 没有 `gdscript` 类型**：`*.gd` 注册在 `gap`（GAP 编程语言）下。
  `rg --type gdscript` 是硬错误 — 搜索永远不会执行。
  始终使用 `rg --glob "*.gd"`（shell）或 `glob: "*.gd"`（Grep 工具）来过滤 GDScript 文件。

## 平台（4.5+）

- **visionOS 导出**：自开源以来的首个新平台（窗口应用模式）
- **SDL3 手柄驱动**：更好的跨平台手柄支持
- **Android**：Edge-to-edge 显示、摄像头访问、16KB page 支持（Android 15+）
- **Linux**：Wayland 子窗口支持，实现多窗口能力 |
