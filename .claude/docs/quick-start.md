# 游戏工作室 Agent 架构 — 快速入门指南

## 这是什么？

这是一套完整的 Claude Code Agent 架构，用于游戏开发。它将 49 个专业 AI Agent
组织成与真实游戏开发团队一致的工作室层级，明确了职责、委托
规则和协调协议。它包含 Godot、Unity 和 Unreal 的引擎专家 Agent —
每个都有主要引擎子系统的专属子专家。所有设计 Agent 和模板都扎根于
established 游戏设计理论（MDA Framework、Self-Determination Theory、
Flow State、Bartle Player Types）。使用与你的项目匹配的引擎集合。

## 如何使用

### 1. 理解层级结构

Agent 分为三个层级：

- **第一层（Opus）**：做高级决策的导演
  - `creative-director` -- 愿景和创意冲突解决
  - `technical-director` -- 架构和技术决策
  - `producer` -- 调度、协调和风险管理

- **第二层（Sonnet）**：拥有各自领域的主管
  - `game-designer`、`lead-programmer`、`art-director`、`audio-director`、
    `narrative-director`、`qa-lead`、`release-manager`、`localization-lead`

- **第三层（Sonnet/Haiku）**：在各自领域内执行的专家
  - 设计师、程序员、美术、作家、测试员、工程师

### 2. 为任务选择正确的 Agent

问自己："在真实工作室里哪个部门会处理这个？"

| 我需要... | 使用这个 Agent |
|-------------|---------------|
| 设计新机制 | `game-designer` |
| 编写战斗代码 | `gameplay-programmer` |
| 创建 shader | `technical-artist` |
| 写对话 | `writer` |
| 规划下一个 sprint | `producer` |
| 审查代码质量 | `lead-programmer` |
| 编写测试用例 | `qa-tester` |
| 设计关卡 | `level-designer` |
| 修复性能问题 | `performance-analyst` |
| 设置 CI/CD | `devops-engineer` |
| 设计战利品表 | `economy-designer` |
| 解决创意冲突 | `creative-director` |
| 做架构决策 | `technical-director` |
| 管理发布 | `release-manager` |
| 准备翻译字符串 | `localization-lead` |
| 快速测试机制想法 | `prototyper` |
| 代码安全审查 | `security-engineer` |
| 检查无障碍合规 | `accessibility-specialist` |
| 获取 Unreal Engine 建议 | `unreal-specialist` |
| 获取 Unity 建议 | `unity-specialist` |
| 获取 Godot 建议 | `godot-specialist` |
| 设计 GAS abilities/effects | `ue-gas-specialist` |
| 定义 BP/C++ 边界 | `ue-blueprint-specialist` |
| 实现 UE 复制 | `ue-replication-specialist` |
| 构建 UMG/CommonUI 小部件 | `ue-umg-specialist` |
| 设计 DOTS/ECS 架构 | `unity-dots-specialist` |
| 编写 Unity shaders/VFX | `unity-shader-specialist` |
| 管理 Addressable 资源 | `unity-addressables-specialist` |
| 构建 UI Toolkit/UGUI 屏幕 | `unity-ui-specialist` |
| 编写惯用 GDScript | `godot-gdscript-specialist` |
| 编写 Godot C# 代码 | `godot-csharp-specialist` |
| 创建 Godot shaders | `godot-shader-specialist` |
| 构建 GDExtension 模块 | `godot-gdextension-specialist` |
| 规划直播活动和赛季 | `live-ops-designer` |
| 为玩家写补丁说明 | `community-manager` |
| 头脑风暴新游戏想法 | 使用 `/brainstorm` 技能 |

### 3. 使用 Slash Commands 处理常见任务

| 命令 | 功能 |
|---------|-------------|
| `/start` | 首次入门 — 询问你处于哪个阶段，引导你到正确的工作流程 |
| `/help` | 上下文感知的"我下一步该做什么？" — 读取当前阶段并显示所需的下一个步骤 |
| `/project-stage-detect` | 完整项目审计 — 检测阶段、识别存在差距、推荐下一步 |
| `/setup-engine` | 配置引擎 + 版本、检测知识差距、填充版本感知参考文档 |
| `/adopt` | Brownfield 格式审计 — 检查现有 GDD/ADR/Story 的内部结构，生成迁移计划 |
| `/brainstorm` | 使用专业工作室方法引导构思（MDA、SDT、Bartle、动词优先） |
| `/map-systems` | 将游戏概念分解为系统，映射依赖，优先化设计顺序 |
| `/design-system` | 引导逐章节为单个游戏系统创作 GDD |
| `/quick-design` | 轻量级设计规格用于小更改 — 调优、调整、小添加 |
| `/review-all-gdds` | 跨 GDD 一致性和游戏设计整体性审查 |
| `/propagate-design-change` | 当 GDD 修订时，找到受影响的 ADR 并生成影响报告 |
| `/art-bible` | 引导式逐节 Art Bible 创作 — 在资产生产开始前创建视觉身份规格 |
| `/asset-spec` | 从 GDD 或角色档案生成每资产视觉规格和 AI 生成提示 |
| `/ux-design` | 引导逐章节创作 UX 规格（屏幕/流程、HUD 或模式库） |
| `/ux-review` | 验证 UX 规格是否符合 GDD 一致性、无障碍和模式合规 |
| `/create-architecture` | 引导创作主架构文档 |
| `/architecture-decision` | 创建一个 ADR |
| `/architecture-review` | 验证所有 ADR 的完整性、依赖排序和 GDD 覆盖率 |
| `/create-control-manifest` | 从 Accepted ADR 生成扁平化程序员规则表 |
| `/create-epics` | 将 GDD + ADR 翻译为 Epic（每个架构模块一个） |
| `/create-stories` | 将单个 Epic 分解为可实现的 Story 文件 |
| `/dev-story` | 读取 Story 并实现 — 路由到正确的程序员 Agent |
| `/sprint-plan` | 创建或更新 Sprint 计划 |
| `/sprint-status` | 快速 30 行 Sprint 快照（读取 sprint-status.yaml） |
| `/story-readiness` | 验证 Story 在领取前是否可实现（READY/NEEDS WORK/BLOCKED） |
| `/story-done` | 实现后的 8 阶段完成审查 — 验证验收标准，更新 Story 文件，显示下一个 Story |
| `/estimate` | 带复杂度、依赖和风险分解的结构化工作量估算 |
| `/design-review` | 审查游戏设计文档的完整性和一致性 |
| `/code-review` | 文件或更改集的架构代码审查 |
| `/balance-check` | 分析游戏平衡数据、公式和配置 — 标记异常值 |
| `/asset-audit` | 审计资源的命名约定、文件大小预算和管线合规 |
| `/content-audit` | GDD 指定的内容计数与已实现内容的审计 — 找到差距 |
| `/scope-check` | 分析功能或 Sprint 范围与原始计划的对比，标记范围蔓延 |
| `/perf-profile` | 带瓶颈识别的结构化性能分析 |
| `/tech-debt` | 扫描、追踪、优先化和报告技术债务 |
| `/gate-check` | 验证开发阶段之间前进的就绪状态（PASS/CONCERNS/FAIL） |
| `/consistency-check` | 扫描所有 GDD 与实体注册表，检测跨文档不一致（相互矛盾的属性、名称、规则） |
| `/security-audit` | 审计游戏的安全漏洞：存档篡改、作弊向量、网络利用、数据暴露 |
| `/reverse-document` | 从现有实现生成设计或架构文档 |
| `/milestone-review` | 审查里程碑进度并生成状态报告 |
| `/retrospective` | 运行结构化 Sprint 或里程碑回顾 |
| `/bug-report` | 创建结构化 Bug 报告 |
| `/playtest-report` | 生成结构化试玩报告或分析现有试玩笔记 |
| `/onboard` | 为新贡献者或 Agent 生成上下文入门文档 |
| `/release-checklist` | 为当前构建生成和验证预发布检查清单 |
| `/launch-checklist` | 跨所有部门的完整发布就绪验证 |
| `/changelog` | 从 git 提交和 Sprint 数据自动生成变更日志 |
| `/patch-notes` | 从 git 历史和内部数据生成玩家面向的补丁说明 |
| `/hotfix` | 带审计追踪的紧急修复，绕过正常 Sprint 流程 |
| `/day-one-patch` | 为金主后发现但发布前或发布时已知的 issues 准备聚焦的 day-one 补丁 |
| `/prototype` | 概念原型 — 在写 GDD 之前验证核心想法（阶段 1） |
| `/vertical-slice` | 生产级端到端构建 — 验证完整游戏循环（阶段 4） |
| `/localize` | 本地化工作流：字符串提取、验证、翻译就绪 |
| `/team-combat` | 编排完整战斗团队管线 |
| `/team-narrative` | 编排完整叙事团队管线 |
| `/team-ui` | 编排完整 UI 团队管线 |
| `/team-release` | 编排完整发布团队管线 |
| `/team-polish` | 编排完整打磨团队管线 |
| `/team-audio` | 编排完整音频团队管线 |
| `/team-level` | 编排完整关卡创建管线 |
| `/team-live-ops` | 编排实时运营团队进行赛季、活动和发布后内容 |
| `/team-qa` | 编排完整 QA 团队周期 — 测试计划、测试用例、冒烟检查、签字 |
| `/qa-plan` | 为 Sprint 或功能生成 QA 测试计划 |
| `/bug-triage` | 重新优先化开放 Bug，分配到 Sprint，暴露系统性趋势 |
| `/smoke-check` | 在 QA 交接前运行关键路径冒烟测试门（PASS/FAIL） |
| `/soak-test` | 为长时间游戏会话生成浸泡测试协议 |
| `/regression-suite` | 将覆盖率映射到 GDD 关键路径，标记缺少回归测试的已修复 Bug |
| `/test-setup` | 为项目引擎搭建测试框架 + CI 管线（运行一次） |
| `/test-helpers` | 为测试套件生成引擎特定测试辅助库和工厂函数 |
| `/test-flakiness` | 从 CI 运行历史检测 flaky 测试，标记隔离或修复 |
| `/test-evidence-review` | 测试文件和手动证据的质量审查 — ADEQUATE/INCOMPLETE/MISSING |
| `/skill-test` | 验证技能文件是否符合合规性和正确性（静态/规格/审计） |
| `/skill-improve` | 使用 test-fix-retest 循环改进技能 — 诊断、建议修复、重写、验证 |

### 4. 使用模板创建新文档

模板位于 `.claude/docs/templates/`：

- `game-design-document.md` -- 用于新机制和系统
- `architecture-decision-record.md` -- 用于技术决策
- `architecture-traceability.md` -- 将 GDD 需求映射到 ADR 到 Story ID
- `risk-register-entry.md` -- 用于新风险
- `narrative-character-sheet.md` -- 用于新角色
- `test-plan.md` -- 用于功能测试计划
- `sprint-plan.md` -- 用于 Sprint 规划
- `milestone-definition.md` -- 用于新里程碑
- `level-design-document.md` -- 用于新关卡
- `game-pillars.md` -- 用于核心设计支柱
- `art-bible.md` -- 用于视觉风格参考
- `technical-design-document.md` -- 用于每个系统的技术设计
- `post-mortem.md` -- 用于项目/里程碑回顾
- `sound-bible.md` -- 用于音频风格参考
- `release-checklist-template.md` -- 用于平台发布检查清单
- `changelog-template.md` -- 用于玩家面向的补丁说明
- `release-notes.md` -- 用于玩家面向的发布说明
- `incident-response.md` -- 用于实时事件响应手册
- `game-concept.md` -- 用于初始游戏概念（MDA、SDT、Flow、Bartle）
- `pitch-document.md` -- 用于向利益相关者推介游戏
- `economy-model.md` -- 用于虚拟经济设计（sink/faucet 模型）
- `faction-design.md` -- 用于派系身份、传说和游戏角色
- `systems-index.md` -- 用于系统分解和依赖映射
- `project-stage-report.md` -- 用于项目阶段检测输出
- `design-doc-from-implementation.md` -- 用于将现有代码反向文档化为 GDD
- `architecture-doc-from-code.md` -- 用于将代码反向文档化为架构文档
- `concept-doc-from-prototype.md` -- 用于将原型反向文档化为概念文档
- `ux-spec.md` -- 用于每屏幕 UX 规格（布局区域、状态、事件）
- `hud-design.md` -- 用于全游戏 HUD 哲学、区域和元素规格
- `accessibility-requirements.md` -- 用于项目范围无障碍层级和功能矩阵
- `interaction-pattern-library.md` -- 用于标准 UI 控件和游戏特定模式
- `player-journey.md` -- 用于 6 阶段情感弧线和按时间范围的留存钩子
- `difficulty-curve.md` -- 用于难度轴线、入门 ramp 和跨系统交互
- `test-evidence.md` -- 用于记录手动测试证据的模板（截图、演练笔记）

也在 `.claude/docs/templates/collaborative-protocols/`（由 Agent 使用，通常不直接编辑）：

- `design-agent-protocol.md` -- 设计 Agent 的提问-选项-草稿-审批周期
- `implementation-agent-protocol.md` -- 编程 Agent 从 Story 领取到 /story-done 的周期
- `leadership-agent-protocol.md` -- 导演层级 Agent 的跨部门委托和升级

### 5. 遵循协调规则

1. 工作沿层级向下流动：导演 -> 主管 -> 专家
2. 冲突沿层级向上升级
3. 跨部门工作由 `producer` 协调
4. Agent 未经委托不得修改其领域外的文件
5. 所有决策都有文档记录

## 新项目的第一步

**不知道从哪里开始？** 运行 `/start`。它询问你处于哪个阶段并引导你到正确的工作流程。
对你的游戏、引擎或经验水平不做任何假设。

如果你已经知道需要什么，直接跳到相关路径：

### 路径 A："我不知道要构建什么"

1. **运行 `/start`**（或 `/brainstorm open`）— 引导创意探索：
   什么让你兴奋、你玩过什么、你的限制
   - 生成 3 个概念，帮助你选择一个，定义核心循环和支柱
   - 生成游戏概念文档并推荐引擎
2. **设置引擎** — 运行 `/setup-engine`（使用头脑风暴建议）
   - 配置 CLAUDE.md，检测知识差距，填充参考文档
   - 创建 `.claude/docs/technical-preferences.md`，包含命名约定、
     性能预算和引擎特定默认值
   - 如果引擎版本比 LLM 训练数据更新，它会从网络获取
     当前文档，以便 Agent 提供正确的 API 建议
3. **验证概念** — 运行 `/design-review design/gdd/game-concept.md`
4. **分解为系统** — 运行 `/map-systems` 映射所有系统和依赖
5. **原型核心机制** — 运行 `/prototype [核心机制]`（1-3 天 — 在写 GDD 之前）
6. **设计每个系统** — 运行 `/design-system [system-name]` 按依赖顺序编写 GDD，
   结合原型发现
7. **规划第一个 Sprint** — 在架构和 `/vertical-slice` 之后，运行 `/sprint-plan new`
8. 开始构建

### 路径 B："我知道我要构建什么"

如果你已经有游戏概念和引擎选择：

1. **设置引擎** — 运行 `/setup-engine [engine] [version]`
   （例如 `/setup-engine godot 4.6`）— 也创建技术偏好
2. **编写游戏支柱** — 委托给 `creative-director`
3. **分解为系统** — 运行 `/map-systems` 列举系统和依赖
4. **设计每个系统** — 按依赖顺序为 GDD 运行 `/design-system [system-name]`
5. **创建初始 ADR** — 运行 `/architecture-decision`
6. **在 `production/milestones/` 中创建第一个里程碑**
7. **规划第一个 Sprint** — 运行 `/sprint-plan new`
8. 开始构建

### 路径 C："我知道游戏但不知道引擎"

如果你有概念但不知道哪个引擎适合：

1. **运行 `/setup-engine`** 不带参数 — 它会询问你游戏的需求
   （2D/3D、平台、团队规模、语言偏好）并根据你的回答推荐引擎
2. 从第 2 步开始跟随路径 B

### 路径 D："我有现有项目"

如果你已经有设计文档、原型或代码：

1. **运行 `/start`**（或 `/project-stage-detect`）— 分析现有内容，
   识别差距，并推荐下一步
2. **如果你有现有 GDD、ADR 或 Story，运行 `/adopt`** — 审计
   内部格式合规性并构建编号迁移计划以填补差距
   而不覆盖你现有工作
3. **如需要则配置引擎** — 如果尚未配置则运行 `/setup-engine`
4. **验证阶段就绪** — 运行 `/gate-check` 了解你所处位置
5. **规划下一个 Sprint** — 运行 `/sprint-plan new`

## 文件结构参考

```
CLAUDE.md                          -- 主配置（首先阅读这个，约 60 行）
.claude/
  settings.json                    -- Claude Code hooks 和项目设置
  agents/                          -- 49 个 Agent 定义（YAML frontmatter）
  skills/                          -- 73 个 Slash 命令定义（YAML frontmatter）
  hooks/                           -- 12 个 Hook 脚本（.sh），由 settings.json 连接
  rules/                           -- 11 个路径特定规则文件
  docs/
    quick-start.md                 -- 本文件
    technical-preferences.md       -- 项目特定标准（由 /setup-engine 填充）
    coding-standards.md            -- 编码和设计文档标准
    coordination-rules.md          -- Agent 协调规则
    context-management.md          -- 上下文预算和压缩说明
    directory-structure.md         -- 项目目录布局
    workflow-catalog.yaml          -- 7 阶段管线定义（由 /help 读取）
    setup-requirements.md          -- 系统前提条件（Git Bash、jq、Python）
    settings-local-template.md     -- 个人 settings.local.json 指南
    templates/                     -- 41 个文档模板
```
