# Agent 名册

以下 Agent 可用。每个在 `.claude/agents/` 中都有专属定义文件。
使用最适合当前任务的 Agent。当任务跨越多个领域时，
协调 Agent（通常是 `producer` 或领域主管）应委托给专家。

## 第一层 — 领导 Agent（Opus）
| Agent | 领域 | 使用场景 |
|-------|--------|-------------|
| `creative-director` | 高级愿景 | 主要创意决策、支柱冲突、基调/方向 |
| `technical-director` | 技术愿景 | 架构决策、技术栈选择、性能策略 |
| `producer` | 制作管理 | Sprint 规划、里程碑追踪、风险管理、协调 |

## 第二层 — 部门主管 Agent（Sonnet）
| Agent | 领域 | 使用场景 |
|-------|--------|-------------|
| `game-designer` | 游戏设计 | 机制、系统、进度、经济、平衡 |
| `lead-programmer` | 代码架构 | 系统设计、代码审查、API 设计、重构 |
| `art-director` | 视觉方向 | 风格指南、美术圣经、资源标准、UI/UX 方向 |
| `audio-director` | 音频方向 | 音乐方向、音效调色板、音频实现策略 |
| `narrative-director` | 故事与写作 | 故事弧线、世界构建、角色设计、对话策略 |
| `qa-lead` | 质量保证 | 测试策略、Bug 分类、发布就绪、回归规划 |
| `release-manager` | 发布管线 | 构建管理、版本控制、变更日志、部署、回滚 |
| `localization-lead` | 国际化 | 字符串外部化、翻译管线、区域测试 |

## 第三层 — 专家 Agent（Sonnet 或 Haiku）
| Agent | 领域 | 模型 | 使用场景 |
|-------|--------|-------|-------------|
| `systems-designer` | 系统设计 | Sonnet | 特定机制实现、公式设计、循环 |
| `level-designer` | 关卡设计 | Sonnet | 关卡布局、节奏、遭遇设计、流程 |
| `economy-designer` | 经济/平衡 | Sonnet | 资源经济、战利品表、进度曲线 |
| `gameplay-programmer` | 游戏玩法代码 | Sonnet | 功能实现、游戏系统代码 |
| `engine-programmer` | 引擎系统 | Sonnet | 核心引擎、渲染、物理、内存管理 |
| `ai-programmer` | AI 系统 | Sonnet | 行为树、寻路、NPC 逻辑、状态机 |
| `network-programmer` | 网络 | Sonnet | 网络代码、复制、延迟补偿、匹配 |
| `tools-programmer` | 开发工具 | Sonnet | 编辑器扩展、管线工具、调试实用程序 |
| `ui-programmer` | UI 实现 | Sonnet | UI 框架、屏幕、小部件、数据绑定 |
| `technical-artist` | 技术美术 | Sonnet | Shader、VFX、优化、美术管线工具 |
| `sound-designer` | 音效设计 | Sonnet | SFX 设计文档、音频事件列表、混音备注 |
| `writer` | 对话/传说 | Sonnet | 对话写作、传说条目、物品描述 |
| `world-builder` | 世界/传说设计 | Sonnet | 世界规则、派系设计、历史、地理 |
| `qa-tester` | 测试执行 | Haiku | 编写测试用例、Bug 报告、测试检查清单 |
| `performance-analyst` | 性能 | Sonnet | 分析、性能建议、内存分析 |
| `devops-engineer` | 构建/部署 | Haiku | CI/CD、构建脚本、版本控制工作流 |
| `analytics-engineer` | 遥测 | Sonnet | 事件追踪、仪表板、A/B 测试设计 |
| `ux-designer` | UX 流程 | Sonnet | 用户流程、线框图、无障碍、输入处理 |
| `prototyper` | 快速原型 | Sonnet | 一次性原型、机制测试、可行性验证 |
| `security-engineer` | 安全 | Sonnet | 反作弊、漏洞预防、存档加密、网络安全 |
| `accessibility-specialist` | 无障碍 | Haiku | WCAG 合规、色盲模式、键位重映射、文本缩放 |
| `live-ops-designer` | 实时运营 | Sonnet | 赛季、活动、战斗通行证、留存、实时经济 |
| `community-manager` | 社区 | Haiku | 补丁说明、玩家反馈、危机沟通、社区健康 |

## 引擎特定 Agent（使用与你的引擎匹配的集合）

### 引擎主管

| Agent | 引擎 | 模型 | 使用场景 |
| ---- | ---- | ---- | ---- |
| `unreal-specialist` | Unreal Engine 5 | Sonnet | Blueprint vs C++、GAS 概览、UE 子系统、Unreal 优化 |
| `unity-specialist` | Unity | Sonnet | MonoBehaviour vs DOTS、Addressables、URP/HDRP、Unity 优化 |
| `godot-specialist` | Godot 4 | Sonnet | GDScript 模式、node/scene 架构、signals、Godot 优化 |

### Unreal Engine 子专家

| Agent | 子系统 | 模型 | 使用场景 |
| ---- | ---- | ---- | ---- |
| `ue-gas-specialist` | Gameplay Ability System | Sonnet | Abilities、gameplay effects、attribute sets、tags、prediction |
| `ue-blueprint-specialist` | Blueprint 架构 | Sonnet | BP/C++ 边界、graph 标准、命名、BP 优化 |
| `ue-replication-specialist` | 网络/复制 | Sonnet | Property replication、RPCs、prediction、relevancy、bandwidth |
| `ue-umg-specialist` | UMG/CommonUI | Sonnet | Widget 层级、数据绑定、CommonUI 输入、UI 性能 |

### Unity 子专家

| Agent | 子系统 | 模型 | 使用场景 |
| ---- | ---- | ---- | ---- |
| `unity-dots-specialist` | DOTS/ECS | Sonnet | Entity Component System、Jobs、Burst compiler、hybrid renderer |
| `unity-shader-specialist` | Shaders/VFX | Sonnet | Shader Graph、VFX Graph、URP/HDRP 定制、后处理 |
| `unity-addressables-specialist` | 资源管理 | Sonnet | Addressable groups、async loading、memory、content delivery |
| `unity-ui-specialist` | UI Toolkit/UGUI | Sonnet | UI Toolkit、UXML/USS、UGUI Canvas、数据绑定、跨平台输入 |

### Godot 子专家

| Agent | 子系统 | 模型 | 使用场景 |
| ---- | ---- | ---- | ---- |
| `godot-gdscript-specialist` | GDScript | Sonnet | 静态类型、设计模式、signals、协程、GDScript 性能 |
| `godot-csharp-specialist` | C# / .NET | Sonnet | .NET 模式、[Signal] 委托、async、可空类型、类型安全节点访问 |
| `godot-shader-specialist` | Shaders/渲染 | Sonnet | Godot shading language、visual shaders、particles、后处理 |
| `godot-gdextension-specialist` | GDExtension | Sonnet | C++/Rust 绑定、原生性能、custom nodes、构建系统 |
