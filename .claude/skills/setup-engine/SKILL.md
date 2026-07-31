---
name: setup-engine
description: "Configure the project's game engine and version. Pins the engine in CLAUDE.md, detects knowledge gaps, and populates engine reference docs via WebSearch when the version is beyond the LLM's training data."
argument-hint: "[engine] | [engine version] | refresh | upgrade [old-version] [new-version] | no args for guided selection"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, WebSearch, WebFetch, Task, AskUserQuestion
model: sonnet
---

当此 skill 被调用时：

## 1. 解析参数

四种模式：

- **完整规格**：`/setup-engine godot 4.6` — 提供了引擎和版本
- **仅引擎**：`/setup-engine unity` — 提供了引擎，版本将被查找
- **无参数**：`/setup-engine` — 完全引导模式（引擎推荐 + 版本）
- **刷新**：`/setup-engine refresh` — 更新参考文档（见第 10 节）
- **升级**：`/setup-engine upgrade [旧版本] [新版本]` — 迁移到新引擎版本（见第 11 节）

---

## 2. 引导模式（无参数）

如果未指定引擎，运行交互式引擎选择过程：

### 检查现有游戏概念
- 如果存在，读取 `design/gdd/game-concept.md` — 提取类型、范围、平台
  目标、艺术风格、团队规模以及来自 `/brainstorm` 的任何引擎推荐
- 如果概念不存在，告知用户：
  > "未找到游戏概念。考虑先运行 `/brainstorm` 发现你想
  > 构建什么 — 它也会推荐一个引擎。或者告诉我你的
  > 游戏，我可以帮助你选择。"

### 如果用户想在没有概念的情况下选择，按以下顺序询问：

**问题 1 — 先前经验**（始终首先通过 `AskUserQuestion` 询问）：
- 提示："你之前在这些引擎中工作过吗？"
- 选项：`Godot` / `Unity` / `Unreal Engine 5` / `多个 — 我会解释` / `都没有`
- 如果他们选择了特定引擎 → 推荐该引擎。先前经验胜过所有其他因素。与他们确认并跳过矩阵。
- 如果选择"没有"或"多个" → 继续下面的问题。

**问题 2-6 — 决策矩阵输入**（仅在无先前引擎经验时）：

**问题 2 — 目标平台**（始终第二个通过 `AskUserQuestion` 询问 — 平台在任何其他因素之前消除或严重加权引擎）：
- 提示："你这款游戏的目标平台是什么？"
- 选项：`PC (Steam / Epic)` / `Mobile (iOS / Android)` / `Console` / `Web / Browser` / `多个平台`
- 直接输入推荐的平规则：
  - Mobile → 强烈推荐 Unity；Unreal 不适合；Godot 对简单移动游戏可行
  - Console → Unity 或 Unreal；Godot 主机支持需要第三方发行商或大量额外工作
  - Web → Godot 干净地导出到 web；Unity WebGL 功能正常；Unreal 网络支持差
  - 仅 PC → 所有引擎可行；其他因素决定
  - 多个 → Unity 在 PC/移动/主机之间最具可移植性

1. **什么类型的游戏？**（2D、3D 或两者都有？）
2. **主要输入方式？**（键盘/鼠标、手柄、触摸或混合？）
3. **团队规模和经验？**（独立初学者、独立有经验、小团队？）
4. **任何强烈的语言偏好？**（GDScript、C#、C++、可视化脚本？）
5. **引擎许可预算？**（仅免费，还是可以商业许可？）

### 产生推荐

不要使用消除引擎的简单评分矩阵。相反，根据用户的配置文件与下面的诚实权衡进行推理，然后呈现 1-2 个带有完整上下文的推荐。始终以用户选择结束 — 永远不要强制判定。

**引擎诚实权衡：**

**Godot 4**
- 真正的优势：2D（同类最佳）、风格化/独立 3D、快速迭代、永远免费（MIT）、开源、最平缓的学习曲线、最适合想要完全控制的独立开发者
- 真正的限制：3D 生态系统与 Unity/Unreal 相比薄弱（3D 特定问题的教程、资源、社区答案较少）；大型开放世界 3D 非常困难，在 Godot 中基本未经测试；主机导出需要第三方发行商或大量额外工作；较小的专业就业市场
- 许可现实：真正免费，永远无收入门槛。MIT 许可意味着你拥有一切。
- 最适合：任何范围的 2D 游戏；风格化/氛围 3D；封闭的 3D 世界（非开放世界）；学习曲线重要的第一个游戏项目；预算在任何规模都是硬性约束的项目

**Unity**
- 真正的优势：中档 3D 和移动的行业标准；庞大的资产商店和教程生态系统；C# 是专业语言；独立游戏最佳主机认证支持；几乎每种类型的强大社区
- 真正的限制：2023 年的许可争议损害了信任（运行时费用被提出然后撤回 — 政策变更的风险仍然存在）；C# 比 GDScript 有更陡的初始曲线；对于简单项目编辑器比 Godot 重
- 许可现实：在 200K 收入 AND 200K 安装以下免费（Unity Personal/Plus）。只有游戏真正成功时才变得昂贵 — 大多数独立游戏永远达不到这个门槛。2023 年的争议值得了解，但实际当前条款对大多数独立开发者来说是合理的。
- 最适合：移动游戏；中档 3D；目标主机的游戏；有 C# 背景的开发者；需要大型资产商店的项目；2-5 人团队

**Unreal Engine 5**
- 真正的优势：同类最佳的 3D 视觉效果（Lumen、Nanite、混沌物理）；AAA 和照片级逼真 3D 的行业标准；大型开放世界支持成熟且经过生产测试；Blueprint 可视化脚本降低 C++ 障碍；非常适合目标高端 PC 或主机的游戏
- 真正的限制：最陡的学习曲线；最重的编辑器（编译时间长、项目规模大）；对风格化/2D/小范围游戏过度杀伤；C++ 真的很难；不适合移动或网络；超过 100 万美元总收入后 5% 版税
- 许可现实：5% 版税仅适用于每个标题超过 100 万美元总收入之后。对于第一款游戏或任何未达到 100 万美元的游戏，它不花钱。这个门槛足够高，大多数独立开发者永远不会支付。
- 最适合：AAA 质量 3D；大型开放世界游戏；照片级逼真视觉效果；有 C# 经验或愿意使用 Blueprint 的开发者；目标高端 PC/主机的游戏，视觉保真度是核心卖点

**类型特定指引**（将此纳入推荐）：
- 2D 任何风格 → 强烈推荐 Godot
- 3D 风格化 / 氛围 / 封闭世界 → Godot 可行，Unity 可靠替代
- 3D 开放世界（大型、无缝）→ Unity 或 Unreal；Godot 对此未经生产验证
- 3D 照片级逼真 / AAA 质量 → Unreal
- 移动优先 → 强烈推荐 Unity
- 主机优先 → Unity 或 Unreal；Godot 主机支持需要额外工作
- 恐怖 / 叙事 / 步行模拟 → 任何引擎；匹配艺术风格和团队经验
- 动作 RPG / 魂类 → 3D 用 Unity 或 Unreal；社区支持和资源在这里很重要
- 2D 平台游戏 → Godot
- 策略 / 俯视 / RTS → Godot 或 Unity 取决于 2D vs 3D

**推荐格式：**
1. 显示一个比较表，以用户的特定因素为行
2. 给出带有诚实推理的主要推荐
3. 命名最佳替代方案以及何时选择它
4. 明确声明："这是一个起点，不是判定 — 你总是可以迁移引擎，许多开发者在项目之间切换。"
5. 使用 `AskUserQuestion` 确认："这个推荐感觉对吗，还是你想探索不同的引擎？"
   - 选项：`[主要引擎] (推荐)` / `[替代引擎]` / `[第三引擎]` / `进一步探索` / `输入其他`

**如果用户选择"进一步探索"：**
使用 `AskUserQuestion` 提供概念特定的深入主题。始终从用户的实际概念生成这些选项 — 不要使用通用选项。始终至少包括：
- 主要引擎对此概念的具体限制（例如，"Godot 3D 对 [类型] 实际上能走多远？"）
- 替代引擎对此概念的具体权衡
- 语言选择对此概念技术挑战的影响
- 任何概念特定的技术关注点（例如，自适应音频、开放世界流式传输、多人网络代码）

用户可以选择多个主题。在返回引擎确认问题之前，深入回答每个选定的主题。

---

## 3. 查找当前版本

一旦选择了引擎：

- 如果提供了版本，使用它
- 如果未提供版本，使用 WebSearch 查找最新稳定版本：
  - 搜索：`"[引擎] 最新稳定版本 [当前年份]"`
  - 与用户确认："最新的稳定 [引擎] 是 [版本]。使用这个吗？"

---

## 4. 更新 CLAUDE.md 技术栈

### 语言选择（仅 Godot）

如果选择了 Godot，在显示提议的技术栈之前询问用户要使用哪种语言：

> "Godot 支持两种主要语言：
>
>   **A) GDScript** — 类似 Python，Godot 原生，最快迭代。最适合初学者、独立开发者和来自 Python 或 Lua 的团队。
>   **B) C#** — .NET 8+，Unity 开发者熟悉，更强的 IDE 工具（Rider / Visual Studio），在重逻辑上有轻微性能优势。
>   **C) 两者** — GDScript 用于游戏/UI 脚本，C# 用于性能关键系统。高级设置 — 需要 .NET SDK 与 Godot 一起。
>
> 这个项目主要使用哪种？"

记录选择。它决定了 CLAUDE.md 模板、命名约定、专家路由以及整个项目中为代码文件 spawn 哪个 agent。

---

读取 `CLAUDE.md` 并向用户展示提议的技术栈更改。
询问："我可以将这些引擎设置写入 `CLAUDE.md` 吗？"

在进行任何编辑之前等待确认。

更新技术栈部分，将 `[CHOOSE]` 占位符替换为实际值：

**对于 Godot** — 使用与上面选择的语言匹配的模板。见本 skill 底部的 **附录 A** 获取所有三种变体（GDScript、C#、两者）。

**对于 Unity：**
```markdown
- **引擎**: Unity [版本]
- **语言**: C#
- **构建系统**: Unity Build Pipeline
- **资源管线**: Unity Asset Import Pipeline + Addressables
```

**对于 Unreal：**
```markdown
- **引擎**: Unreal Engine [版本]
- **语言**: C++（主要），Blueprint（游戏原型）
- **构建系统**: Unreal Build Tool (UBT)
- **资源管线**: Unreal Content Pipeline
```

---

## 5. 填充技术偏好

更新 CLAUDE.md 后，创建或更新 `.claude/docs/technical-preferences.md`，使用适合引擎的默认值。首先读取现有模板，然后填写：

### 引擎和语言部分
- 从步骤 4 的引擎选择中填写

### 命名约定（引擎默认值）

**对于 Godot** — 见 **附录 A** 获取 GDScript、C# 和两者变体。

**对于 Unity (C#)：**
- 类：PascalCase（例如，`PlayerController`）
- 公共字段/属性：PascalCase（例如，`MoveSpeed`）
- 私有字段：_camelCase（例如，`_moveSpeed`）
- 方法：PascalCase（例如，`TakeDamage()`）
- 文件：PascalCase 匹配类（例如，`PlayerController.cs`）
- 常量：PascalCase 或 UPPER_SNAKE_CASE

**对于 Unreal (C++)：**
- 类：前缀 PascalCase（`A` 表示 Actor，`U` 表示 UObject，`F` 表示 struct）
- 变量：PascalCase（例如，`MoveSpeed`）
- 函数：PascalCase（例如，`TakeDamage()`）
- 布尔值：`b` 前缀（例如，`bIsAlive`）
- 文件：匹配不带前缀的类（例如，`PlayerController.h`）

### 输入和平台部分

使用第 2 节中收集的答案（或从游戏概念中提取）填充 `## Input & Platform`。使用此映射推导值：

| 平台目标 | 手柄支持 | 触摸支持 |
|-----------------|-----------------|---------------|
| 仅 PC | 部分（推荐） | 无 |
| 主机 | 完整 | 无 |
| 移动 | 无 | 完整 |
| PC + 主机 | 完整 | 无 |
| PC + 移动 | 部分 | 完整 |
| Web | 部分 | 部分 |

对于 **Primary Input**，使用游戏类型的主导输入：
- 目标主机的动作/RPG/平台游戏 → 手柄
- 策略/点击/RTS → 键盘/鼠标
- 移动游戏 → 触摸
- 跨平台 → 询问用户

展示推导出的值并要求用户在写入之前确认或调整。

填充部分示例：
```markdown
## Input & Platform
- **目标平台**: PC, Console
- **输入方式**: 键盘/鼠标, 手柄
- **主要输入**: 手柄
- **手柄支持**: 完整
- **触摸支持**: 无
- **平台备注**: 所有 UI 必须支持 d-pad 导航。无仅悬停交互。
```

### 剩余部分
- **性能预算**：使用 `AskUserQuestion`：
  - 提示："我现在应该设置默认性能预算，还是留到以后？"
  - 选项：`[A] 现在设置默认值（60fps、16.6ms 帧预算、引擎适当的 draw call 限制）` / `[B] 留为 [待配置] — 我了解目标硬件后会设置这些`
  - 如果选择 [A]：用建议的默认值填充。如果选择 [B]：留为占位符。
- **测试**：建议引擎适当的框架（Godot 用 GUT，Unity 用 NUnit 等）— 在添加前询问。
- **禁止的模式**：留为占位符 — 不要预填充。
- **允许的库**：留为占位符 — 不要预填充项目当前不需要的依赖。仅在此处主动集成库时添加，不要投机性地添加。

> **护栏**：永远不要向 Allowed Libraries 添加投机性依赖。例如，除非 Steam 集成正在本会话中积极开始，否则不要添加 GodotSteam。发布后集成应在工作开始时添加到 Allowed Libraries，而不是在引擎设置期间。

### 引擎专家路由

还要填充 `technical-preferences.md` 中的 `## Engine Specialists` 部分，使用所选引擎的正确路由：

**对于 Godot** — 见 **附录 A** 获取与所选语言匹配的路由表。

**对于 Unity：**
```markdown
## Engine Specialists
- **主要**: unity-specialist
- **语言/代码专家**: unity-specialist（C# 审查 — 主要的涵盖）
- **Shader 专家**: unity-shader-specialist（Shader Graph、HLSL、URP/HDRP 材质）
- **UI 专家**: unity-ui-specialist（UI Toolkit UXML/USS、UGUI Canvas、运行时 UI）
- **额外专家**: unity-dots-specialist（ECS、Jobs 系统、Burst 编译器）、unity-addressables-specialist（资源加载、内存管理、内容目录）
- **路由备注**: 调用主要进行架构和通用 C# 代码审查。调用 DOTS 专家进行任何 ECS/Jobs/Burst 代码。调用 shader 专家进行渲染和视觉效果。调用 UI 专家进行所有界面实现。调用 Addressables 专家进行资源管理系统。

### 文件扩展名路由

| 文件扩展名 / 类型 | 要 Spawn 的专家 |
|-----------------------|---------------------|
| 游戏代码（.cs 文件） | unity-specialist |
| Shader / 材质文件（.shader、.shadergraph、.mat） | unity-shader-specialist |
| UI / 屏幕文件（.uxml、.uss、Canvas prefab） | unity-ui-specialist |
| 场景 / prefab / 关卡文件（.unity、.prefab） | unity-specialist |
| 原生扩展 / 插件文件（.dll、原生插件） | unity-specialist |
| 通用架构审查 | unity-specialist |
```

**对于 Unreal：**
```markdown
## Engine Specialists
- **主要**: unreal-specialist
- **语言/代码专家**: ue-blueprint-specialist（Blueprint 图）或 unreal-specialist（C++）
- **Shader 专家**: unreal-specialist（无专门 shader 专家 — 主要的涵盖材质）
- **UI 专家**: ue-umg-specialist（UMG 小部件、CommonUI、输入路由、小部件样式）
- **额外专家**: ue-gas-specialist（Gameplay Ability System、属性、游戏效果）、ue-replication-specialist（属性复制、RPC、客户端预测、网络代码）
- **路由备注**: 调用主要进行 C++ 架构和广泛的引擎决策。调用 Blueprint 专家进行 Blueprint 图架构和 BP/C++ 边界设计。调用 GAS 专家进行所有能力和属性代码。调用复制专家进行任何多人或网络系统。调用 UMG 专家进行所有 UI 实现。

### 文件扩展名路由

| 文件扩展名 / 类型 | 要 Spawn 的专家 |
|-----------------------|---------------------|
| 游戏代码（.cpp、.h 文件） | unreal-specialist |
| Shader / 材质文件（.usf、.ush、材质资源） | unreal-specialist |
| UI / 屏幕文件（.umg、UMG Widget Blueprint） | ue-umg-specialist |
| 场景 / prefab / 关卡文件（.umap、.uasset） | unreal-specialist |
| 原生扩展 / 插件文件（Plugin .uplugin、模块） | unreal-specialist |
| Blueprint 图（.uasset BP 类） | ue-blueprint-specialist |
| 通用架构审查 | unreal-specialist |
```

### 协作步骤
向用户展示填充后的偏好。对于 Godot，包括所选语言并注明完整命名约定和路由表的位置：
> "这是 [引擎]（[如果是 Godot 则注明语言]）的默认技术偏好。命名约定和专家路由在本 skill 的附录 A 中 — 我将应用 [GDScript/C#/两者] 变体。想自定义其中任何一个，还是保存默认值？"

对于所有其他引擎，直接展示默认值而不引用附录。

在写入文件之前等待批准。

---

## 6. 确定知识缺口

检查引擎版本是否可能超出 LLM 的训练数据。

**已知大致覆盖范围**（随模型变化更新）：
- LLM 知识截止日期：**2025 年 5 月**
- Godot：训练数据可能覆盖到 ~4.3
- Unity：训练数据可能覆盖到 ~2023.x / 早期 6000.x
- Unreal：训练数据可能覆盖到 ~5.3 / 早期 5.4

将用户选择的版本与这些基线进行比较：

- **在训练数据内** → `低风险` — 参考文档可选但推荐
- **接近边缘** → `中风险` — 推荐参考文档
- **超出训练数据** → `高风险` — 需要参考文档

告知用户他们属于哪个类别以及为什么。

---

## 7. 填充引擎参考文档

### 如果在训练数据内（低风险）：

创建最小的 `docs/engine-reference/<引擎>/VERSION.md`：

```markdown
# [引擎] — 版本参考

| 字段 | 值 |
|-------|-------|
| **引擎版本** | [版本] |
| **项目固定** | [今天的日期] |
| **LLM 知识截止日期** | 2025 年 5 月 |
| **风险级别** | 低 — 版本在 LLM 训练数据内 |

## 注意

此引擎版本在 LLM 的训练数据内。引擎参考
文档是可选的，但如果 agent 建议错误的 API，以后可以添加。

随时运行 `/setup-engine refresh` 以填充完整参考文档。
```

不要创建 breaking-changes.md、deprecated-apis.md 等 — 它们会增加上下文成本而价值很小。

### 如果超出训练数据（中或高风险）：

通过搜索网络创建完整的参考文档集：

1. **搜索官方迁移/升级指南**：
   - `"[引擎] [旧版本] 到 [新版本] 迁移指南"`
   - `"[引擎] [版本] 破坏性变更"`
   - `"[引擎] [版本] 变更日志"`
   - `"[引擎] [版本] 已弃用 API"`

2. **从官方文档获取并提取**：
   - 从训练截止日期到当前版本之间的每个版本的破坏性变更
   - 带替代方案的已弃用 API
   - 新功能和最佳实践

询问："我可以在 `docs/engine-reference/<引擎>/` 下创建引擎参考文档吗？"

在写入任何文件之前等待确认。

3. **创建完整的参考目录**：
   ```
   docs/engine-reference/<引擎>/
   ├── VERSION.md              # 版本固定 + 知识缺口分析
   ├── breaking-changes.md     # 逐版本破坏性变更
   ├── deprecated-apis.md      # "不要使用 X → 使用 Y" 表格
   ├── current-best-practices.md  # 训练截止日期以来的新实践
   └── modules/                # 按子系统参考（按需创建）
   ```

4. **使用网络搜索的真实数据填充每个文件**，遵循现有参考文档中建立的格式。每个文件必须有"Last verified: [日期]"标题。

5. **对于模块文件**：仅为发生重大变化的子系统创建模块。不要创建空的最小模块文件。

---

## 8. 更新 CLAUDE.md 导入

询问："我可以更新 `CLAUDE.md` 中的 `@` 导入以指向新的引擎参考吗？"

等待确认，然后更新"Engine Version Reference"下的 `@` 导入以指向正确的引擎：

```markdown
## Engine Version Reference

@docs/engine-reference/<引擎>/VERSION.md
```

如果之前的导入指向不同的引擎（例如，从 Godot 切换到 Unity），更新它。

---

## 9. 更新 Agent 指令

在进行任何编辑之前询问："我可以在引擎专家 agent 文件中添加 Version Awareness 部分吗？"

对于所选引擎的专家 agent，验证它们是否有"Version Awareness"部分。如果没有，按照现有 Godot 专家 agent 中的模式添加一个。

该部分应指示 agent：
1. 读取 `docs/engine-reference/<引擎>/VERSION.md`
2. 在建议代码之前检查已弃用 API
3. 检查相关版本转换的破坏性变更
4. 使用 WebSearch 验证不确定的 API

---

## 10. Refresh 子命令

如果作为 `/setup-engine refresh` 调用：

1. 读取现有的 `docs/engine-reference/<引擎>/VERSION.md` 以获取当前引擎和版本
2. 使用 WebSearch 检查：
   - 自上次验证以来的新引擎版本
   - 更新的迁移指南
   - 新近弃用的 API
3. 用新发现更新所有参考文档
4. 更新所有修改文件的"Last verified"日期
5. 报告变更内容

---

## 11. Upgrade 子命令

如果作为 `/setup-engine upgrade [旧版本] [新版本]` 调用：

### 步骤 1 — 读取当前版本状态

读取 `docs/engine-reference/<引擎>/VERSION.md` 以确认当前固定的版本、风险级别和已记录的任何迁移笔记 URL。如果参数中未提供 `旧版本`，使用此文件中的固定版本。

### 步骤 2 — 获取迁移指南

使用 WebSearch 和 WebFetch 定位 `旧版本` 和 `新版本` 之间的官方迁移指南：

- 搜索：`"[引擎] [旧版本] 到 [新版本] 迁移指南"`
- 搜索：`"[引擎] [新版本] 破坏性变更变更日志"`
- 如果已记录，从 VERSION.md 获取迁移指南 URL，或使用通过搜索找到的 URL。

提取：重命名的 API、删除的 API、更改的默认值、行为变更以及任何"必须迁移"项。

### 步骤 3 — 升级前审计

扫描 `src/` 中使用了目标版本中已知已弃用或更改的 API 的代码：

- 使用 Grep 搜索从迁移指南中提取的已弃用 API 名称（例如，旧函数名、删除的节点类型、更改的属性名）
- 列出每个匹配的文件以及找到的具体 API 引用

将审计结果展示为表格：

```
升级前审计: [引擎] [旧版本] → [新版本]
==========================================================

需要更改的文件:
  文件                              | 找到的已弃用 API       | 工作量
  --------------------------------- | -------------------------- | ------
  src/gameplay/player_movement.gd   | old_api_name               | 低
  src/ui/hud.gd                     | removed_node_type          | 中

需要注意的破坏性变更:
  - [迁移指南中的变更描述]
  - [迁移指南中的变更描述]

推荐的迁移顺序（按依赖关系排序）:
  1. [依赖最少的系统/层级优先]
  2. [下一个系统]
  ...
```

如果在 `src/` 中未找到已弃用的 API，报告："在 src/ 中未找到已弃用的 API 使用 — 升级可能是低风险的。"

### 步骤 4 — 更新前确认

在进行任何更改之前询问用户：

> "升级前审计完成。发现 [N] 个文件使用已弃用的 API。
> 继续将 VERSION.md 升级到 [新版本] 吗？
> （这将更新固定版本并添加迁移笔记 — 它不会
> 更改任何源文件。源迁移是手动或通过故事完成的。）"

在继续之前等待明确确认。

### 步骤 5 — 更新 VERSION.md

确认后：

1. 更新 `docs/engine-reference/<引擎>/VERSION.md`：
   - `Engine Version` → `[新版本]`
   - `Project Pinned` → 今天的日期
   - `Last Docs Verified` → 今天的日期
   - 如果新版本超出 LLM 知识截止日期，重新评估并更新 `Risk Level` 和 `Post-Cutoff Version Timeline` 表格
   - 添加 `## Migration Notes — [旧版本] → [新版本]` 部分，包含：迁移指南 URL、关键破坏性变更、此项目中发现的已弃用 API 以及审计中的推荐迁移顺序

2. 如果引擎参考目录中存在 `breaking-changes.md` 或 `deprecated-apis.md`，将新版本的更改追加到这些文件中。

### 步骤 6 — 升级后提醒

更新 VERSION.md 后，输出：

```
VERSION.md 已更新: [引擎] [旧版本] → [新版本]

后续步骤:
1. 迁移上面列出的 [N] 个文件中的已弃用 API 使用
2. 在实际升级引擎二进制文件后运行 /setup-engine refresh 以验证没有遗漏新的弃用
3. 运行 /architecture-review — 引擎升级可能使引用特定 API 或引擎功能的 ADR 失效
4. 如果任何 ADR 失效，运行 /propagate-design-change 以更新下游故事
```

---

## 12. 输出摘要

设置完成后，输出：

```
引擎设置完成
=====================
引擎:          [名称] [版本]
语言:        [GDScript | C# | GDScript + C# | C# | C++ + Blueprint]
知识风险:  [低/中/高]
参考文档:  [已创建/已跳过]
CLAUDE.md:       [已更新]
技术偏好:      [已创建/已更新]
Agent 配置:    [已验证]

后续步骤:
1. 审查 docs/engine-reference/<引擎>/VERSION.md
2. [如果来自 /brainstorm] 运行 /map-systems 将你的概念分解为各个系统
3. [如果来自 /brainstorm] 运行 /design-system 编写每系统 GDD（引导式，逐节）
4. [如果来自 /brainstorm] 运行 /prototype [核心机制] 在编写 GDD 之前验证核心想法
5. [如果是全新开始] 运行 /brainstorm 发现你的游戏概念
6. 创建你的第一个里程碑: /sprint-plan new
```

---

判定：**COMPLETE** — 引擎已配置且参考文档已填充。

## 护栏

- 永远不要猜测引擎版本 — 始终通过 WebSearch 或用户确认验证
- 永远不要未经询问就覆盖现有参考文档 — 追加或更新
- 如果参考文档已存在于不同引擎，在替换前询问
- 在进行 CLAUDE.md 编辑之前始终向用户展示你将要更改的内容
- 如果 WebSearch 返回模糊结果，向用户展示并让他们决定
- 当用户选择 **GDScript** 时：从附录 A1 精确复制 GDScript CLAUDE.md 模板。永远不要在 Language 字段中添加"C++ via GDExtension"。GDScript 项目可能使用 GDExtension，但它不是主要项目语言。路由表中的 `godot-gdextension-specialist` 在需要原生扩展时可用 — 它不会使 C++ 成为项目语言。

---

## 附录 A — Godot 语言配置

所有语言依赖配置的 Godot 特定变体。从第 4 和第 5 节引用 — 仅在 Godot 是所选引擎时相关。使用与第 4 节中选择的语言匹配的子部分。

---

### A1. CLAUDE.md 技术栈模板

**GDScript：**
```markdown
- **引擎**: Godot [版本]
- **语言**: GDScript
- **构建系统**: SCons（引擎），Godot Export Templates
- **资源管线**: Godot Import System + 自定义资源管线
```

> **护栏**：使用此 GDScript 模板时，将 Language 字段写为精确的 "`GDScript`" — 无添加。不要追加"C++ via GDExtension"或任何其他语言。下面的 C# 模板包含 GDExtension，因为 C# 项目通常包装原生代码；GDScript 项目不这样做。

**C#：**
```markdown
- **引擎**: Godot [版本]
- **语言**: C#（.NET 8+，主要），C++ via GDExtension（仅原生插件）
- **构建系统**: .NET SDK + Godot Export Templates
- **资源管线**: Godot Import System + 自定义资源管线
```

**两者 — GDScript + C#：**
```markdown
- **引擎**: Godot [版本]
- **语言**: GDScript（游戏/UI 脚本），C#（性能关键系统），C++ via GDExtension（仅原生）
- **构建系统**: .NET SDK + Godot Export Templates
- **资源管线**: Godot Import System + 自定义资源管线
```

---

### A2. 命名约定

**GDScript：**
- 类：PascalCase（例如，`PlayerController`）
- 变量/函数：snake_case（例如，`move_speed`）
- 信号：snake_case 过去时（例如，`health_changed`）
- 文件：snake_case 匹配类（例如，`player_controller.gd`）
- 场景：PascalCase 匹配根节点（例如，`PlayerController.tscn`）
- 常量：UPPER_SNAKE_CASE（例如，`MAX_HEALTH`）

**C#：**
- 类：PascalCase（`PlayerController`）— 还必须为 `partial`
- 公共属性/字段：PascalCase（`MoveSpeed`、`JumpVelocity`）
- 私有字段：`_camelCase`（`_currentHealth`、`_isGrounded`）
- 方法：PascalCase（`TakeDamage()`、`GetCurrentHealth()`）
- 信号委托：PascalCase + `EventHandler` 后缀（`HealthChangedEventHandler`）
- 文件：PascalCase 匹配类（`PlayerController.cs`）
- 场景：PascalCase 匹配根节点（`PlayerController.tscn`）
- 常量：PascalCase（`MaxHealth`、`DefaultMoveSpeed`）

**两者 — GDScript + C#：**
对 `.gd` 文件使用 GDScript 约定，对 `.cs` 文件使用 C# 约定。混合语言文件不存在 — 边界是按文件的。当不确定新系统应使用哪种语言时询问用户并将决定记录在 `technical-preferences.md` 中。

---

### A3. 引擎专家路由

**GDScript：**
```markdown
## Engine Specialists
- **主要**: godot-specialist
- **语言/代码专家**: godot-gdscript-specialist（所有 .gd 文件）
- **Shader 专家**: godot-shader-specialist（.gdshader 文件，VisualShader 资源）
- **UI 专家**: godot-specialist（无专门 UI 专家 — 主要的涵盖所有 UI）
- **额外专家**: godot-gdextension-specialist（GDExtension / 仅原生 C++ 绑定）
- **路由备注**: 调用主要进行架构决策、ADR 验证和跨领域代码审查。调用 GDScript 专家进行代码质量、信号架构、静态类型强制和 GDScript 习惯用法。调用 shader 专家进行材质设计和 shader 代码。仅在涉及原生扩展时调用 GDExtension 专家。

### 文件扩展名路由

| 文件扩展名 / 类型 | 要 Spawn 的专家 |
|-----------------------|---------------------|
| 游戏代码（.gd 文件） | godot-gdscript-specialist |
| Shader / 材质文件（.gdshader、VisualShader） | godot-shader-specialist |
| UI / 屏幕文件（Control 节点、CanvasLayer） | godot-specialist |
| 场景 / prefab / 关卡文件（.tscn、.tres） | godot-specialist |
| 原生扩展 / 插件文件（.gdextension、C++） | godot-gdextension-specialist |
| 通用架构审查 | godot-specialist |
```

**C#：**
```markdown
## Engine Specialists
- **主要**: godot-specialist
- **语言/代码专家**: godot-csharp-specialist（所有 .cs 文件）
- **Shader 专家**: godot-shader-specialist（.gdshader 文件，VisualShader 资源）
- **UI 专家**: godot-specialist（无专门 UI 专家 — 主要的涵盖所有 UI）
- **额外专家**: godot-gdextension-specialist（GDExtension / 仅原生 C++ 绑定）
- **路由备注**: 调用主要进行架构决策、ADR 验证和跨领域代码审查。调用 C# 专家进行代码质量、[Signal] 委托模式、[Export] 属性、.csproj 管理和 C# 特定的 Godot 习惯用法。调用 shader 专家进行材质设计和 shader 代码。仅在涉及原生 C++ 插件时调用 GDExtension 专家。

### 文件扩展名路由

| 文件扩展名 / 类型 | 要 Spawn 的专家 |
|-----------------------|---------------------|
| 游戏代码（.cs 文件） | godot-csharp-specialist |
| Shader / 材质文件（.gdshader、VisualShader） | godot-shader-specialist |
| UI / 屏幕文件（Control 节点、CanvasLayer） | godot-specialist |
| 场景 / prefab / 关卡文件（.tscn、.tres） | godot-specialist |
| 项目配置（.csproj、NuGet） | godot-csharp-specialist |
| 原生扩展 / 插件文件（.gdextension、C++） | godot-gdextension-specialist |
| 通用架构审查 | godot-specialist |
```

**两者 — GDScript + C#：**
```markdown
## Engine Specialists
- **主要**: godot-specialist
- **GDScript 专家**: godot-gdscript-specialist（.gd 文件 — 游戏/UI 脚本）
- **C# 专家**: godot-csharp-specialist（.cs 文件 — 性能关键系统）
- **Shader 专家**: godot-shader-specialist（.gdshader 文件，VisualShader 资源）
- **UI 专家**: godot-specialist（无专门 UI 专家 — 主要的涵盖所有 UI）
- **额外专家**: godot-gdextension-specialist（GDExtension / 仅原生 C++ 绑定）
- **路由备注**: 调用主要进行跨语言架构决策以及哪些系统属于哪种语言。调用 GDScript 专家处理 .gd 文件。调用 C# 专家处理 .cs 文件和 .csproj 管理。在边界处优先使用信号而不是直接的跨语言方法调用。

### 文件扩展名路由

| 文件扩展名 / 类型 | 要 Spawn 的专家 |
|-----------------------|---------------------|
| 游戏代码（.gd 文件） | godot-gdscript-specialist |
| 游戏代码（.cs 文件） | godot-csharp-specialist |
| 跨语言边界决策 | godot-specialist |
| Shader / 材质文件（.gdshader、VisualShader） | godot-shader-specialist |
| UI / 屏幕文件（Control 节点、CanvasLayer） | godot-specialist |
| 场景 / prefab / 关卡文件（.tscn、.tres） | godot-specialist |
| 项目配置（.csproj、NuGet） | godot-csharp-specialist |
| 原生扩展 / 插件文件（.gdextension、C++） | godot-gdextension-specialist |
| 通用架构审查 | godot-specialist |
```
