# Unity 6.3 — 渲染模块参考

**最后验证时间：** 2026-02-13
**知识缺口：** LLM 基于 Unity 2022 LTS 训练；Unity 6 有重大渲染变化

---

## 概述

Unity 6.3 LTS 使用 **Scriptable Render Pipelines (SRP)** 作为现代渲染架构：
- **URP (Universal Render Pipeline)**：跨平台、移动端友好（推荐）
- **HDRP (High Definition Render Pipeline)**：高端 PC/主机、照片级真实感
- **Built-in Pipeline**：已弃用，新项目避免使用

---

## 与 2022 LTS 相比的关键变化

### RenderGraph API（Unity 6+）
自定义渲染 pass 现在使用 RenderGraph 而非 CommandBuffer：

```csharp
// ✅ Unity 6+（RenderGraph）
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData) {
    using var builder = renderGraph.AddRasterRenderPass<PassData>("MyPass", out var passData);
    builder.SetRenderFunc((PassData data, RasterGraphContext ctx) => {
        // 渲染命令
    });
}

// ❌ 旧版（CommandBuffer - 仍然可用但已弃用）
public override void Execute(ScriptableRenderContext context, ref RenderingData data) { }
```

### GPU Resident Drawer（Unity 6+）
自动批处理以大幅减少 draw call：

```csharp
// 在 URP Asset 设置中启用：
// Rendering > GPU Resident Drawer = Enabled
// 自动批处理数千个对象，CPU 开销最小
```

---

## URP 快速参考

### 创建 URP Asset
1. `Assets > Create > Rendering > URP Asset (with Universal Renderer)`
2. 分配到 `Project Settings > Graphics > Scriptable Render Pipeline Settings`

### URP Renderer Features
添加自定义渲染 pass：

```csharp
using UnityEngine.Rendering.Universal;

public class OutlineRendererFeature : ScriptableRendererFeature {
    OutlineRenderPass pass;

    public override void Create() {
        pass = new OutlineRenderPass();
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData data) {
        renderer.EnqueuePass(pass);
    }
}
```

---

## 材质与 Shader

### Shader Graph（可视化 Shader 编辑器）
Unity 6 Shader Graph 对所有 shader 类型都已生产就绪：

```csharp
// 创建：Assets > Create > Shader Graph > URP > Lit Shader Graph
// 无需代码，可视化节点编辑
```

### HLSL 自定义 Shader（URP）

```hlsl
// URP Lit shader 模板
Shader "Custom/URPLit" {
    Properties {
        _BaseColor ("Base Color", Color) = (1,1,1,1)
    }
    SubShader {
        Tags { "RenderType"="Opaque" "RenderPipeline"="UniversalPipeline" }

        Pass {
            Name "ForwardLit"
            Tags { "LightMode"="UniversalForward" }

            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct Attributes {
                float4 positionOS : POSITION;
            };

            struct Varyings {
                float4 positionCS : SV_POSITION;
            };

            Varyings vert(Attributes input) {
                Varyings output;
                output.positionCS = TransformObjectToHClip(input.positionOS.xyz);
                return output;
            }

            half4 frag(Varyings input) : SV_Target {
                return half4(1, 0, 0, 1); // 红色
            }
            ENDHLSL
        }
    }
}
```

---

## 光照

### 烘焙光照（Unity 6 Progressive Lightmapper）

```csharp
// 将对象标记为静态：Inspector > Static > Contribute GI
// 烘焙：Window > Rendering > Lighting > Generate Lighting
```

### 实时光照（URP）

```csharp
// 主光源（Directional）：由 URP 自动处理
// 附加光源：受 URP Asset 中 "Additional Lights" 设置限制

// 在 shader 中检查光源数量：
int lightCount = GetAdditionalLightsCount();
```

---

## 后处理

### Volume 系统（Unity 6+）

```csharp
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

// 将 Volume 组件添加到 GameObject
// 添加 Volume Profile 资源
// 配置效果：Bloom、Color Grading、Depth of Field 等

// 脚本访问：
Volume volume = GetComponent<Volume>();
if (volume.profile.TryGet<Bloom>(out var bloom)) {
    bloom.intensity.value = 2.5f;
}
```

---

## 性能

### SRP Batcher（自动批处理）

```csharp
// 启用：URP Asset > Advanced > SRP Batcher = Enabled
// 批处理使用相同 shader variant 的绘制（CPU 开销最小）
```

### GPU Instancing

```csharp
// 材质：勾选 "Enable GPU Instancing" 复选框
// 批处理相同的网格（相同材质 + 网格）

Graphics.RenderMeshInstanced(
    new RenderParams(material),
    mesh,
    0,
    matrices // NativeArray<Matrix4x4>
);
```

### 遮挡剔除

```csharp
// Window > Rendering > Occlusion Culling
// 为静态几何体烘焙遮挡数据
```

---

## 常见模式

### 自定义相机渲染

```csharp
// 获取 URP 相机数据
var cameraData = frameData.Get<UniversalCameraData>();
var camera = cameraData.camera;

// 访问渲染目标
var colorTarget = cameraData.renderer.cameraColorTargetHandle;
```

### 屏幕空间效果

```csharp
// 创建 ScriptableRendererFeature
// 在特定点注入 pass：AfterRenderingOpaques、AfterRenderingTransparents 等
```

---

## 调试

### Frame Debugger
- `Window > Analysis > Frame Debugger`
- 逐步执行 draw call，检查状态

### Rendering Debugger（Unity 6+）
- `Window > Analysis > Rendering Debugger`
- 实时查看 URP 设置、overdraw、光照

---

## 来源
- https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@17.0/manual/index.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/render-pipelines.html
