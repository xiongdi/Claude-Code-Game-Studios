# Unity 6.3 — Cinemachine

**最后验证时间：** 2026-02-13
**状态：** 生产就绪
**包：** `com.unity.cinemachine` v3.0+（Package Manager）

---

## 概述

**Cinemachine** 是 Unity 的虚拟相机系统，无需手动脚本即可实现专业、动态的相机行为。它是 Unity 相机工作的行业标准。

**将 Cinemachine 用于：**
- 第三人称跟随相机
- 过场动画和电影摄影
- 相机混合和过渡
- 动态相机取景
- 屏幕震动和相机效果

**⚠️ 知识缺口：** Cinemachine 3.0（Unity 6）是从 2.x 的重大重写。
许多 API 名称和组件已更改。

---

## 安装

### 通过 Package Manager 安装

1. `Window > Package Manager`
2. Unity Registry > 搜索 "Cinemachine"
3. 安装 `Cinemachine`（版本 3.0+）

---

## 核心概念

### 1. **虚拟相机**
- 定义相机行为（位置、旋转、镜头）
- 可以存在多个虚拟相机；一次只有一个处于"活动"状态

### 2. **Cinemachine Brain**
- 主 Camera 上的组件
- 在虚拟相机之间混合
- 将虚拟相机设置应用到 Unity Camera

### 3. **优先级**
- 虚拟相机具有优先级值
- 优先级最高的相机处于活动状态
- 优先级变化时平滑混合

---

## 基本设置

### 1. 将 Cinemachine Brain 添加到主相机

```csharp
// 创建第一个虚拟相机时自动添加
// 或手动：Add Component > Cinemachine Brain
```

### 2. 创建虚拟相机

`GameObject > Cinemachine > Cinemachine Camera`

这将创建一个具有默认设置的 **CinemachineCamera** GameObject。

---

## 虚拟相机组件

### CinemachineCamera（Unity 6 / Cinemachine 3.0+）

```csharp
using Unity.Cinemachine;

public class CameraController : MonoBehaviour {
    public CinemachineCamera virtualCamera;

    void Start() {
        // 设置优先级（越高越活动）
        virtualCamera.Priority = 10;

        // 设置跟随目标
        virtualCamera.Follow = playerTransform;

        // 设置看向目标
        virtualCamera.LookAt = playerTransform;
    }
}
```

---

## 跟随模式（Body 组件）

### 第三人称跟随（轨道跟随）

```csharp
// 在 Inspector 中：
// CinemachineCamera > Body > 3rd Person Follow

// 配置：
// - Shoulder Offset：(0.5, 0, 0) 用于越肩
// - Camera Distance：5.0
// - Vertical Damping：0.5（平滑上下）
```

### Framing Transposer（平滑跟随）

```csharp
// CinemachineCamera > Body > Position Composer

// 配置：
// - Screen Position：Center (0.5, 0.5)
// - Dead Zone：目标在区域内时不移动相机
// - Damping：平滑跟随
```

### Hard Lock（精确跟随）

```csharp
// CinemachineCamera > Body > Hard Lock to Target
// 相机精确匹配目标位置（无偏移或阻尼）
```

---

## 瞄准模式（Aim 组件）

### Composer（取景目标）

```csharp
// CinemachineCamera > Aim > Composer

// 配置：
// - Tracked Object Offset：瞄准目标的头部而非脚部
// - Screen Position：目标在屏幕上的位置
// - Dead Zone：目标在区域内时不旋转
```

### 看向目标

```csharp
// CinemachineCamera > Aim > Rotate With Follow Target
// 相机旋转匹配目标旋转（例如，第一人称）
```

---

## 相机间混合

### 基于优先级的混合

```csharp
public CinemachineCamera normalCamera; // 优先级：10
public CinemachineCamera aimCamera;    // 优先级：5

void StartAiming() {
    // 将瞄准相机设置为更高优先级
    aimCamera.Priority = 15; // 现在活动
    // Brain 自动从 normalCamera 混合到 aimCamera
}

void StopAiming() {
    aimCamera.Priority = 5; // 恢复正常
}
```

### 自定义混合时间

```csharp
// 创建自定义混合资源：
// Assets > Create > Cinemachine > Cinemachine Blender Settings

// 在 Cinemachine Brain 中：
// - Custom Blends = 你的资源
// - 配置每对相机之间的混合时间
```

---

## 相机震动

### Impulse Source（触发震动）

```csharp
using Unity.Cinemachine;

public class ExplosionShake : MonoBehaviour {
    public CinemachineImpulseSource impulseSource;

    void Explode() {
        // 触发相机震动
        impulseSource.GenerateImpulse();
    }
}
```

### Impulse Listener（接收震动）

```csharp
// 添加到 CinemachineCamera：
// Add Component > CinemachineImpulseListener

// Impulse listener 自动接收附近 Impulse Source 的震动
```

---

## Freelook 相机（带鼠标视角的第三人称）

### Cinemachine Free Look

```csharp
// GameObject > Cinemachine > Cinemachine Free Look

// 创建 3 个 rig（Top、Middle、Bottom），根据垂直输入混合
// 配置：
// - Orbit Radius：与目标的距离
// - Height Offset：每个 rig 的相机高度
// - X/Y 轴：鼠标或摇杆输入
```

---

## State-Driven 相机（基于 Animator）

### Cinemachine State-Driven Camera

```csharp
// GameObject > Cinemachine > Cinemachine State-Driven Camera

// 配置：
// - Animated Target：带 Animator 的角色
// - Layer：要跟踪的 Animator 层
// - State：为每个动画状态分配相机（Idle、Run、Jump 等）

// 相机根据动画状态自动切换
```

---

## Dolly 轨道（过场动画）

### Cinemachine Dolly Track

```csharp
// 1. 创建 Spline：GameObject > Cinemachine > Cinemachine Spline

// 2. 创建 Dolly 相机：
//    GameObject > Cinemachine > Cinemachine Camera
//    Body > Spline Dolly
//    分配 Spline

// 3. 在 spline 上动画 dolly 位置（Timeline 或脚本）
```

---

## 常见模式

### 第三人称跟随相机

```csharp
// CinemachineCamera
// - Follow：Player Transform
// - Body：3rd Person Follow（shoulder offset，距离：5）
// - Aim：Composer（将玩家居中取景）
```

---

### 瞄准相机（放大）

```csharp
// 普通相机（优先级 10）：
//   - 距离：5.0

// 瞄准相机（优先级 5）：
//   - 距离：2.0
//   - FOV：更窄

// 脚本：
void StartAiming() {
    aimCamera.Priority = 15; // 混合到瞄准相机
}
```

---

### 过场动画相机序列

```csharp
// 使用 Timeline：
// 1. 创建 Timeline（Assets > Create > Timeline）
// 2. 添加 Cinemachine Track
// 3. 将虚拟相机添加为片段
// 4. Timeline 自动在相机之间混合
```

---

## 从 Cinemachine 2.x 迁移（Unity 2021）

### API 变更（Unity 6 / Cinemachine 3.0）

```csharp
// ❌ 旧版（Cinemachine 2.x）：
CinemachineVirtualCamera vcam;
vcam.m_Follow = target;

// ✅ 新版（Cinemachine 3.0+）：
CinemachineCamera vcam;
vcam.Follow = target; // 更清晰的 API
```

**重大变更：**
- `CinemachineVirtualCamera` → `CinemachineCamera`
- `m_Follow`、`m_LookAt` → `Follow`、`LookAt`（无 "m_" 前缀）
- 组件为清晰起见重命名
- 更好的性能

---

## 性能提示

- 限制活动虚拟相机（仅在需要时激活）
- 使用低优先级相机而不是销毁/创建
- 当远离玩家时禁用虚拟相机

---

## 调试

### Cinemachine Debug

```csharp
// Window > Analysis > Cinemachine Debugger
// 显示活动相机、混合信息、镜头质量
```

---

## 来源
- https://docs.unity3d.com/Packages/com.unity.cinemachine@3.0/manual/index.html
- https://learn.unity.com/tutorial/cinemachine
