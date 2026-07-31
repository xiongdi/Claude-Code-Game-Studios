# Unity 6.3 — 物理模块参考

**最后验证时间：** 2026-02-13
**知识缺口：** Unity 6 物理改进、求解器变化

---

## 概述

Unity 6.3 使用 **PhysX 5.1**（从 2022 LTS 的 PhysX 4.x 改进）：
- 更好的求解器稳定性
- 改进的性能
- 增强的碰撞检测

---

## 与 2022 LTS 相比的关键变化

### 默认求解器迭代次数增加
Unity 6 增加了默认求解器迭代次数以获得更好的稳定性：

```csharp
// 默认值从 6 次迭代更改为 8 次
Physics.defaultSolverIterations = 8; // 检查是否依赖旧行为
```

### 增强的碰撞检测

```csharp
// ✅ Unity 6：改进的连续碰撞检测（CCD）
rigidbody.collisionDetectionMode = CollisionDetectionMode.ContinuousDynamic;
// 更好地处理快速移动的对象
```

---

## 核心物理组件

### Rigidbody

```csharp
// ✅ 最佳实践：使用 AddForce，而不是直接写入速度
Rigidbody rb = GetComponent<Rigidbody>();
rb.AddForce(Vector3.forward * 10f, ForceMode.Impulse);

// ❌ 避免：直接速度赋值（可能导致不稳定）
rb.velocity = new Vector3(0, 10, 0); // 仅在必要时使用
```

### Collider

```csharp
// 原始碰撞体：Box、Sphere、Capsule（最便宜）
// Mesh 碰撞体：开销大，仅用于静态几何体

// ✅ 组合碰撞体（多个原始碰撞体）> 单个 mesh 碰撞体
```

---

## 射线检测

### 高效射线检测（避免分配）

```csharp
// ✅ 非分配射线检测
if (Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance)) {
    Debug.Log($"Hit: {hit.collider.name}");
}

// ✅ 多次命中（非分配）
RaycastHit[] results = new RaycastHit[10];
int hitCount = Physics.RaycastNonAlloc(origin, direction, results, maxDistance);
for (int i = 0; i < hitCount; i++) {
    Debug.Log($"Hit {i}: {results[i].collider.name}");
}

// ❌ 避免：RaycastAll（每次调用都分配数组）
RaycastHit[] hits = Physics.RaycastAll(origin, direction); // GC 分配！
```

### LayerMask 用于选择性射线检测

```csharp
// ✅ 使用 LayerMask 过滤碰撞
int layerMask = 1 << LayerMask.NameToLayer("Enemy");
Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance, layerMask);
```

---

## 物理查询

### OverlapSphere（检查附近对象）

```csharp
// ✅ 非分配版本
Collider[] results = new Collider[10];
int count = Physics.OverlapSphereNonAlloc(center, radius, results);
for (int i = 0; i < count; i++) {
    // 处理 results[i]
}
```

### SphereCast（粗射线检测）

```csharp
// 对角色控制器有用
if (Physics.SphereCast(origin, radius, direction, out RaycastHit hit, maxDistance)) {
    // 用球形射线击中了某物
}
```

---

## 碰撞事件

### OnCollisionEnter / Stay / Exit

```csharp
void OnCollisionEnter(Collision collision) {
    // 碰撞开始时触发
    Debug.Log($"Collided with {collision.gameObject.name}");

    // 访问接触点
    foreach (ContactPoint contact in collision.contacts) {
        Debug.DrawRay(contact.point, contact.normal, Color.red, 2f);
    }
}
```

### OnTriggerEnter / Stay / Exit

```csharp
void OnTriggerEnter(Collider other) {
    // 触发碰撞体（Is Trigger = true）
    if (other.CompareTag("Pickup")) {
        Destroy(other.gameObject);
    }
}
```

---

## 角色控制器

### CharacterController 组件

```csharp
CharacterController controller = GetComponent<CharacterController>();

// ✅ 带碰撞检测的移动
Vector3 move = transform.forward * speed * Time.deltaTime;
controller.Move(move);

// 手动应用重力
if (!controller.isGrounded) {
    velocity.y += Physics.gravity.y * Time.deltaTime;
}
controller.Move(velocity * Time.deltaTime);
```

---

## 物理材质

### 摩擦力与弹性

```csharp
// 创建：Assets > Create > Physic Material
// 分配到碰撞体：Collider > Material

// PhysicMaterial 设置：
// - Dynamic Friction：0.6（滑动摩擦）
// - Static Friction：0.6（起始摩擦）
// - Bounciness：0.0 - 1.0
// - Friction Combine：Average、Minimum、Maximum、Multiply
// - Bounce Combine：Average、Minimum、Maximum、Multiply
```

---

## 关节

### Fixed Joint（连接两个刚体）

```csharp
FixedJoint joint = gameObject.AddComponent<FixedJoint>();
joint.connectedBody = otherRigidbody;
```

### Hinge Joint（门、轮子）

```csharp
HingeJoint hinge = gameObject.AddComponent<HingeJoint>();
hinge.axis = Vector3.up; // 旋转轴
hinge.useLimits = true;
hinge.limits = new JointLimits { min = -90, max = 90 };
```

---

## 性能优化

### 物理层碰撞矩阵
`Edit > Project Settings > Physics > Layer Collision Matrix`
- 禁用层之间不必要的碰撞检查
- 巨大的性能提升

### 固定时间步长
`Edit > Project Settings > Time > Fixed Timestep`
- 默认：0.02（50 FPS 物理）
- 越低 = 越准确，CPU 成本越高
- 如果可能，匹配游戏的目标帧率

### 简化碰撞几何体
- 使用原始碰撞体（box、sphere、capsule）而非 mesh 碰撞体
- 在构建时烘焙 mesh 碰撞体，而非运行时

---

## 常见模式

### 地面检测（角色控制器）

```csharp
bool IsGrounded() {
    float rayLength = 0.1f;
    return Physics.Raycast(transform.position, Vector3.down, rayLength);
}
```

### 施加爆炸力

```csharp
void ApplyExplosion(Vector3 explosionPos, float radius, float force) {
    Collider[] colliders = Physics.OverlapSphere(explosionPos, radius);
    foreach (Collider hit in colliders) {
        Rigidbody rb = hit.GetComponent<Rigidbody>();
        if (rb != null) {
            rb.AddExplosionForce(force, explosionPos, radius);
        }
    }
}
```

---

## 调试

### 物理调试器（Unity 6+）
- `Window > Analysis > Physics Debugger`
- 可视化碰撞体、接触点、查询

### Gizmos

```csharp
void OnDrawGizmos() {
    Gizmos.color = Color.red;
    Gizmos.DrawWireSphere(transform.position, detectionRadius);
}
```

---

## 来源
- https://docs.unity3d.com/6000.0/Documentation/Manual/PhysicsOverview.html
- https://docs.unity3d.com/ScriptReference/Physics.html
