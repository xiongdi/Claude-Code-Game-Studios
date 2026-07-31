# Unreal Engine 5.7 — 物理模块参考

**最后验证时间：** 2026-02-13
**知识差距：** UE 5.7 Chaos Physics 改进

---

## 概述

UE 5 使用 **Chaos Physics**（取代了 UE 4 中的 PhysX）：
- 更好的性能
- 支持破坏
- 载具物理改进

---

## 刚体物理

### 在静态网格体上启用物理

```cpp
UStaticMeshComponent* MeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
MeshComp->SetSimulatePhysics(true);
MeshComp->SetEnableGravity(true);
MeshComp->SetMassOverrideInKg(NAME_None, 50.0f); // 50 kg
```

### 施加力

```cpp
// 施加冲量（瞬时速度变化）
MeshComp->AddImpulse(FVector(0, 0, 1000), NAME_None, true);

// 施加力（持续）
MeshComp->AddForce(FVector(0, 0, 500));

// 施加扭矩（旋转）
MeshComp->AddTorqueInRadians(FVector(0, 0, 100));
```

---

## 碰撞

### 碰撞通道

```cpp
// Project Settings > Engine > Collision
// 定义自定义碰撞通道和响应

// 在 C++ 中设置碰撞
MeshComp->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
MeshComp->SetCollisionObjectType(ECollisionChannel::ECC_Pawn);
MeshComp->SetCollisionResponseToAllChannels(ECR_Block);
MeshComp->SetCollisionResponseToChannel(ECC_Camera, ECR_Ignore);
```

### 碰撞事件

```cpp
// 启用碰撞事件
MeshComp->SetNotifyRigidBodyCollision(true);

// 绑定到 OnComponentHit
MeshComp->OnComponentHit.AddDynamic(this, &AMyActor::OnHit);

UFUNCTION()
void AMyActor::OnHit(UPrimitiveComponent* HitComp, AActor* OtherActor,
    UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit) {
    UE_LOG(LogTemp, Warning, TEXT("Hit %s"), *OtherActor->GetName());
}
```

### 重叠事件

```cpp
// 启用重叠事件
MeshComp->SetGenerateOverlapEvents(true);

// 绑定到 OnComponentBeginOverlap
MeshComp->OnComponentBeginOverlap.AddDynamic(this, &AMyActor::OnOverlapBegin);

UFUNCTION()
void AMyActor::OnOverlapBegin(UPrimitiveComponent* OverlappedComp, AActor* OtherActor,
    UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult) {
    UE_LOG(LogTemp, Warning, TEXT("Overlapped %s"), *OtherActor->GetName());
}
```

---

## 射线投射（Line Trace）

### 单线射线

```cpp
FHitResult HitResult;
FVector Start = GetActorLocation();
FVector End = Start + GetActorForwardVector() * 1000.0f;

FCollisionQueryParams QueryParams;
QueryParams.AddIgnoredActor(this);

// 执行射线
bool bHit = GetWorld()->LineTraceSingleByChannel(
    HitResult,
    Start,
    End,
    ECC_Visibility,
    QueryParams
);

if (bHit) {
    UE_LOG(LogTemp, Warning, TEXT("Hit: %s"), *HitResult.GetActor()->GetName());
    DrawDebugLine(GetWorld(), Start, HitResult.Location, FColor::Red, false, 2.0f);
}
```

### 多线射线

```cpp
TArray<FHitResult> HitResults;
bool bHit = GetWorld()->LineTraceMultiByChannel(
    HitResults,
    Start,
    End,
    ECC_Visibility,
    QueryParams
);

for (const FHitResult& Hit : HitResults) {
    UE_LOG(LogTemp, Warning, TEXT("Hit: %s"), *Hit.GetActor()->GetName());
}
```

### Sweep（粗射线）

```cpp
FHitResult HitResult;
FCollisionShape Sphere = FCollisionShape::MakeSphere(50.0f);

bool bHit = GetWorld()->SweepSingleByChannel(
    HitResult,
    Start,
    End,
    FQuat::Identity,
    ECC_Visibility,
    Sphere,
    QueryParams
);
```

---

## 角色移动

### Character Movement Component

```cpp
// 内置于 ACharacter 类
UCharacterMovementComponent* MoveComp = GetCharacterMovement();

// 配置移动
MoveComp->MaxWalkSpeed = 600.0f;
MoveComp->JumpZVelocity = 600.0f;
MoveComp->AirControl = 0.2f;
MoveComp->GravityScale = 1.0f;
MoveComp->bOrientRotationToMovement = true;
```

### 添加移动输入

```cpp
// 在 Character 类中
void AMyCharacter::MoveForward(float Value) {
    if (Value != 0.0f) {
        AddMovementInput(GetActorForwardVector(), Value);
    }
}

void AMyCharacter::MoveRight(float Value) {
    if (Value != 0.0f) {
        AddMovementInput(GetActorRightVector(), Value);
    }
}
```

---

## 物理材质

### 创建物理材质

1. Content Browser > Right Click > Physics > Physical Material
2. 配置属性：
   - 摩擦力：0.0 - 1.0
   - 弹性（bounciness）：0.0 - 1.0

### 分配物理材质

```cpp
// 在静态网格体编辑器中：Physics > Phys Material Override
// 或在 C++ 中：
MeshComp->SetPhysMaterialOverride(PhysicalMaterial);
```

---

## 约束（物理关节）

### Physics Constraint Component

```cpp
UPhysicsConstraintComponent* Constraint = CreateDefaultSubobject<UPhysicsConstraintComponent>(TEXT("Constraint"));
Constraint->SetConstrainedComponents(ComponentA, NAME_None, ComponentB, NAME_None);

// 配置约束
Constraint->SetLinearXLimit(ELinearConstraintMotion::LCM_Limited, 100.0f);
Constraint->SetLinearYLimit(ELinearConstraintMotion::LCM_Locked, 0.0f);
Constraint->SetLinearZLimit(ELinearConstraintMotion::LCM_Free, 0.0f);

Constraint->SetAngularSwing1Limit(EAngularConstraintMotion::ACM_Limited, 45.0f);
```

---

## 破坏（Chaos Destruction）

### 启用 Chaos Destruction

```cpp
// 插件：启用 "Chaos" 插件
// 为可破坏对象创建 Geometry Collection 资源
```

### 破坏 Geometry Collection

```cpp
// 在 Chaos 编辑器中破碎网格体
// 在游戏中，施加伤害：
UGeometryCollectionComponent* GeoComp = /* 获取组件 */;
GeoComp->ApplyPhysicsField(/* 场参数 */);
```

---

## 性能技巧

### 物理优化

```cpp
// 简化碰撞形状（使用简单基本体）
MeshComp->SetCollisionEnabled(ECollisionEnabled::NoCollision); // 不需要时禁用

// 对骨骼网格体使用 Physics Asset（简化碰撞）
// 不要为远处物体模拟物理

// 减少物理子步：
// Project Settings > Engine > Physics > Max Substep Delta Time
```

---

## 调试

### 物理调试图可视化

```cpp
// 控制台命令：
// show collision - 显示碰撞形状
// p.Chaos.DebugDraw.Enabled 1 - 显示 Chaos 调试
// pxvis collision - 可视化碰撞

// 绘制调试形状：
DrawDebugSphere(GetWorld(), Location, Radius, 12, FColor::Green, false, 2.0f);
DrawDebugBox(GetWorld(), Location, Extent, FColor::Red, false, 2.0f);
```

---

## 来源
- https://docs.unrealengine.com/5.7/en-US/physics-in-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/chaos-physics-overview-in-unreal-engine/
