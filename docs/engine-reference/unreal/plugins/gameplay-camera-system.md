# Unreal Engine 5.7 — Gameplay Camera System

**最后验证时间：** 2026-02-13
**状态：** ⚠️ 实验性（UE 5.5 中引入）
**插件：** `GameplayCameras`（内置，在 Plugins 中启用）

---

## 概述

**Gameplay Camera System** 是 UE 5.5 中引入的模块化摄像机管理框架。
它用灵活的基于节点的系统取代了传统摄像机设置，处理摄像机模式、混合和上下文感知摄像机行为。

**将 Gameplay Cameras 用于：**
- 动态摄像机行为（第三人称、瞄准、载具、电影级）
- 上下文感知摄像机切换（战斗、探索、对话）
- 模式间平滑摄像机混合
- 程序化摄像机运动（摄像机抖动、延迟、偏移）

**⚠️ 警告：** 此插件在 UE 5.5-5.7 中为实验性。预计未来版本中会有 API 变更。

---

## 核心概念

### 1. **Camera Rig**
- 定义摄像机配置（位置、旋转、FOV 等）
- 模块化节点图（类似材质编辑器）

### 2. **Camera Director**
- 管理哪个摄像机 rig 处于活动状态
- 处理摄像机 rig 之间的混合

### 3. **Camera Nodes**
- 摄像机行为的构建块：
  - **位置节点**：Orbit、Follow、Fixed Position
  - **旋转节点**：Look At、Match Actor Rotation
  - **修饰器**：Camera Shake、Lag、Offset

---

## 设置

### 1. 启用插件

`Edit > Plugins > Gameplay Cameras > Enabled > Restart`

### 2. 添加摄像机组件

```cpp
#include "GameplayCameras/Public/GameplayCameraComponent.h"

UCLASS()
class AMyCharacter : public ACharacter {
    GENERATED_BODY()

public:
    AMyCharacter() {
        // 创建摄像机组件
        CameraComponent = CreateDefaultSubobject<UGameplayCameraComponent>(TEXT("GameplayCamera"));
        CameraComponent->SetupAttachment(RootComponent);
    }

protected:
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Camera")
    TObjectPtr<UGameplayCameraComponent> CameraComponent;
};
```

---

## 创建 Camera Rig

### 1. 创建 Camera Rig 资源

1. Content Browser > Gameplay > Gameplay Camera Rig
2. 打开 Camera Rig Editor（基于节点的图）

### 2. 构建 Camera Rig（示例：第三人称）

**节点设置：**
```
Actor Position（角色）
  ↓
Orbit Node（围绕角色轨道）
  ↓
Offset Node（肩膀偏移）
  ↓
Look At Node（看向角色）
  ↓
Camera Output
```

---

## 摄像机节点

### 位置节点

#### Orbit 节点（第三人称）
- 围绕目标 actor 轨道运动
- 配置：
  - **Orbit Distance**：与目标的距离（例如 300 单位）
  - **Pitch Range**：最小/最大俯仰角
  - **Yaw Range**：最小/最大偏航角

#### Follow 节点（平滑跟随）
- 带延迟地跟随目标
- 配置：
  - **Lag Speed**：摄像机追上目标的速度
  - **Offset**：与目标的固定偏移

#### Fixed Position 节点
- 世界空间中的静态摄像机位置

---

### 旋转节点

#### Look At 节点
- 将摄像机对准目标
- 配置：
  - **Target**：要看向的 actor 或组件
  - **Offset**：看向偏移（例如瞄准头部而非脚部）

#### Match Actor Rotation
- 匹配目标 actor 的旋转
- 适用于第一人称或载具摄像机

---

### 修饰器节点

#### Camera Shake
- 添加程序化抖动（例如脚步声、爆炸）
- 配置：
  - **Shake Pattern**：Perlin 噪声、正弦波、自定义
  - **Amplitude**：抖动强度

#### Camera Lag
- 摄像机移动的平滑阻尼
- 配置：
  - **Lag Speed**：阻尼因子（0 = 即时，越高 = 更多延迟）

#### Offset 节点
- 计算位置的静态偏移
- 适用于肩膀摄像机偏移

---

## Camera Director（在 Rig 间切换）

### 分配 Camera Rig

```cpp
#include "GameplayCameras/Public/GameplayCameraComponent.h"

void AMyCharacter::SetCameraMode(UGameplayCameraRig* NewRig) {
    if (CameraComponent) {
        CameraComponent->SetCameraRig(NewRig);
    }
}
```

### 在 Camera Rig 间混合

```cpp
// 在 0.5 秒内混合到瞄准摄像机
CameraComponent->BlendToCameraRig(AimingCameraRig, 0.5f);
```

---

## 示例：第三人称 + 瞄准

### 1. 创建两个 Camera Rig

**第三人称 Rig：**
```
Actor Position → Orbit（距离：300）→ Look At → Output
```

**瞄准 Rig：**
```
Actor Position → Orbit（距离：150）→ Offset（肩膀）→ Look At → Output
```

### 2. 瞄准时切换

```cpp
UPROPERTY(EditAnywhere, Category = "Camera")
TObjectPtr<UGameplayCameraRig> ThirdPersonRig;

UPROPERTY(EditAnywhere, Category = "Camera")
TObjectPtr<UGameplayCameraRig> AimingRig;

void StartAiming() {
    CameraComponent->BlendToCameraRig(AimingRig, 0.3f); // 在 0.3 秒内混合
}

void StopAiming() {
    CameraComponent->BlendToCameraRig(ThirdPersonRig, 0.3f);
}
```

---

## 常见模式

### 越肩摄像机

```
Actor Position
  ↓
Orbit Node（距离：250，偏航偏移：30°）
  ↓
Offset Node（X：0，Y：50，Z：50）// 肩膀偏移
  ↓
Look At Node（目标：角色头部）
  ↓
Output
```

---

### 载具摄像机

```
Vehicle Position
  ↓
Follow Node（延迟：0.2）
  ↓
Offset Node（载具后方：X：-400，Z：150）
  ↓
Look At Node（目标：载具）
  ↓
Output
```

---

### 第一人称摄像机

```
Character Head Socket
  ↓
Match Actor Rotation
  ↓
Output
```

---

## 摄像机抖动

### 触发摄像机抖动

```cpp
#include "GameplayCameras/Public/GameplayCameraShake.h"

void TriggerExplosionShake() {
    if (APlayerController* PC = GetWorld()->GetFirstPlayerController()) {
        if (UGameplayCameraComponent* CameraComp = PC->FindComponentByClass<UGameplayCameraComponent>()) {
            CameraComp->PlayCameraShake(ExplosionShakeClass, 1.0f);
        }
    }
}
```

---

## 性能技巧

- 限制摄像机抖动频率（不要每帧触发）
- 谨慎使用摄像机延迟（高延迟值开销大）
- 缓存摄像机 rig 引用（不要每帧搜索）

---

## 调试

### 摄像机调试图可视化

```cpp
// 控制台命令：
// GameplayCameras.Debug 1 - 显示活动摄像机 rig 信息
// showdebug camera - 显示摄像机调试信息
```

---

## 从传统摄像机迁移

### 旧版 Spring Arm + Camera Component

```cpp
// ❌ 旧：Spring Arm Component
USpringArmComponent* SpringArm;
UCameraComponent* Camera;

// ✅ 新：Gameplay Camera Component
UGameplayCameraComponent* CameraComponent;
// 在 Camera Rig 资源中构建 orbit + look-at rig
```

---

## 限制（实验性状态）

- **API 不稳定性**：预计 UE 5.8+ 中会有破坏性变更
- **文档有限**：官方文档仍在完善中
- **Blueprint 支持**：主要面向 C++（Blueprint 支持在改进中）
- **生产风险**：发布前彻底测试

---

## 来源
- https://docs.unrealengine.com/5.7/en-US/gameplay-cameras-in-unreal-engine/
- UE 5.5+ 发布说明
- **注意：** 此系统为实验性。始终检查最新官方文档以了解 API 变更。
