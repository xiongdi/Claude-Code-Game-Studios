# Agent Test Spec: unity-ui-specialist

## Agent Summary
Domain: Unity UI Toolkit (UXML/USS)、UGUI (Canvas)、数据绑定、运行时 UI 性能和 UI 输入事件处理。
Does NOT own: UX 流程设计（ux-designer）、视觉美术风格（art-director）。
Model tier: Sonnet (default)。
No gate IDs assigned。

---

## Static Assertions (Structural)

- [ ] `description:` 字段存在且为领域特定（引用 UI Toolkit / UGUI / Canvas / 数据绑定）
- [ ] `allowed-tools:` 列表包含 Read, Write, Edit, Bash, Glob, Grep
- [ ] Model tier 为 Sonnet（specialist 默认值）
- [ ] Agent 定义不声称对 UX 流程设计或视觉美术方向拥有权限

---

## Test Cases

### Case 1: 领域内请求 — 适当的输出
**Input:** "使用 Unity UI Toolkit 实现背包 UI 界面。"
**Expected behavior:**
- 生成定义背包面板结构的 UXML 文档（ListView、物品模板、详情面板）
- 生成背包布局和物品状态（默认、悬停、选中）的 USS 样式
- 提供通过 `INotifyValueChanged` 或 `IBindable` 将背包数据模型绑定到 UI 的 C# 代码
- 对可滚动物品列表使用带 `makeItem` / `bindItem` 回调的 `ListView`
- 不生成 UX 流程设计 — 根据提供的规格实现

### Case 2: 领域外重定向
**Input:** "设计背包的 UX 流程 — 玩家装备与丢弃物品时会发生什么。"
**Expected behavior:**
- 不生成 UX 流程设计
- 明确声明交互流程设计属于 `ux-designer`
- 将请求重定向到 `ux-designer`
- 注意它将实现 ux-designer 指定的任何流程

### Case 3: 动态列表的 UI Toolkit 数据绑定
**Input:** "背包列表需要在玩家背包中添加或移除物品时实时更新。"
**Expected behavior:**
- 生成带绑定的 `ObservableList<T>` 或事件驱动刷新方法的 `ListView` 模式
- 在底层集合变更事件上使用 `ListView.Rebuild()` 或 `ListView.RefreshItems()`
- 注意大型列表的性能考虑（通过 `makeItem`/`bindItem` 模式实现虚拟化）
- 不使用 `QuerySelector` 循环作为列表刷新策略更新单个元素 — 将其标记为性能反模式

### Case 4: Canvas 性能 — 过度绘制
**Input:** "主菜单 Canvas 导致 GPU 过度绘制警告；存在许多重叠面板。"
**Expected behavior:**
- 识别过度绘制原因：多个堆叠 Canvas、非活动时未剔除的全屏覆盖面板
- 建议：
  - 为世界空间、屏幕空间覆盖层和屏幕空间相机层分离 Canvas
  - 禁用/停用面板而不是将 alpha 设置为 0（不可见的 alpha-0 面板仍然绘制）
  - 使用 Canvas Group + alpha 实现淡入淡出效果，而非单个 Image alpha
- 如果项目处于迁移阶段，注意 UI Toolkit 替代方案

### Case 5: 上下文传递 — Unity 版本
**Input:** 项目上下文：Unity 2022.3 LTS。请求："实现带数据绑定的设置面板。"
**Expected behavior:**
- 使用 Unity 2022.3 LTS 版本的运行时绑定系统的 UI Toolkit
- 注意 Unity 2022.3 引入了运行时数据绑定（相对于早期版本的编辑器专用绑定）
- 如果 Unity 6 增强绑定 API 功能在 2022.3 中不可用，则不使用
- 生成与所述 Unity 版本兼容的代码，附带版本特定 API 说明

---

## Protocol Compliance

- [ ] 保持在声明领域内（UI Toolkit、UGUI、数据绑定、UI 性能）
- [ ] 将 UX 流程设计重定向到 ux-designer
- [ ] 返回结构化输出（UXML、USS、C# 绑定代码）
- [ ] 对项目的 Unity 版本使用正确的 Unity UI 框架版本
- [ ] 将 Canvas 过度绘制标记为性能反模式并提供具体修复方案
- [ ] 不使用 alpha-0 作为隐藏/显示模式 — 使用 SetActive() 或 VisualElement.style.display

---

## Coverage Notes
- 背包 UI（Case 1）应在 `production/qa/evidence/` 中有手动演练文档
- 动态列表绑定（Case 3）应有集成测试或自动化交互测试
- Canvas 过度绘制（Case 4）验证 agent 了解正确的 Unity UI 性能模式
