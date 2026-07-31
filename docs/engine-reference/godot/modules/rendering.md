# Godot Rendering — 快速参考

Last verified: 2026-02-12 | Engine: Godot 4.6

## 自 ~4.3 版本（LLM 截止）以来的变更

### 4.6 变更
- **D3D12 是 Windows 上的默认渲染后端**（之前是 Vulkan）
- **Glow 在 tonemapping 之前处理**（之前是在之后）— 使用屏幕混合模式
- **AgX tonemapper**: 新的白点和对比度控制
- **SSR 全面改进**: 更好的真实感、视觉稳定性和性能

### 4.5 变更
- **Shader Baker**: 预编译 shader 以减少启动时间
- **SMAA 1x**: 新的抗锯齿选项（比 FXAA 更清晰，比 TAA 更便宜）
- **Stencil buffer 支持**: 启用选择性几何体遮罩/传送门效果
- **Bent normal maps**: 在 normal map 纹理中编码的方向性遮挡
- **Specular occlusion**: 环境光遮蔽现在正确影响反射

### 4.4 变更
- **`RenderingDevice.draw_list_begin`**: 移除了许多参数；添加了可选的 `breadcrumb`
- **Shader 纹理类型**: 从 `Texture2D` 更改为 `Texture` 基类型
- **Particles `.restart()`**: 添加了可选的 `keep_seed` 参数

### 4.3 变更（在训练数据中）
- **Compositor 节点**: `Compositor` + `CompositorEffect` 用于后处理链

## 当前 API 模式

### 后处理（4.3+）
```gdscript
# 使用 Compositor 节点 — 而非手动 viewport shader 链
# 将 Compositor 添加为 WorldEnvironment 或 Camera3D 的子节点
# 为每个后处理步骤创建 CompositorEffect 资源
```

### 抗锯齿选项（4.6）
```
Project Settings → Rendering → Anti Aliasing:
- MSAA 2D/3D: 硬件 MSAA（高质量但昂贵）
- Screen Space AA: FXAA（快速、模糊）或 SMAA（清晰、中等开销）  # SMAA 在 4.5 中新增
- TAA: Temporal（最佳质量，快速运动时有重影）
```

### 渲染后端选择（4.6）
```
Project Settings → Rendering → Renderer:
- Forward+（默认）: 全功能，面向桌面
- Mobile: 针对移动端/低端设备优化，功能受限
- Compatibility: OpenGL 3.3 / WebGL 2，最广泛的硬件支持

Windows 默认后端: D3D12（4.6 之前为 Vulkan）
```

## 常见错误
- 假设 Vulkan 是 Windows 上的默认后端（自 4.6 起为 D3D12）
- 使用后处理时手动 viewport 链而非 Compositor
- 在 shader uniform 类型中使用 `Texture2D`（自 4.4 起使用 `Texture`）
- 对具有许多 shader 变体的项目未使用 Shader Baker
