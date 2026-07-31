# Agent Test Spec: unity-shader-specialist

## Agent Summary
Domain: Unity Shader Graph、自定义 HLSL、VFX Graph、URP/HDRP 管线定制和后处理效果。
Does NOT own: 游戏逻辑代码、美术风格方向。
Model tier: Sonnet (default)。
No gate IDs assigned。

---

## Static Assertions (Structural)

- [ ] `description:` 字段存在且为领域特定（引用 Shader Graph / HLSL / VFX Graph / URP / HDRP）
- [ ] `allowed-tools:` 列表包含 Read, Write, Edit, Glob, Grep
- [ ] Model tier 为 Sonnet（specialist 默认值）
- [ ] Agent 定义不声称对游戏逻辑代码或美术方向拥有权限

---

## Test Cases

### Case 1: 领域内请求 — 适当的输出
**Input:** "在 URP 中使用 Shader Graph 为角色创建描边效果。"
**Expected behavior:**
- 生成 Shader Graph 节点设置描述：
  - 反转外壳法：Scale Normal → 顶点阶段顶点偏移，Cull Front
  - 或使用深度/法线边缘检测的屏幕空间后处理描边
- 根据 URP 能力推荐适当的方法（反转外壳法用于 URP 兼容性，后处理用于 HDRP）
- 注意 URP 限制：不支持几何着色器（排除几何着色器描边方法）
- 在未确认渲染管线的情况下不生成 HDRP 专用节点

### Case 2: 领域外重定向
**Input:** "用代码实现角色血条 UI。"
**Expected behavior:**
- 不生成 UI 实现代码
- 明确声明 UI 实现属于 `ui-programmer`（或 `unity-ui-specialist`）
- 适当地重定向请求
- 可以注意，如果视觉效果本身是 shader 驱动的，血条的 shader 填充效果（例如溶解/填充渐变）在其领域内

### Case 3: HDRP 自定义通道描边
**Input:** "我们在 HDRP 上，希望将描边作为后处理效果。"
**Expected behavior:**
- 生成 HDRP `CustomPassVolume` 模式：
  - 继承 `CustomPass` 的 C# 类
  - 使用 `CoreUtils.SetRenderTarget()` 和全屏 shader blit 的 `Execute()` 方法
  - 用于边缘检测的深度/法线缓冲区采样
- 注意 CustomPass 需要 HDRP 包，在 URP 中不可用
- 在提供 HDRP 专用代码前确认项目在 HDRP 上

### Case 4: VFX Graph 性能 — GPU 事件批处理
**Input:** "爆炸 VFX Graph 每个事件有 10,000 个粒子，同时生成 20 个爆炸导致 GPU 帧率飙升。"
**Expected behavior:**
- 将 GPU 粒子生成识别为成本驱动因素（200,000 个同时粒子）
- 提出 GPU 事件批处理：生成事件跨多帧延迟，错开初始化
- 建议每个活动爆炸的粒子预算上限（例如每个爆炸 3,000，超出排队）
- 注意 VFX Graph Event Batcher 模式和用于跨帧分发的 Output Event API
- 不修改游戏事件系统 — 提出 VFX 端预算方案

### Case 5: 上下文传递 — 渲染管线（URP 或 HDRP）
**Input:** 项目上下文：URP 渲染管线，Unity 2022.3。请求："添加景深后处理。"
**Expected behavior:**
- 使用 URP Volume 框架：`DepthOfField` Volume Override 组件
- 不使用 HDRP Volume 组件（例如 HDRP 的 `DepthOfField`，参数名不同）
- 注意 URP 特定的 DOF 限制与 HDRP 的差异（例如 Bokeh 质量差异）
- 生成与 Unity 2022.3 URP 包版本兼容的 C# Volume 配置文件设置代码

---

## Protocol Compliance

- [ ] 保持在声明领域内（Shader Graph、HLSL、VFX Graph、URP/HDRP 定制）
- [ ] 将游戏逻辑和 UI 代码重定向到适当的 agent
- [ ] 返回结构化输出（节点图描述、HLSL 代码、CustomPass 模式）
- [ ] 区分 URP 和 HDRP 方法 — 绝不交叉污染管线专用 API
- [ ] 在相关时将几何着色器方法标记为 URP 不兼容
- [ ] 生成不改变游戏逻辑行为的 VFX 优化

---

## Coverage Notes
- 描边效果（Case 1）应与 `production/qa/evidence/` 中的视觉截图测试配对
- HDRP CustomPass（Case 3）确认 agent 生成正确的 Unity 模式，而非通用后处理方法
- 管线分离（Case 5）验证 agent 从不假设渲染管线而不依赖上下文
