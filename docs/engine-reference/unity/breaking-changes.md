# Unity 6.3 LTS — 破坏性变更

**Last verified:** 2026-02-13

本文档跟踪 Unity 2022 LTS（可能在模型训练中）与 Unity 6.3 LTS（当前版本）之间的 API 破坏性变更和行为差异。按风险级别组织。

## 高风险 — 将破坏现有代码

### Entities/DOTS API 全面革新
**Versions:** Entities 1.0+（Unity 6.0+）

```csharp
// ❌ 旧版（Unity 6 之前，GameObjectEntity 模式）
public class HealthComponent : ComponentData {
    public float Value;
}

// ✅ 新版（Unity 6+，IComponentData）
public struct HealthComponent : IComponentData {
    public float Value;
}

// ❌ 旧版：ComponentSystem
public class DamageSystem : ComponentSystem { }

// ✅ 新版：ISystem（非托管，Burst 兼容）
public partial struct DamageSystem : ISystem {
    public void OnCreate(ref SystemState state) { }
    public void OnUpdate(ref SystemState state) { }
}
```

**迁移：** 遵循 Unity 的 ECS 迁移指南。需要进行重大架构变更。

---

### Input System — 旧版输入已弃用
**Versions:** Unity 6.0+

```csharp
// ❌ 旧版：Input 类（已弃用）
if (Input.GetKeyDown(KeyCode.Space)) { }

// ✅ 新版：Input System 包
using UnityEngine.InputSystem;
if (Keyboard.current.spaceKey.wasPressedThisFrame) { }
```

**迁移：** 安装 Input System 包，将所有 `Input.*` 调用替换为新 API。

---

### URP/HDRP Renderer Feature API 变更
**Versions:** Unity 6.0+

```csharp
// ❌ 旧版：ScriptableRenderPass.Execute 签名
public override void Execute(ScriptableRenderContext context, ref RenderingData data)

// ✅ 新版：使用 RenderGraph API
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
```

**迁移：** 更新自定义渲染通道以使用 RenderGraph API。

---

## 中风险 — 行为变更

### Addressables — 资产加载返回值
**Versions:** Unity 6.2+

资产加载失败现在默认抛出异常，而不是返回 null。
添加适当的异常处理或使用 `TryLoad` 变体。

```csharp
// ❌ 旧版：失败时静默返回 null
var handle = Addressables.LoadAssetAsync<Sprite>("key");
var sprite = handle.Result; // 失败时为 null

// ✅ 新版：失败时抛出异常，使用 try/catch 或 TryLoad
try {
    var handle = Addressables.LoadAssetAsync<Sprite>("key");
    var sprite = await handle.Task;
} catch (Exception e) {
    Debug.LogError($"Failed to load: {e}");
}
```

---

### Physics — 默认求解器迭代次数变更
**Versions:** Unity 6.0+

默认求解器迭代次数增加以提高稳定性。
如果依赖旧行为，请检查 `Physics.defaultSolverIterations`。

---

## 低风险 — 弃用（仍可运行）

### UGUI（旧版 UI）
**Status:** 已弃用但受支持
**Replacement:** UI Toolkit

UGUI 仍可运行，但新项目推荐使用 UI Toolkit。

---

### 旧版粒子系统
**Status:** 已弃用
**Replacement:** Visual Effect Graph（VFX Graph）

---

### 旧版动画系统
**Status:** 已弃用
**Replacement:** Animator Controller（Mecanim）

---

## 平台特定的破坏性变更

### WebGL
- **Unity 6.0+**：WebGPU 现在是默认（WebGL 2.0 回退可用）
- 更新 shader 以兼容 WebGPU

### Android
- **Unity 6.0+**：最低 API 级别提升至 24（Android 7.0）

### iOS
- **Unity 6.0+**：最低部署目标提升至 iOS 13

---

## 迁移清单

从 2022 LTS 升级到 Unity 6.3 LTS 时：

- [ ] 审计所有 DOTS/ECS 代码（可能需要完全重写）
- [ ] 用 Input System 包替换 `Input` 类
- [ ] 将自定义渲染通道更新为 RenderGraph API
- [ ] 为 Addressables 调用添加异常处理
- [ ] 测试物理行为（求解器迭代次数已变更）
- [ ] 考虑将 UGUI 迁移到 UI Toolkit 用于新 UI
- [ ] 更新 WebGL shader 以兼容 WebGPU
- [ ] 验证最低平台版本（Android/iOS）

---

**来源：**
- https://docs.unity3d.com/6000.0/Documentation/Manual/upgrade-guides.html
- https://docs.unity3d.com/Packages/com.unity.entities@1.3/manual/upgrade-guide.html
