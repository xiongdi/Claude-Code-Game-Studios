---
paths:
  - "assets/shaders/**"
---

# Shader 代码标准

`assets/shaders/` 中的所有 shader 文件必须遵循以下标准，以保持
视觉质量、性能和跨平台兼容性。

## 命名约定
- 文件命名：`[type]_[category]_[name].[ext]`
  - `spatial_env_water.gdshader`（Godot）
  - `SG_Env_Water`（Unity Shader Graph）
  - `M_Env_Water`（Unreal Material）
- 使用能表明材质用途的描述性名称
- 以 shader 类型作为前缀：`spatial_`、`canvas_`、`particles_`、`post_`

## 代码质量
- 所有 uniform/参数必须具有描述性名称和适当的 hint
- 对相关参数进行分组（Godot：`group_uniforms`，Unity：`[Header]`，Unreal：Category）
- 为不明显的计算添加注释（尤其是数学密集部分）
- 不允许使用魔法数字 — 使用命名常量或文档化的 uniform 值
- 在每个 shader 文件顶部添加作者和用途注释

## 性能要求
- 记录每个 shader 的目标平台和复杂度预算
- 在适当的精度下使用：移动端不需要全精度时使用 `half`/`mediump`
- 尽量减少 fragment shader 中的纹理采样
- 避免 fragment shader 中的动态分支 — 使用 `step()`、`mix()`、`smoothstep()`
- 循环内不允许纹理读取
- 模糊效果使用两步法（先水平后垂直）

## 跨平台
- 在最低规格的目标硬件上测试 shader
- 为较低质量层级提供 fallback/简化版本
- 记录 shader 针对的渲染管线（Forward/Deferred、URP/HDRP、Forward+/Mobile/Compatibility）
- 不在同一目录中混合来自不同渲染管线的 shader

## 变体管理
- 尽量减少 shader 变体 — 每个变体都是独立编译的 shader
- 记录所有关键字/变体及其用途
- 尽可能使用功能剥离来减少构建体积
- 记录并监控每个 shader 的总变体数量
