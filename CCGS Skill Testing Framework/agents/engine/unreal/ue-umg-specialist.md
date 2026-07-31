# Agent 测试规格：ue-umg-specialist

## Agent 摘要
- **领域**：UMG widget 层次结构设计、数据绑定模式、CommonUI 输入路由和操作标签、widget 样式（WidgetStyle 资产）、UI 优化（widget 池、ListView、失效）
- **不负责**：UX 流程和屏幕导航设计（ux-designer）、游戏逻辑（gameplay-programmer）、后端数据源（游戏代码）、服务器通信
- **模型层级**：Sonnet
- **Gate ID**：无；将 UX 流程决策推迟给 ux-designer

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且领域特定（引用 UMG、widget 层次结构、CommonUI）
- [ ] `allowed-tools:` 列表与 agent 角色匹配（UI 资产和 Blueprint 文件的 Read/Write；无服务器或游戏源代码工具）
- [ ] 模型层级为 Sonnet（专业人员默认值）
- [ ] Agent 定义不声称对 UX 流程、导航架构或游戏数据逻辑有权限

---

## 测试用例

### 用例 1：领域内请求 — 带数据绑定的库存 widget
**输入**："创建一个显示物品槽网格的库存 widget。每个槽应显示物品图标、数量和稀有度颜色。它需要在库存变化时更新。"
**预期行为**：
- 生成 UMG widget 结构：包含 UniformGridPanel 或 TileView 的父 WBP_Inventory，每个物品有一个子 WBP_InventorySlot widget
- 描述数据绑定方法：Inventory Component 上的 Event Dispatchers 触发刷新，或带有实现 IUserObjectListEntry 的 UObject 物品数据类的 ListView
- 指定稀有度颜色的驱动方式：WidgetStyle 资产或数据表查找，而非硬编码颜色值
- 输出包括 widget 层次结构、绑定模式和刷新触发机制

### 用例 2：领域外请求 — UX 流程设计
**输入**："设计我们库存系统的完整导航流程 — 玩家如何打开它、转换到角色属性以及退出到暂停菜单。"
**预期行为**：
- 不生成导航流程或屏幕转换架构
- 明确指出："导航流程和屏幕转换设计由 ux-designer 负责；一旦流程定义好，我可以实现 UMG widget 结构"
- 没有 UX 规范不做 UX 决策（返回按钮行为、转换动画、模态与全屏）

### 用例 3：领域边界 — CommonUI 输入操作不匹配
**输入**："我们的库存 widget 不响应控制器返回按钮。我们正在使用 CommonUI。"
**预期行为**：
- 识别可能原因：widget 的返回输入操作标签与项目注册的 CommonUI InputAction 数据资产不匹配
- 解释 CommonUI 输入路由模型：widget 通过 `CommonUI_InputAction` 标签声明输入操作；CommonActivatableWidget 处理路由
- 提供修复：验证 widget 的返回操作标签是否与项目 CommonUI 输入操作数据表中注册的标签匹配
- 将此与硬件输入绑定问题区分开来（后者是 Enhanced Input 的领域）

### 用例 4：Widget 性能问题 — 每帧多个 widget 实例
**输入**："我们的排行榜 widget 一次创建 500 个 WBP_LeaderboardRow 实例。打开排行榜时游戏卡顿 300ms。"
**预期行为**：
- 识别根本原因：单帧内 500 个 widget 实例化导致构建卡顿
- 建议切换到带虚拟化的 ListView 或 TileView — 仅构建可见行
- 解释 ListView 数据对象的 IUserObjectListEntry 接口要求
- 如果不适合 ListView，建议池化：预实例化固定数量的行并用新数据回收它们
- 输出是具体推荐，包含要使用的特定 UMG 组件，而非模糊的"优化它"

### 用例 5：上下文传递 — CommonUI 设置已配置
**输入上下文**：项目使用 CommonUI，已注册以下 InputAction 标签：UI.Action.Confirm、UI.Action.Back、UI.Action.Pause、UI.Action.Secondary。
**输入**："向库存 widget 添加一个与 CommonUI 配合使用的'排序库存'按钮。"
**预期行为**：
- 使用 UI.Action.Secondary（或如果 Secondary 已分配，则建议注册新标签如 UI.Action.Sort）
- 不会在未注明必须在 CommonUI 数据表中注册的情况下发明新的 InputAction 标签
- 当 CommonUI 是既定模式时，不会使用非 CommonUI 输入绑定方法（例如 Event Graph 中的原始按键）
- 在推荐中明确引用提供的标签列表

---

## 协议合规

- [ ] 保持在声明领域内（UMG 结构、数据绑定、CommonUI、widget 性能）
- [ ] 将 UX 流程和导航设计请求重定向到 ux-designer
- [ ] 返回结构化发现（widget 层次结构 + 绑定模式），而非自由格式意见
- [ ] 使用上下文中现有的 CommonUI InputAction 标签；不发明新的而不标记注册要求
- [ ] 对于大型集合，在 widget 池之前推荐虚拟化列表（ListView/TileView）

---

## 覆盖说明
- 用例 3（CommonUI 输入路由）要求项目配置了 CommonUI；如果项目不使用 CommonUI，则跳过测试
- 用例 4（性能）是高影响失败模式 — 300ms 卡顿是发布阻塞的；优先处理此测试用例
- 用例 5 是 UI 流水线一致性的最重要上下文感知测试
- 无自动化运行器；手动或通过 `/skill-test` 审查
