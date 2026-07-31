# Agent Test Spec: unity-dots-specialist

## Agent Summary
Domain: ECS 架构（IComponentData、ISystem、SystemAPI）、Jobs 系统（IJob、IJobEntity、Burst）、Burst 编译器约束、DOTS 游戏系统和 hybrid renderer。
Does NOT own: MonoBehaviour 游戏逻辑代码（gameplay-programmer）、UI 实现（unity-ui-specialist）。
Model tier: Sonnet (default)。
No gate IDs assigned。

---

## Static Assertions (Structural)

- [ ] `description:` 字段存在且为领域特定（引用 ECS / Jobs / Burst / IComponentData）
- [ ] `allowed-tools:` 列表包含 Read, Write, Edit, Bash, Glob, Grep
- [ ] Model tier 为 Sonnet（specialist 默认值）
- [ ] Agent 定义不声称对 MonoBehaviour 游戏逻辑或 UI 系统拥有权限

---

## Test Cases

### Case 1: 领域内请求 — 适当的输出
**Input:** "将玩家移动系统转换为 ECS。"
**Expected behavior:**
- 生成：
  - 包含 velocity、speed 和 input vector 字段的 `PlayerMovementData : IComponentData` 结构体
  - 使用 `SystemAPI.Query<>` 或 `IJobEntity` 的带 `OnUpdate()` 的 `PlayerMovementSystem : ISystem`
  - 通过 `IBaker` 从编写用 MonoBehaviour 烘焙玩家初始状态
- 对位置更新使用 `RefRW<LocalTransform>`（而非已弃用的 `Translation`）
- 将 job 标记为 `[BurstCompile]` 并注明为 Burst 兼容性必须是非托管的内容
- 不修改输入轮询系统 — 从现有的 `PlayerInputData` 组件读取

### Case 2: MonoBehaviour 回推
**Input:** "对玩家移动就用 MonoBehaviour 吧 — 更简单。"
**Expected behavior:**
- 承认简单性论点
- 解释 DOTS 权衡：前期设置更多，但 ECS/Burst 方法提供项目 ADR 或需求中记录的性能特征
- 如果项目已承诺使用 DOTS，则不实现 MonoBehaviour 版本
- 如果不存在承诺，将架构决策标记给 `lead-programmer` / `technical-director` 解决
- 不单方面做出 MonoBehaviour 与 DOTS 的决策

### Case 3: Burst 不兼容的托管内存
**Input:** "这个 Burst job 访问 `List<EnemyData>` 来寻找最近的敌人。"
**Expected behavior:**
- 将 `List<T>` 标记为与 Burst 编译不兼容的托管类型
- 不批准带托管内存访问的 Burst job
- 提供正确的替代方案：根据用例使用 `NativeArray<EnemyData>`、`NativeList<EnemyData>` 或 `NativeHashMap<>`
- 注意 `NativeArray` 必须显式释放或通过 `[DeallocateOnJobCompletion]` 释放
- 使用非托管原生容器生成修正后的 job

### Case 4: 混合访问 — DOTS 系统需要 MonoBehaviour 数据
**Input:** "DOTS 移动系统需要读取由 MonoBehaviour CameraController 管理的相机变换。"
**Expected behavior:**
- 将此识别为混合访问场景
- 提供正确的混合模式：将相机变换存储在单例 `IComponentData` 中（每帧从 MonoBehaviour 端通过 `EntityManager.SetComponentData` 更新）
- 或者建议 `CompanionComponent` / 托管组件方法
- 不在 Burst job 内部访问 MonoBehaviour — 将其标记为不安全
- 在 MonoBehaviour 端（写入 ECS）和 DOTS 系统端（从 ECS 读取）都提供桥接代码

### Case 5: 上下文传递 — 性能目标
**Input:** 来自上下文的技术偏好：60fps 目标，每帧最大 2ms CPU 脚本预算。请求："为 10,000 个敌人实体设计 ECS chunk 布局。"
**Expected behavior:**
- 在设计理由中明确引用 2ms CPU 预算
- 为缓存效率设计 `IComponentData` chunk 布局：
  - 将频繁一起查询的组件分组到同一 archetype 中
  - 将很少使用的数据分离到独立组件中，以保持热数据紧凑
  - 根据 2ms 预算估算实体迭代时间
- 提供内存布局分析（每实体字节数、16KB chunk 大小时每 chunk 实体数）
- 不设计明显超出所述 2ms 预算且不标记的布局

---

## Protocol Compliance

- [ ] 保持在声明领域内（ECS、Jobs、Burst、DOTS 游戏系统）
- [ ] 将纯 MonoBehaviour 游戏逻辑重定向到 gameplay-programmer
- [ ] 返回结构化输出（IComponentData 结构体、ISystem 实现、IBaker 编写类）
- [ ] 将 Burst job 中的托管内存访问标记为编译错误并提供非托管替代方案
- [ ] 当 DOTS 系统需要与 MonoBehaviour 系统交互时提供混合访问模式
- [ ] 根据提供的性能预算设计 chunk 布局

---

## Coverage Notes
- ECS 转换（Case 1）必须包含使用 ECS 测试框架（`World`、`EntityManager`）的单元测试
- Burst 不兼容（Case 3）是安全关键的 — agent 必须在代码编写前捕获此问题
- Chunk 布局（Case 5）验证 agent 将定量性能推理应用于架构决策
