# 技术偏好

<!-- 由 /setup-engine 填充。随着用户在开发过程中做决策而更新。 -->
<!-- 所有 Agent 引用此文件以获取项目特定的标准和约定。 -->

## 引擎与语言

- **Engine（引擎）**: [待配置 — 运行 /setup-engine]
- **Language（语言）**: [待配置]
- **Rendering（渲染）**: [待配置]
- **Physics（物理）**: [待配置]

## 输入与平台

<!-- 由 /setup-engine 编写。由 /ux-design、/ux-review、/test-setup、/team-ui 和 /dev-story 读取 -->
<!-- 以将交互规格、测试辅助和实现限定到正确的输入方法。 -->

- **Target Platforms（目标平台）**: [待配置 — 例如 PC、Console、Mobile、Web]
- **Input Methods（输入方法）**: [待配置 — 例如 Keyboard/Mouse、Gamepad、Touch、Mixed]
- **Primary Input（主要输入）**: [待配置 — 此游戏的主导输入]
- **Gamepad Support（手柄支持）**: [待配置 — Full / Partial / None]
- **Touch Support（触摸支持）**: [待配置 — Full / Partial / None]
- **Platform Notes（平台备注）**: [待配置 — 任何平台特定的 UX 约束]

## 命名约定

- **Classes（类）**: [待配置]
- **Variables（变量）**: [待配置]
- **Signals/Events（信号/事件）**: [待配置]
- **Files（文件）**: [待配置]
- **Scenes/Prefabs（场景/预制体）**: [待配置]
- **Constants（常量）**: [待配置]

## 性能预算

- **Target Framerate（目标帧率）**: [待配置]
- **Frame Budget（帧预算）**: [待配置]
- **Draw Calls（绘制调用）**: [待配置]
- **Memory Ceiling（内存上限）**: [待配置]

## 测试

- **Framework（框架）**: [待配置]
- **Minimum Coverage（最低覆盖率）**: [待配置]
- **Required Tests（必需测试）**: 平衡公式、游戏系统、网络（如果适用）

## 禁止的模式

<!-- 添加永远不应出现在此项目代码库中的模式 -->
- [尚未配置 — 随着架构决策进行添加]

## 允许的库 / 插件

<!-- 在此添加已批准的第三方依赖 -->
- [尚未配置 — 随着依赖被批准而添加]

## 架构决策日志

<!-- 快速参考，链接到 docs/architecture/ 中的完整 ADR -->
- [尚无 ADR — 使用 /architecture-decision 创建一个]

## 引擎专家

<!-- 当引擎被配置时由 /setup-engine 编写。 -->
<!-- 由 /code-review、/architecture-decision、/architecture-review 和团队技能读取 -->
<!-- 以了解对于引擎特定验证应派生哪个专家。 -->

- **Primary（主要）**: [待配置 — 运行 /setup-engine]
- **Language/Code Specialist（语言/代码专家）**: [待配置]
- **Shader Specialist（Shader 专家）**: [待配置]
- **UI Specialist（UI 专家）**: [待配置]
- **Additional Specialists（额外专家）**: [待配置]
- **Routing Notes（路由备注）**: [待配置]

### 文件扩展名路由

<!-- 技能使用此表根据文件类型选择正确的专家。 -->
<!-- 如果某行显示 [待配置]，回退到该文件类型的 Primary。 -->

| 文件扩展名 / 类型 | 派生的专家 |
|-----------------------|---------------------|
| 游戏代码（主要语言） | [待配置] |
| Shader / 材质文件 | [待配置] |
| UI / 屏幕文件 | [待配置] |
| Scene / 预制体 / 关卡文件 | [待配置] |
| Native extension / 插件文件 | [待配置] |
| 通用架构审查 | Primary |