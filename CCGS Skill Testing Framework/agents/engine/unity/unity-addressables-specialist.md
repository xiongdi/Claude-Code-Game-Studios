# Agent Test Spec: unity-addressables-specialist

## Agent Summary
Domain: Addressable Asset System — 资源组、异步加载/卸载、handle 生命周期管理、内存预算、内容目录（content catalog）和远程内容分发。
Does NOT own: 渲染系统（engine-programmer）、使用已加载资源的游戏逻辑（gameplay-programmer）。
Model tier: Sonnet (default)。
No gate IDs assigned。

---

## Static Assertions (Structural)

- [ ] `description:` 字段存在且为领域特定（引用 Addressables / 资源加载 / 内容目录 / 远程分发）
- [ ] `allowed-tools:` 列表包含 Read, Write, Edit, Bash, Glob, Grep
- [ ] Model tier 为 Sonnet（specialist 默认值）
- [ ] Agent 定义不声称对渲染系统或使用已加载资源的游戏逻辑拥有权限

---

## Test Cases

### Case 1: 领域内请求 — 适当的输出
**Input:** "异步加载角色角色纹理，并在角色被销毁时释放它。"
**Expected behavior:**
- 生成 `Addressables.LoadAssetAsync<Texture2D>()` 调用模式
- 将返回的 `AsyncOperationHandle<Texture2D>` 存储在请求对象中
- 在角色销毁时（`OnDestroy()`），使用存储的 handle 调用 `Addressables.Release(handle)`
- 不使用 `Resources.Load()` 作为加载机制
- 注意使用 null 或未初始化的 handle 释放会导致错误 — 包含有效性检查
- 注意释放 handle 与释放 asset 之间的区别（释放 handle 是正确的做法）

### Case 2: 领域外重定向
**Input:** "实现将加载的纹理应用到角色网格的渲染系统。"
**Expected behavior:**
- 不生成渲染或网格材质分配代码
- 明确声明渲染系统实现属于 `engine-programmer`
- 将请求重定向到 `engine-programmer`
- 可以描述它将提供的资源类型和 API 表面（例如，handle 完成后的 `Texture2D` 引用）作为交接规格

### Case 3: 内存泄漏 — 未释放的 handle
**Input:** "每次关卡加载后内存使用量持续攀升。我们使用 Addressables 加载关卡资源。"
**Expected behavior:**
- 诊断可能原因：`AsyncOperationHandle` 对象在使用后未被释放
- 识别 handle 泄漏模式：将资源加载到局部变量中，丢失引用，从未调用 `Addressables.Release()`
- 生成审计方法：搜索所有 `LoadAssetAsync` / `LoadSceneAsync` 调用并验证匹配的 `Release()` 调用
- 提供使用已跟踪 handle 列表（`List<AsyncOperationHandle>`）和 `ReleaseAll()` 清理方法的修正模式
- 在没有证据的情况下不假设泄漏来自其他地方

### Case 4: 远程内容分发 — 目录版本控制
**Input:** "我们需要支持无需完整重新安装应用的内容更新。"
**Expected behavior:**
- 生成远程目录更新模式：
  - 启动时调用 `Addressables.CheckForCatalogUpdates()`
  - 对检测到的更新调用 `Addressables.UpdateCatalogs()`
  - 调用 `Addressables.DownloadDependenciesAsync()` 预热更新后的内容
- 注意用于变更检测的目录哈希检查
- 处理边缘情况：如果玩家开始会话后，目录在会话中途更新 — 定义行为（在当前会话使用旧目录完成，下次启动时重新加载）
- 不设计服务端 CDN 基础设施（交由 devops-engineer 处理）

### Case 5: 上下文传递 — 平台内存约束
**Input:** 平台上下文：Nintendo Switch 目标平台，4GB RAM，实际资源内存上限 512MB。请求："为大型开放世界关卡设计 Addressables 加载策略。"
**Expected behavior:**
- 引用提供上下文中的 512MB 内存上限
- 设计流式加载策略：
  - 将世界划分为基于玩家距离加载/卸载的 addressable 区域
  - 定义每个活动区域的内存预算（例如 128MB，最多 4 个活动区域）
  - 指定异步预加载触发距离和卸载距离（迟滞）
  - 注意 Switch 特定约束：SD 卡加载时间较慢，建议预热相邻区域
- 不生成超出所述 512MB 上限且不标记的加载策略

---

## Protocol Compliance

- [ ] 保持在声明领域内（Addressables 加载、handle 生命周期、内存、目录、远程分发）
- [ ] 将渲染和游戏逻辑资源使用代码重定向到 engine-programmer 和 gameplay-programmer
- [ ] 返回结构化输出（加载模式、handle 生命周期代码、流式区域设计）
- [ ] 始终将 `LoadAssetAsync` 与对应的 `Release()` 配对 — 将 handle 泄漏标记为内存 bug
- [ ] 根据提供的内存上限设计加载策略
- [ ] 不设计 CDN/服务器基础设施 — 服务端交由 devops-engineer 处理

---

## Coverage Notes
- Handle 生命周期（Case 1）必须包含一个测试，验证释放后内存被回收
- Handle 泄漏诊断（Case 3）应生成适用于 bug 工单的发现报告
- 平台内存用例（Case 5）验证 agent 应用来自上下文的硬约束，而非默认假设
