# Claude Code Game Studios — 完整工作流程指南

> **如何使用 Agent 架构从零到发布游戏。**
>
> 本指南引导你完成使用 48-Agent 系统、68 个 Slash 命令和 12 个自动化 Hook
> 的游戏开发的每个阶段。假设你已安装 Claude Code 并从项目根目录工作。
>
> 管线有 7 个阶段。每个阶段有一个正式门控（`/gate-check`），
> 必须通过才能进入下一阶段。权威阶段序列定义在
> `.claude/docs/workflow-catalog.yaml` 中，由 `/help` 读取。

---

## 目录

1. [快速开始](#快速开始)
2. [阶段 1：概念](#阶段-1-概念)
3. [阶段 2：系统设计](#阶段-2-系统设计)
4. [阶段 3：技术设置](#阶段-3-技术设置)
5. [阶段 4：预生产](#阶段-4-预生产)
6. [阶段 5：生产](#阶段-5-生产)
7. [阶段 6：打磨](#阶段-6-打磨)
8. [阶段 7：发布](#阶段-7-发布)
9. [跨领域关注点](#跨领域关注点)
10. [附录 A：Agent 快速参考](#附录-a-agent-快速参考)
11. [附录 B：Slash 命令快速参考](#附录-b-slash-命令快速参考)
12. [附录 C：常见工作流程](#附录-c-常见工作流程)

---

## 快速开始

### 你需要什么

在开始之前，确保你有：

- **Claude Code** 已安装并工作
- **Git** 配合 Git Bash（Windows）或标准终端（Mac/Linux）
- **jq**（推荐但可选 — 如果缺失 hooks 回退到 `grep`）
- **Python 3**（可选 — 一些 hooks 使用它进行 JSON 验证）

### 步骤 1：克隆并打开

```bash
git clone <repo-url> my-game
cd my-game
```

### 步骤 2：运行 /start

如果这是你的第一次会话：

```
/start
```

此引导式入门询问你处于哪个阶段并引导你到正确的阶段：

- **路径 A** -- 还没有想法：路由到 `/brainstorm`
- **路径 B** -- 模糊想法：带种子路由到 `/brainstorm`
- **路径 C** -- 清晰概念：路由到 `/setup-engine` 和 `/map-systems`
- **路径 D1** -- 现有项目，Few artifacts：正常流程
- **路径 D2** -- 现有项目，有 GDD/ADR：运行 `/project-stage-detect`
  然后 `/adopt` 用于 brownfield 迁移

### 步骤 3：验证 Hooks 正在工作

启动一个新的 Claude Code 会话。你应该看到 `session-start.sh` hook 的输出：

```
=== Claude Code Game Studios -- Session Context ===
Branch: main
Recent commits:
  abc1234 Initial commit
===================================
```

如果你看到这个，hooks 正在工作。如果没有，检查 `.claude/settings.json`
确保 hook 路径对你的 OS 正确。

### 步骤 4：随时请求帮助

在任何时候，运行：

```
/help
```

这从 `production/stage.txt` 读取你当前的阶段，检查存在哪些产物，
并准确告诉你下一步做什么。它区分 REQUIRED 下一个步骤和 OPTIONAL 机会。

### 步骤 5：创建你的目录结构

目录按需创建。系统期望此布局：

```
src/                  # 游戏源代码
  core/               # 引擎/框架代码
  gameplay/           # 游戏玩法系统
  ai/                 # AI 系统
  networking/         # 多人游戏代码
  ui/                 # UI 代码
  tools/              # 开发工具
assets/               # 游戏资源
  art/                # 精灵、模型、纹理
  audio/              # 音乐、音效
  vfx/                # 粒子效果
  shaders/            # Shader 文件
  data/               # JSON 配置/平衡数据
design/               # 设计文档
  gdd/                # 游戏设计文档
  narrative/          # 故事、传说、对话
  levels/             # 关卡设计文档
  balance/            # 平衡电子表格和数据
  ux/                 # UX 规格
docs/                 # 技术文档
  architecture/       # 架构决策记录
  api/                # API 文档
  postmortems/        # 复盘
tests/                # 测试套件
prototypes/           # 一次性原型
production/           # Sprint 计划、里程碑、发布
  sprints/
  milestones/
  releases/
  epics/              # Epic 和 Story 文件（来自 /create-epics + /create-stories）
  playtests/          # 试玩报告
  session-state/      # 临时会话状态（gitignored）
  session-logs/       # 会话审计追踪（gitignored）
```

> **提示：** 第一天你不需要所有这些。在需要该阶段的目录时创建它们。
> 重要的是在创建时遵循此结构，因为 **规则系统** 基于文件路径强制执行标准。
> `src/gameplay/` 中的代码获得 gameplay 规则，`src/ai/` 中的代码获得 AI 规则，以此类推。

---

## 阶段 1：概念

### 这个阶段发生什么

你从"没有想法"或"模糊想法"到具有定义支柱和玩家旅程的结构化游戏概念文档。
这是你弄清楚**什么**你在建造和**为什么**的地方。

### 阶段 1 管线

```
/brainstorm  -->  game-concept.md  -->  /design-review  -->  /setup-engine
     |                                        |                    |
     v                                        v                    v
  10 个概念     概念文档包含           概念文档           引擎固定在
  MDA 分析    支柱、MDA、            验证           technical-preferences.md
  玩家动机    核心循环、USP                                |
                                                            v
                                                      /map-systems
                                                            |
                                                            v
                                                   systems-index.md
                                                   （所有系统、依赖、
                                                    优先级层级）
```

### 步骤 1.1：用 /brainstorm 头脑风暴

这是你的起点。运行 brainstorm 技能：

```
/brainstorm
```

或带类型提示：

```
/brainstorm roguelike deckbuilder
```

**发生什么：** brainstorm 技能使用专业工作室技术引导你完成协作 6 阶段构思过程：

1. 询问你的兴趣、主题和约束
2. 生成 10 个概念种子及 MDA（力学、动态、美学）分析
3. 你选择 2-3 个最喜欢的进行深入分析
4. 执行玩家动机映射和受众定位
5. 你选择获胜概念
6. 将其正式化为 `design/gdd/game-concept.md`

概念文档包括：

- 电梯演讲（一句话）
- 核心幻想（玩家想象自己做什么）
- MDA 分解
- 目标受众（Bartle 类型、人口统计）
- 核心循环图
- 独特销售主张
- 可比标题和差异化
- 游戏支柱（3-5 个不可谈判的设计价值）
- 反支柱（游戏故意避免的事情）

### 步骤 1.2：审查概念（可选但推荐）

```
/design-review design/gdd/game-concept.md
```

在继续之前验证结构和完整性。

### 步骤 1.3：选择你的引擎

```
/setup-engine
```

或带特定引擎：

```
/setup-engine godot 4.6
```

**/setup-engine 做什么：**

- 用命名约定、性能预算和引擎特定默认值填充 `.claude/docs/technical-preferences.md`
- 检测知识差距（引擎版本比 LLM 训练数据新）并建议交叉引用 `docs/engine-reference/`
- 在 `docs/engine-reference/` 创建版本固定参考文档

**为什么这很重要：** 一旦你设置了引擎，系统知道使用哪个引擎专家 Agent。
如果你选择 Godot，像 `godot-specialist`、`godot-gdscript-specialist`
和 `godot-shader-specialist` 成为你的首选专家。

### 步骤 1.4：将你的概念分解为系统

在编写单个 GDD 之前，列举你的游戏需要的所有系统：

```
/map-systems
```

这创建 `design/gdd/systems-index.md` — 一个主跟踪文档：

- 列出你的游戏需要的每个系统（战斗、移动、UI 等）
- 映射系统之间的依赖
- 分配优先级层级（MVP、Vertical Slice、Alpha、Full Vision）
- 确定设计顺序（Foundation > Core > Feature > Presentation > Polish）

此步骤在进入阶段 2 之前**必需**。来自 155 个游戏复盘的研究证实
跳过系统列举会在生产中损失 5-10 倍的成本。

### 阶段 1 门控

```
/gate-check concept
```

**通过要求：**

- `technical-preferences.md` 中配置了引擎
- `design/gdd/game-concept.md` 存在且有支柱
- `design/gdd/systems-index.md` 存在且有依赖排序

**裁决：** PASS / CONCERNS / FAIL。CONCERNS 可带有承认的风险通过。FAIL 阻止进展。

---

## 阶段 2：系统设计

### 这个阶段发生什么

你创建定义游戏如何工作的所有设计文档。还没有编码 — 这是纯设计。
在系统索引中识别的每个系统都有自己的 GDD，逐节创作，逐个审查，
然后所有 GDD 交叉检查一致性。

### 阶段 2 管线

```
/map-systems next  -->  /design-system  -->  /design-review
       |                     |                     |
       v                     v                     v
  从 systems-index   逐节 GDD 创作       验证 8 个
  选择下一个系统     （增量写入）      必需章节
                       APPROVED/NEEDS REVISION
       |
       |  （对每个 MVP 系统重复）
       v
/review-all-gdds
       |
       v
  跨 GDD 一致性 + 设计理论审查
  PASS / CONCERNS / FAIL
```

### 步骤 2.1：创作系统 GDD

使用引导工作流程按依赖顺序设计每个系统：

```
/map-systems next
```

这选择最高优先级未设计的系统并交给 `/design-system`，
引导你逐节创建其 GDD。

你也可以直接设计特定系统：

```
/design-system combat-system
```

**/design-system 做什么：**

1. 读取你的游戏概念、系统索引和任何上游/下游 GDD
2. 运行技术可行性预检查（领域映射 + 可行性简报）
3. 逐节引导你完成 8 个必需的 GDD 章节
4. 每个章节遵循：上下文 > 问题 > 选项 > 决策 > 草稿 > 批准 > 写入
5. 每个章节获批后立即写入文件（崩溃存活）
6. 标记与现有已批准 GDD 的冲突
7. 按类别路由到专家 Agent（数学的 systems-designer、经济的 economy-designer、故事系统的 narrative-director）

**8 个必需的 GDD 章节：**

| # | 章节 | 这里放什么 |
|---|---------|---------------|
| 1 | **概览** | 系统的一段落摘要 |
| 2 | **玩家幻想** | 玩家使用此系统时想象/感受什么 |
| 3 | **详细规则** | 明确的机械规则 |
| 4 | **公式** | 每个计算，变量定义和范围 |
| 5 | **边缘情况** | 奇怪情况下发生什么？明确解决。 |
| 6 | **依赖** | 此系统连接到的其他系统（双向） |
| 7 | **调优旋钮** | 设计师可以安全更改哪些值，带安全范围 |
| 8 | **验收标准** | 你如何测试它有效？具体、可衡量。 |

加上一个 **游戏手感** 章节：手感参考、输入响应（ms/帧）、
动画手感目标（启动/活动/恢复）、冲击时刻、重量轮廓。

### 步骤 2.2：审查每个 GDD

在下一个系统开始之前，验证当前系统：

```
/design-review design/gdd/combat-system.md
```

检查全部 8 个章节的完整性、公式清晰度、边缘情况解决、
双向依赖和可测试的验收标准。

**裁决：** APPROVED / NEEDS REVISION / MAJOR REVISION。只有 APPROVED GDD 应该继续。

### 步骤 2.3：无需完整 GDD 的小更改

对于不值得完整 GDD 的调优更改、小添加或调整：

```
/quick-design "添加侧翼攻击 10% 伤害加成"
```

这在 `design/quick-specs/` 中创建轻量级规格，而不是完整 8 章节 GDD。
用于调优、数字更改和小添加。

### 步骤 2.4：跨 GDD 一致性审查

在所有 MVP 系统 GDD 单独获批后：

```
/review-all-gdds
```

这同时读取所有 GDD 并运行两个分析阶段：

**阶段 1 -- 跨 GDD 一致性：**
- 依赖双向性（A 引用 B，B 是否引用 A？）
- 系统间的规则矛盾
- 对已重命名或删除系统的过时引用
- 所有权冲突（两个系统声称相同职责）
- 公式范围兼容性（系统 A 的输出是否适合系统 B 的输入？）
- 验收标准交叉检查

**阶段 2 -- 设计理论（游戏设计整体性）：**
- 竞争进度循环（两个系统争夺相同的奖励空间？）
- 认知负荷（一次超过 4 个活动系统？）
- 主导策略（一种方法使所有其他方法无关？）
- 经济循环分析（源和汇平衡？）
- 系统间的难度曲线一致性
- 支柱对齐和反支柱违规
- 玩家幻想一致性

**输出：** 带裁决的 `design/gdd/gdd-cross-review-[date].md`。

### 步骤 2.5：叙事设计（如适用）

如果你的游戏有故事、传说或对话，这是你构建它的时候：

1. **世界构建** — 使用 `world-builder` 定义派系、历史、地理和世界规则
2. **故事结构** — 使用 `narrative-director` 设计故事弧线、角色弧线和叙事节拍
3. **角色表** — 使用 `narrative-character-sheet.md` 模板

### 阶段 2 门控

```
/gate-check systems-design
```

**通过要求：**

- `systems-index.md` 中所有 MVP 系统有 `Status: Approved`
- 每个 MVP 系统有已审查的 GDD
- 跨 GDD 审查报告存在（`design/gdd/gdd-cross-review-*.md`）且裁决为 PASS 或 CONCERNS（不是 FAIL）

---

## 阶段 3：技术设置

### 这个阶段发生什么

你做出关键技术决策，将它们记录为架构决策记录（ADR），
通过审查验证，并生成给程序员扁平、可操作规则的 control manifest。
你还建立 UX 基础。

### 阶段 3 管线

```
/create-architecture  -->  /architecture-decision (x N)  -->  /architecture-review
        |                          |                                   |
        v                          v                                   v
  主架构文档          每决策 ADR              验证完整性、
  覆盖所有系统       在 docs/architecture/   依赖排序、
                    adr-*.md                引擎兼容性
                                                                      |
                                                                      v
                                                         /create-control-manifest
                                                                      |
                                                                      v
                                                         扁平程序员规则
                                                         docs/architecture/
                                                         control-manifest.md
        本阶段还有：
        -------------------
        /ux-design  -->  /ux-review
        无障碍要求文档
        交互模式库
```

### 步骤 3.1：主架构文档

```
/create-architecture
```

在 `docs/architecture/architecture.md` 创建覆盖系统边界、数据流和集成点的总体架构文档。

### 步骤 3.2：架构决策记录（ADR）

对于每个重大技术决策：

```
/architecture-decision "NPC AI 的状态机 vs 行为树"
```

**发生什么：** 技能引导你创建包含以下内容的 ADR：
- 上下文和决策驱动因素
- 带优缺点和引擎兼容性的所有选项
- 选择的选项及理由
- 后果（正面、负面、风险）
- 依赖（取决于、启用、阻塞、排序说明）
- GDD 需求地址（通过 TR-ID 链接）

ADR 经历生命周期：Proposed > Accepted > Superseded/Deprecated。

**门控检查前至少需要 3 个 Foundation 层 ADR。**

**改造现有 ADR：** 如果你有来自 brownfield 项目的现有 ADR：

```
/architecture-decision retrofit docs/architecture/adr-005.md
```

这检测哪些模板章节缺失，仅添加那些，永不覆盖现有内容。

### 步骤 3.3：架构审查

```
/architecture-review
```

一起验证所有 ADR：
- ADR 依赖的拓扑排序（检测循环）
- 引擎兼容性验证
- GDD 修订标志（根据 ADR 选择标记需要更新的 GDD 章节）
- TR-ID 注册表维护（`docs/architecture/tr-registry.yaml`）

### 步骤 3.4：Control Manifest

```
/create-control-manifest
```

从所有 Accepted ADR 生成扁平程序员规则表：

```
docs/architecture/control-manifest.md
```

这包含按代码层组织的 Required 模式、Forbidden 模式和 Guardrails。
后来创建的故事嵌入 manifest 版本日期，以便检测过时。

### 步骤 3.5：无障碍要求

使用模板创建 `design/accessibility-requirements.md`。承诺一个层级
（Basic / Standard / Comprehensive / Exemplary）并填写 4 轴特征矩阵
（视觉、运动、认知、听觉）。

此文档在阶段 3 是必需的，因为 UX 规格（在阶段 4 编写）引用此层级 —
它是设计前提条件，不是 UX 交付物。

### 阶段 3 门控

```
/gate-check technical-setup
```

**通过要求：**

- `docs/architecture/architecture.md` 存在
- 至少 3 个 ADR 存在且为 Accepted
- 架构审查报告存在
- `docs/architecture/control-manifest.md` 存在
- `design/accessibility-requirements.md` 存在

---

## 阶段 4：预生产

### 这个阶段发生什么

你为关键屏幕创建 UX 规格，原型化有风险的机制，
将设计文档转换为可实现的 story，计划你的第一个 sprint，
并构建证明核心循环有趣的 Vertical Slice。

### 阶段 4 管线

```
/ux-design  -->  /prototype  -->  /create-epics  -->  /create-stories  -->  /sprint-plan
    |                |                  |                   |                       |
    v                v                  v                   v                       v
  UX 规格       一次性原型          production/         production/         第一个 sprint，
  design/ux/    在 prototypes/     epics/*/EPIC.md     epics/*/story-*.md   优先级 story
                隔离工作树          （每个模块一个）    （每个行为一个）     production/sprints/
                                                                              sprint-*.md
    |                                                      |
    v                                                      v
 /ux-review                                            /story-readiness
 （验证规格                                             （验证每个 story
  在 epics 前）                                          在领取前）
                                                           |
                                                           v
                                                       /dev-story
                                                     （实现 story，
                                                      路由到正确的 agent）
                         |
                         v
                   Vertical Slice
                   （可玩构建，
                    3 次无人引导会话）
```

### 步骤 4.1：关键屏幕的 UX 规格

在编写 epics 之前，创建 UX 规格以便 story 作者知道存在哪些屏幕
以及他们必须支持哪些玩家交互。

**UX 规格：**

```
/ux-design main-menu
/ux-design core-gameplay-hud
```

三种模式：屏幕/流程、HUD 和交互模式。输出到 `design/ux/`。
每个规格包括：玩家需求、布局区域、状态、交互映射、数据需求、触发的事件、无障碍、本地化。

读取你在阶段 3 编写的 `accessibility-requirements.md`
和你在 `technical-preferences.md` 中的输入方法配置以驱动无障碍和输入覆盖检查 —
每个屏幕不需要重新指定。

> **提示：** `/design-system` 为每个有 UI 要求的系统发出 📌 UX Flag。
> 使用那些 flags 作为检查清单来确定哪些屏幕需要规格。

**交互模式库：**

```
/ux-design interaction-patterns
```

创建 `design/ux/interaction-patterns.md` — 16 个标准控件加上
游戏特定模式（库存槽、能力图标、HUD 条、对话框等）
及动画和声音标准。

**UX 审查：**

```
/ux-review all
```

验证 UX 规格的 GDD 一致性和无障碍层级合规性。
产生 APPROVED / NEEDS REVISION / MAJOR REVISION NEEDD 裁决。

### 步骤 4.2：原型化有风险的机制

不是所有东西都需要原型。当以下情况时原型：
- 一个机制是新颖的你不确定它是否有趣
- 一个技术方法有风险你不确定它是否可行
- 两个设计选项都看起来可行你需要感受差异

```
/prototype "带动量的抓钩移动"
```

**发生什么：** 技能与你协作定义假设、成功标准和最小范围。
`prototyper` Agent 在隔离的 git worktree（`isolation: worktree`）中工作，
以便一次性代码永远不会污染 `src/`。

**关键规则：** `prototype-code` 规则故意放宽编码标准 —
硬编码值可以，没有测试要求 — 但带有假设和发现的 README 是强制的。

### 步骤 4.3：从设计产物创建 Epics 和 Stories

```
/create-epics layer: foundation
/create-stories [epic-slug]   # 对每个 epic 重复
/create-epics layer: core
/create-stories [epic-slug]   # 对每个 core epic 重复
```

`/create-epics` 读取你的 GDD、ADR 和架构来定义 epic 范围 —
每个架构模块一个 epic。然后 `/create-stories` 将每个 epic 分解为
`production/epics/[slug]/` 中的可实现 story 文件。每个 story 嵌入：
- GDD 需求引用（TR-ID，不是引用文本 — 保持新鲜）
- ADR 引用（仅来自 Accepted ADR；Proposed ADR 导致 `Status: Blocked`）
- Control manifest 版本日期（用于过时检测）
- 引擎特定实现说明
- 来自 GDD 的验收标准

一旦 stories 存在，运行 `/dev-story [story-path]` 实现一个 — 它自动路由到正确的程序员 Agent。

### 步骤 4.4：在领取前验证 Stories

```
/story-readiness production/stories/combat-damage-calc.md
```

检查：设计完整性、架构覆盖、范围清晰度、定义完成。
裁决：READY / NEEDS WORK / BLOCKED。

### 步骤 4.5：工作量估算

```
/estimate production/stories/combat-damage-calc.md
```

提供带风险评估的工作量估算。

### 步骤 4.6：计划你的第一个 Sprint

```
/sprint-plan new
```

**发生什么：** `producer` Agent 协作进行 sprint 规划：
- 询问 sprint 目标和可用时间
- 将目标分解为 Must Have / Should Have / Nice to Have 任务
- 识别风险和阻塞
- 创建 `production/sprints/sprint-01.md`
- 填充 `production/sprint-status.yaml`（机器可读 story 追踪）

### 步骤 4.7：Vertical Slice（硬门控）

在进入生产之前，你必须构建和试玩 Vertical Slice：

- 一个完整的端到端核心循环，从头到尾可玩
- 代表性质量（不是占位符一切）
- 至少 3 次会话无人引导试玩
- 写试玩报告（`/playtest-report`）

这是一个 **硬门控** — 如果人类没有无人引导地玩过构建，`/gate-check` 会自动 FAIL。

### 阶段 4 门控

```
/gate-check pre-production
```

**通过要求：**

- `design/ux/` 中至少 1 个 UX 规格已审查
- UX 审查完成（APPROVED 或 NEEDS REVISION 且有文档化风险）
- 至少 1 个带 README 的原型
- `production/stories/` 中存在 story 文件
- 至少 1 个 sprint 计划存在
- 至少 1 个试玩报告存在（Vertical Slice 在 3+ 次会话中玩过）

---

## 阶段 5：生产

### 这个阶段发生什么

这是核心生产循环。你在 sprints 中工作（通常 1-2 周），
逐个 story 实现特性，追踪进度，并通过结构化完成审查关闭 stories。
此阶段重复直到你的游戏内容完整。

### 阶段 5 管线（每 Sprint）

```
/sprint-plan new  -->  /story-readiness  -->  implement  -->  /story-done
       |                     |                    |                |
       v                     v                    v                v
  Sprint 创建         Story 验证          代码编写         8 阶段审查：
  sprint-status.yaml  READY 裁决          测试通过         验证标准，
  填充                                       检查偏差，
                                                更新 story 状态
       |
       |  （对每个 story 重复直到 sprint 完成）
       v
  /sprint-status （随时快速 30 行快照）
  /scope-check    （如果范围增长）
  /retrospective  （在 sprint 结束时）
```

### 步骤 5.1：Story 生命周期

生产阶段以 **story 生命周期** 为中心：

```
/story-readiness  -->  implement  -->  /story-done  -->  next story
```

**1. Story 就绪：** 在领取 story 前验证它：

```
/story-readiness production/stories/combat-damage-calc.md
```

这检查设计完整性、架构覆盖、ADR 状态（如果 ADR 仍是 Proposed 则阻塞）、
control manifest 版本（如果过时则警告）和范围清晰度。
裁决：READY / NEEDS WORK / BLOCKED。

**2. 实现：** 与适当的 agent 一起工作：

- `gameplay-programmer` 用于游戏系统
- `engine-programmer` 用于核心引擎工作
- `ai-programmer` 用于 AI 行为
- `network-programmer` 用于多人游戏
- `ui-programmer` 用于 UI 代码
- `tools-programmer` 用于开发工具

所有 agent 遵循协作协议：它们读取设计文档，提问澄清，
呈现架构选项，获得你的批准，然后实现。

**3. Story 完成：** 当一个 story 完成时：

```
/story-done production/stories/combat-damage-calc.md
```

这运行 8 阶段完成审查：
1. 找到并读取 story 文件
2. 加载引用的 GDD、ADR 和 control manifest
3. 验证验收标准（自动检查、手动、推迟）
4. 检查 GDD/ADR 偏差（BLOCKING / ADVISORY / OUT OF SCOPE）
5. 提示代码审查
6. 生成完成报告（COMPLETE / COMPLETE WITH NOTES / BLOCKED）
7. 用完成笔记更新 story `Status: Complete`
8. 显示下一个就绪 story

在审查期间发现的技术债务记录到 `docs/tech-debt-register.md`。

### 步骤 5.2：Sprint 追踪

随时检查进度：

```
/sprint-status
```

从 `production/sprint-status.yaml` 读取快速 30 行快照。

如果范围增长：

```
/scope-check production/sprints/sprint-03.md
```

这将当前范围与原始计划比较并标记范围增加，建议削减。

### 步骤 5.3：内容追踪

```
/content-audit
```

将 GDD 指定的内容与已实现的内容进行比较。尽早捕获内容差距。

### 步骤 5.4：设计变更传播

当在 story 创建后 GDD 更改时：

```
/propagate-design-change design/gdd/combat-system.md
```

Git-diffs GDD，找到受影响的 ADR，生成影响报告，
并引导你完成 Superseded/update/keep 决策。

### 步骤 5.5：多系统特性（团队编排）

对于跨越多个领域的特性，使用团队技能：

```
/team-combat "带 HoT 和 clean 的治疗能力"
/team-narrative "第 2 幕故事内容"
/team-ui "库存屏幕重新设计"
/team-level "森林地下城关卡"
/team-audio "战斗音频通道"
```

每个团队技能协调 6 阶段协作工作流程：
1. **设计** — game-designer 提问，呈现选项
2. **架构** — lead-programmer 提议代码结构
3. **并行实现** — specialists 同时工作
4. **集成** — gameplay-programmer 将一切连接在一起
5. **验证** — qa-tester 根据验收标准运行
6. **报告** — 协调器总结状态

编排是自动化的，但 **决策点留在你手中**。

### 步骤 5.6：Sprint 审查和下一个 Sprint

在一个 sprint 结束时：

```
/retrospective
```

分析计划 vs 完成、速度、阻塞和可操作的改进。

然后计划下一个 sprint：

```
/sprint-plan new
```

### 步骤 5.7：里程碑审查

在里程碑检查点：

```
/milestone-review "alpha"
```

产生特性完整性、质量指标、风险评估和 go/no-go 推荐。

### 阶段 5 门控

```
/gate-check production
```

**通过要求：**

- 所有 MVP stories 完成
- 试玩：3 次会话覆盖新玩家、中期游戏和难度曲线
- 乐趣假设已验证
- 试玩数据中没有困惑循环

---

## 阶段 6：打磨

### 这个阶段发生什么

你的游戏特性完整。现在你让它变得更好。此阶段专注于性能、平衡、
无障碍、音频、视觉打磨和试玩。

### 阶段 6 管线

```
/perf-profile  -->  /balance-check  -->  /asset-audit  -->  /playtest-report (x3)
       |                  |                    |                    |
       v                  v                    v                    v
  分析 CPU/GPU      分析公式          验证命名、        覆盖：新玩家、
  内存、优化        和数据查找        格式、尺寸        中期游戏、难度
  瓶颈              破损进度                               曲线

  /tech-debt  -->  /team-polish
       |                |
       v                v
  追踪和          协调通过：
  优先化         性能 + 美术 +
  债务项目       音频 + UX + QA
```

### 步骤 6.1：性能分析

```
/perf-profile
```

引导你完成结构化性能分析：
- 建立目标（FPS、内存、平台）
- 按影响排名识别瓶颈
- 生成带代码位置和预期增益的可操作优化任务

### 步骤 6.2：平衡分析

```
/balance-check assets/data/combat_damage.json
```

分析平衡数据以查找统计异常值、破损的进度曲线、
退化策略和经济不平衡。

### 步骤 6.3：资源审计

```
/asset-audit
```

跨所有资源验证命名约定、文件格式标准和大小预算。

### 步骤 6.4：试玩（必需：3 次会话）

```
/playtest-report
```

生成结构化试玩报告。需要三次会话，涵盖：
- 新玩家体验
- 中期游戏系统
- 难度曲线

### 步骤 6.5：技术债务评估

```
/tech-debt
```

扫描 TODO/FIXME/HACK 注释、代码重复、过于复杂的函数、
缺失测试和过时依赖。每个项目分类和优先化。

### 步骤 6.6：协调打磨通道

```
/team-polish "战斗系统"
```

协调 4 个专家并行：
1. 性能优化（performance-analyst）
2. 视觉打磨（technical-artist）
3. 音频打磨（sound-designer）
4. 手感/果汁（gameplay-programmer + technical-artist）

你设置优先级；团队在每个步骤获得你的批准后执行。

### 步骤 6.7：本地化和无障碍

```
/localize src/
```

扫描硬编码字符串、破坏翻译的连接、不考虑扩展的文本
和缺失的区域文件。

根据阶段 3 无障碍要求文档中承诺的层级审计无障碍。

### 阶段 6 门控

```
/gate-check polish
```

**通过要求：**

- 至少 3 个试玩报告存在
- 协调打磨通道完成（`/team-polish`）
- 没有阻塞性能问题
- 无障碍层级要求已满足

---

## 阶段 7：发布

### 这个阶段发生什么

你的游戏已打磨、测试完毕、准备就绪。现在你发货。

### 阶段 7 管线

```
/release-checklist  -->  /launch-checklist  -->  /team-release
        |                       |                      |
        v                       v                      v
  发布前验证         跨部门完整验证    协调：
  跨代码、内容、     （Go/No-Go       构建、QA 签字、
  商店、法律        per 部门）        部署、发布
                    还有：/changelog、/patch-notes、/hotfix
```

### 步骤 7.1：发布检查清单

```
/release-checklist v1.0.0
```

生成涵盖以下内容的综合预发布检查清单：
- 构建验证（所有平台编译和运行）
- 认证要求（平台特定）
- 商店元数据（描述、截图、预告片）
- 法律合规（EULA、隐私政策、评级）
- 保存游戏兼容性
- 分析验证

### 步骤 7.2：发布就绪（完整验证）

```
/launch-checklist
```

完整的跨部门验证：

| 部门 | 检查什么 |
|-----------|---------------|
| **工程** | 构建稳定性、崩溃率、内存泄漏、加载时间 |
| **设计** | 特性完整性、新手教程流程、难度曲线 |
| **美术** | 资源质量、缺失纹理、LOD 级别 |
| **音频** | 缺失声音、混音级别、空间音频 |
| **QA** | 按严重性划分的开放 bug 计数、回归套件通过率 |
| **叙事** | 对话完整性、传说一致性、拼写错误 |
| **本地化** | 所有字符串已翻译、无截断、区域测试 |
| **无障碍** | 合规检查清单、辅助功能测试 |
| **商店** | 元数据完整、截图批准、定价设置 |
| **营销** | 新闻包就绪、发布预告片、社交媒体计划 |
| **社区** | 补丁说明草稿、FAQ 准备、支持渠道就绪 |
| **基础设施** | 服务器扩展、CDN 配置、监控活跃 |
| **法律** | EULA 定稿、隐私政策、COPPA/GDPR 合规 |

每个项目获得 **Go / No-Go** 状态。全部必须 Go 才能发货。

### 步骤 7.3：生成玩家面向内容

```
/patch-notes v1.0.0
```

从 git 历史和 sprint 数据生成玩家友好的补丁说明。
将开发者语言翻译为玩家语言。

```
/changelog v1.0.0
```

生成内部变更日志（更技术化，为团队）。

### 步骤 7.4：协调发布

```
/team-release
```

协调 release-manager、QA 和 DevOps 通过：
1. 发布前验证
2. 构建管理
3. 最终 QA 签字
4. 部署准备
5. Go/No-Go 决策

### 步骤 7.5：发货

`validate-push` hook 会在推送到 `main` 或 `develop` 时警告你。
这是故意的 — 发布推送应该是深思熟虑的：

```bash
git tag v1.0.0
git push origin main --tags
```

### 步骤 7.6：发布后

**关键生产 bug 的热修复工作流程：**

```
/hotfix "当库存超过 99 项时玩家丢失保存数据"
```

绕过正常 sprint 流程并带有完整审计追踪：
1. 创建热修复分支
2. 实现修复
3. 确保 backport 到开发分支
4. 记录事件

**发布稳定后复盘：**

```
让 Claude 使用 `.claude/docs/templates/post-mortem.md` 中的模板创建复盘
```

---

## 跨领域关注点

这些主题适用于所有阶段。

### 导演评审模式

导演门控是在关键工作流程步骤审查你工作的专家 agent。
默认情况下它们在每个检查点运行。你可以控制你获得多少评审。

**在 `/start` 期间设置一次你的评审强度。** 保存到 `production/review-mode.txt`。

| 模式 | 运行什么 | 适用于 |
|------|-----------|----------|
| `full` | 每一步所有导演门控活跃 | 新项目、学习系统 |
| `lean` | 导演仅在阶段转换（`/gate-check`） | 有经验的开发者 |
| `solo` | 任何地方没有导演评审 | Game jam、原型、最大速度 |

**单次运行覆盖**而不更改你的全局设置：

```
/brainstorm space horror --review full
/architecture-decision --review solo
```

`--review` 标志适用于所有使用门控的技能。随时通过直接编辑
`production/review-mode.txt` 或重新运行 `/start` 更改全局模式。

完整门控定义和检查模式：`.claude/docs/director-gates.md`

---

### 协作协议

此系统是 **用户驱动的协作**，不是自主的。

**模式：** 提问 > 选项 > 决策 > 草稿 > 审批

每个 agent 交互遵循此模式：
1. Agent 提问澄清
2. Agent 呈现 2-4 个带权衡和推理的选项
3. 你决定
4. Agent 基于你的决策起草
5. 你审查和细化
6. Agent 在写入前问"可以写入 [filepath] 吗？"

参见 `docs/COLLABORATIVE-DESIGN-PRINCIPLE.md` 获取带示例的完整协议。

### AskUserQuestion 工具

Agent 使用 `AskUserQuestion` 工具进行结构化选项呈现。
模式是解释然后捕获：首先在对话文本中进行完整分析，
然后是干净的 UI 选择器用于决策。用于设计选择、架构决策和战略问题。
不要用于开放式发现提问或简单的 yes/no 确认。

### Agent 协调（三层层级）

```
第一层（导演）：    creative-director、technical-director、producer
                                          |
第二层（主管）：    game-designer、lead-programmer、art-director、
                   audio-director、narrative-director、qa-lead、
                   release-manager、localization-lead
                                          |
第三层（专家）：    gameplay-programmer、engine-programmer、
                   ai-programmer、network-programmer、ui-programmer、
                   tools-programmer、systems-designer、level-designer、
                   economy-designer、world-builder、writer、
                   technical-artist、sound-designer、ux-designer、
                   qa-tester、performance-analyst、devops-engineer、
                   analytics-engineer、accessibility-specialist、
                   live-ops-designer、prototyper、security-engineer、
                   community-manager、godot-specialist、
                   godot-gdscript-specialist、godot-shader-specialist、
                   unity-specialist、unity-csharp-specialist、
                   unreal-specialist、unreal-blueprint-specialist、
                   unreal-cpp-specialist
```

**协调规则：**
- 垂直委托：导演 > 主管 > 专家。复杂决策绝不跳过层级。
- 水平协商：同层级的 Agent 可以互相协商但不能在其领域外做出约束性决策。
- 冲突解决：设计冲突找 `creative-director`。技术冲突找 `technical-director`。范围冲突找 `producer`。
- 禁止单方面跨域变更。

### 自动化 Hooks（安全网）

系统有 12 个自动运行的 hooks：

| Hook | 触发 | 它做什么 |
|------|---------|-------------|
| `session-start.sh` | 会话开始 | 显示分支、最近提交，检测 active.md 用于恢复 |
| `detect-gaps.sh` | 会话开始 | 检测新项目（无引擎、无概念）并建议 `/start` |
| `pre-compact.sh` | 压缩前 | 将会话状态转储到对话中以自动恢复 |
| `post-compact.sh` | 压缩后 | 提醒 Claude 从 `active.md` 恢复会话状态 |
| `notify.sh` | 通知事件 | 通过 PowerShell 显示 Windows toast 通知 |
| `validate-commit.sh` | 提交前 | 检查设计文档引用、有效 JSON、无硬编码值 |
| `validate-push.sh` | 推送前 | 警告推送到 main/develop |
| `validate-assets.sh` | 提交前 | 检查资源命名和大小 |
| `validate-skill-change.sh` | 技能文件写入 | 建议在任何 `.claude/skills/` 更改后运行 `/skill-test` |
| `log-agent.sh` | Agent 开始 | 记录 agent 调用用于审计追踪 |
| `log-agent-stop.sh` | Agent 停止 | 完成 agent 审计追踪（开始 + 停止） |
| `session-stop.sh` | 会话结束 | 最终会话日志 |

### 上下文弹性

**会话状态文件：** `production/session-state/active.md` 是一个动态检查点。
在每个重要里程碑后更新。任何中断后（压缩、崩溃、`/clear`）首先读取此文件。

**增量写入：** 创建多章节文档时，每个章节获批后立即写入文件。
这意味着已完成的章节在崩溃和上下文压缩中存活。关于已写入章节的先前讨论可以安全压缩。

**自动恢复：** `session-start.sh` hook 自动检测和预览 `active.md`。
`pre-compact.sh` hook 在压缩前将状态转储到对话中。

**Sprint 状态追踪：** `production/sprint-status.yaml` 是机器可读的 story 追踪器。
由 `/sprint-plan`（初始化）和 `/story-done`（状态更新）写入。
由 `/sprint-status`、`/help` 和 `/story-done`（下一个 story）读取。
消除了脆弱的 markdown 扫描。

### Brownfield 采用

对于已有一些产物的现有项目：

```
/adopt
```

或针对性：

```
/adopt gdds
/adopt adrs
/adopt stories
/adopt infra
```

这审计现有产物的 **格式**（不是存在性），将差距分类为 BLOCKING/HIGH/MEDIUM/LOW，
构建有序迁移计划，并写入 `docs/adoption-plan-[date].md`。
核心原则：迁移而非替换 — 它从不重新生成现有工作，只填补差距。

单个技能也支持改造模式：

```
/design-system retrofit design/gdd/combat-system.md
/architecture-decision retrofit docs/architecture/adr-005.md
```

这些检测哪些章节存在 vs 缺失，仅填补差距。

### 门控系统

阶段门控是正式检查点。用过渡名称运行 `/gate-check`：

```
/gate-check concept              # Concept -> Systems Design
/gate-check systems-design       # Systems Design -> Technical Setup
/gate-check technical-setup      # Technical Setup -> Pre-Production
/gate-check pre-production       # Pre-Production -> Production
/gate-check production           # Production -> Polish
/gate-check polish               # Polish -> Release
```

**裁决：**
- **PASS** — 所有要求满足，进入下一阶段
- **CONCERNS** — 要求已满足但有承认的风险，可通过
- **FAIL** — 要求未满足，阻止进展并提供具体补救

当门控通过时，`production/stage.txt` 被更新（仅此时），
这控制状态行和 `/help` 行为。

### 反向文档

对于存在而无设计文档的代码（brownfield 采用后常见）：

```
/reverse-document src/gameplay/combat/
```

从现有代码读取并从中生成 GDD 格式设计文档。

---

## 附录 A：Agent 快速参考

### "我需要做 X — 我使用哪个 agent？"

| 我需要... | Agent | 层级 |
|-------------|-------|------|
| 想出一个游戏想法 | `/brainstorm` 技能 | -- |
| 设计游戏机制 | `game-designer` | 2 |
| 设计特定公式/数字 | `systems-designer` | 3 |
| 设计游戏关卡 | `level-designer` | 3 |
| 设计战利品表/经济 | `economy-designer` | 3 |
| 构建世界传说 | `world-builder` | 3 |
| 写对话 | `writer` | 3 |
| 计划故事 | `narrative-director` | 2 |
| 计划 sprint | `producer` | 1 |
| 做创意决策 | `creative-director` | 1 |
| 做技术决策 | `technical-director` | 1 |
| 实现游戏玩法代码 | `gameplay-programmer` | 3 |
| 实现核心引擎系统 | `engine-programmer` | 3 |
| 实现 AI 行为 | `ai-programmer` | 3 |
| 实现多人游戏 | `network-programmer` | 3 |
| 实现 UI | `ui-programmer` | 3 |
| 构建开发工具 | `tools-programmer` | 3 |
| 审查代码架构 | `lead-programmer` | 2 |
| 创建 shaders / VFX | `technical-artist` | 3 |
| 定义视觉风格 | `art-director` | 2 |
| 定义音频风格 | `audio-director` | 2 |
| 设计音效 | `sound-designer` | 3 |
| 设计 UX 流程 | `ux-designer` | 3 |
| 编写测试用例 | `qa-tester` | 3 |
| 计划测试策略 | `qa-lead` | 2 |
| 分析性能 | `performance-analyst` | 3 |
| 设置 CI/CD | `devops-engineer` | 3 |
| 设计分析 | `analytics-engineer` | 3 |
| 检查无障碍 | `accessibility-specialist` | 3 |
| 计划实时运营 | `live-ops-designer` | 3 |
| 管理发布 | `release-manager` | 2 |
| 管理本地化 | `localization-lead` | 2 |
| 快速原型 | `prototyper` | 3 |
| 审计安全 | `security-engineer` | 3 |
| 与玩家沟通 | `community-manager` | 3 |
| Godot 特定帮助 | `godot-specialist` | 3 |
| GDScript 特定帮助 | `godot-gdscript-specialist` | 3 |
| Godot shader 帮助 | `godot-shader-specialist` | 3 |
| GDExtension 模块 | `godot-gdextension-specialist` | 3 |
| Unity 特定帮助 | `unity-specialist` | 3 |
| Unity DOTS/ECS | `unity-dots-specialist` | 3 |
| Unity shaders/VFX | `unity-shader-specialist` | 3 |
| Unity Addressables | `unity-addressables-specialist` | 3 |
| Unity UI Toolkit | `unity-ui-specialist` | 3 |
| Unreal 特定帮助 | `unreal-specialist` | 3 |
| Unreal GAS | `ue-gas-specialist` | 3 |
| Unreal Blueprints | `ue-blueprint-specialist` | 3 |
| Unreal replication | `ue-replication-specialist` | 3 |
| Unreal UMG/CommonUI | `ue-umg-specialist` | 3 |

### Agent 层级

```
                    creative-director / technical-director / producer
                                         |
          ---------------------------------------------------------------
          |            |           |           |          |        |       |
    game-designer  lead-prog  art-dir  audio-dir  narr-dir  qa-lead  release-mgr
          |            |           |           |          |        |        |
     specialists  programmers  tech-art  snd-design  writer   qa-tester  devops
     (systems,    (gameplay,             (sound)     (world-  (perf,     (analytics,
      economy,     engine,                           builder)  access.)   security)
      level)       ai, net,
                   ui, tools)
```

**升级规则：** 如果两个 agent 不同意，向上走。设计冲突找 `creative-director`。
技术冲突找 `technical-director`。范围冲突找 `producer`。

---

## 附录 B：Slash 命令快速参考

### 全部 66 个命令按类别

#### 入门和导航（5）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/start` | 引导式入门，路由到正确工作流程 | 任何（首次会话） |
| `/help` | 上下文感知的"我下一步做什么？" | 任何 |
| `/project-stage-detect` | 完整项目审计以确定当前阶段 | 任何 |
| `/setup-engine` | 配置引擎，固定版本，设置偏好 | 1 |
| `/adopt` | Brownfield 审计和迁移计划 | 任何（现有项目） |

#### 游戏设计（6）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/brainstorm` | 带 MDA 分析的协作构思 | 1 |
| `/map-systems` | 将概念分解为系统索引 | 1-2 |
| `/design-system` | 引导逐节 GDD 创作 | 2 |
| `/quick-design` | 用于小更改的轻量级规格 | 2+ |
| `/review-all-gdds` | 跨 GDD 一致性和设计理论审查 | 2 |
| `/propagate-design-change` | 找到受 GDD 更改影响的 ADR/Story | 5 |

#### UX 和界面（2）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/ux-design` | 创作 UX 规格（屏幕/流程、HUD、模式） | 4 |
| `/ux-review` | 验证 UX 规格的无障碍和 GDD 对齐 | 4 |

#### 架构（4）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/create-architecture` | 主架构文档 | 3 |
| `/architecture-decision` | 创建或改造 ADR | 3 |
| `/architecture-review` | 验证所有 ADR、依赖排序 | 3 |
| `/create-control-manifest` | 从 Accepted ADR 生成扁平程序员规则 | 3 |

#### Story 和 Sprint（8）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/create-epics` | 将 GDD + ADR 翻译为 Epic（每个模块一个） | 4 |
| `/create-stories` | 将单个 Epic 分解为 story 文件 | 4 |
| `/dev-story` | 实现一个 Story — 路由到正确的程序员 Agent | 5 |
| `/sprint-plan` | 创建或管理 sprint 计划 | 4-5 |
| `/sprint-status` | 快速 30 行 sprint 快照 | 5 |
| `/story-readiness` | 验证 Story 可实现就绪 | 4-5 |
| `/story-done` | 8 阶段 Story 完成审查 | 5 |
| `/estimate` | 带风险评估的工作量估算 | 4-5 |

#### 评审和分析（10）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/design-review` | 根据 8 章节标准验证 GDD | 1-2 |
| `/code-review` | 架构代码审查 | 5+ |
| `/balance-check` | 游戏平衡公式分析 | 5-6 |
| `/asset-audit` | 资源命名、格式、大小验证 | 6 |
| `/content-audit` | GDD 指定内容 vs 已实现 | 5 |
| `/scope-check` | 范围蔓延检测 | 5 |
| `/perf-profile` | 性能分析工作流程 | 6 |
| `/tech-debt` | 技术债务扫描和优先化 | 6 |
| `/gate-check` | 带 PASS/CONCERNS/FAIL 的正式阶段门控 | 所有转换 |
| `/reverse-document` | 从现有代码生成设计文档 | 任何 |

#### QA 和测试（9）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/qa-plan` | 为 sprint 或特性生成 QA 测试计划 | 5 |
| `/smoke-check` | QA 交接前关键路径冒烟测试门 | 5-6 |
| `/soak-test` | 长时间游戏会话的浸泡测试协议 | 6 |
| `/regression-suite` | 映射测试覆盖，识别缺少回归测试的已修复 bug | 5-6 |
| `/test-setup` | 为项目引擎搭建测试框架和 CI/CD 管线 | 4 |
| `/test-helpers` | 生成引擎特定测试辅助库 | 4-5 |
| `/test-evidence-review` | 测试文件和手动证据的质量审查 | 5 |
| `/test-flakiness` | 从 CI 日志检测非确定性测试 | 5-6 |
| `/skill-test` | 验证技能文件的结构和行为正确性 | 任何 |

#### 生产管理（6）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/milestone-review` | 里程碑进度和 go/no-go | 5 |
| `/retrospective` | Sprint 回顾分析 | 5 |
| `/bug-report` | 结构化 bug 报告创建 | 5+ |
| `/bug-triage` | 重新评估开放 bug 的优先级、严重性和所有者 | 5+ |
| `/playtest-report` | 结构化试玩会话报告 | 4-6 |
| `/onboard` | 为新团队成员入门 | 任何 |

#### 发布（5）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/release-checklist` | 预发布验证 | 7 |
| `/launch-checklist` | 完整跨部门发布就绪 | 7 |
| `/changelog` | 自动生成内部变更日志 | 7 |
| `/patch-notes` | 玩家面向的补丁说明 | 7 |
| `/hotfix` | 紧急修复工作流程 | 7+ |

#### 创意（2）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/prototype` | 在隔离工作树中的一次性原型 | 4 |
| `/localize` | 字符串提取和验证 | 6-7 |

#### 团队编排（9）

| 命令 | 用途 | 阶段 |
|---------|---------|-------|
| `/team-combat` | 战斗特性：设计到实现 | 5 |
| `/team-narrative` | 叙事内容：结构到对话 | 5 |
| `/team-ui` | UI 特性：UX 规格到打磨实现 | 5 |
| `/team-level` | 关卡：布局到装扮遭遇 | 5 |
| `/team-audio` | 音频：方向到实现事件 | 5-6 |
| `/team-polish` | 协调打磨：性能 + 美术 + 音频 + QA | 6 |
| `/team-release` | 发布协调：构建 + QA + 部署 | 7 |
| `/team-live-ops` | 实时-ops 规划：赛季、活动、战斗通行证、留存 | 7+ |
| `/team-qa` | 完整 QA 周期：策略、执行、覆盖、签字 | 6-7 |

---

## 附录 C：常见工作流程

### 工作流程 1："我刚起步没有游戏想法"

```
1. /start（根据你所在的阶段路由）
2. /brainstorm（协作构思，选择一个概念）
3. /setup-engine（固定引擎和版本）
4. /design-review 关于概念文档（可选，推荐）
5. /map-systems（将概念分解为带依赖和优先级的系统）
6. /gate-check concept（验证你准备好进入系统设计）
7. /design-system 每个系统（引导 GDD 创作）
```

### 工作流程 2："我有设计想开始编码"

```
1. /design-review 关于每个 GDD（确保它们是 solid）
2. /review-all-gdds（跨 GDD 一致性）
3. /gate-check systems-design
4. /create-architecture + /architecture-decision（每个主要决策）
5. /architecture-review
6. /create-control-manifest
7. /gate-check technical-setup
8. /create-epics layer: foundation + /create-stories [slug]（定义 epics，分解为 stories）
9. /sprint-plan new
10. /story-readiness -> implement -> /story-done（story 生命周期）
```

### 工作流程 3："我需要在生产中途添加复杂特性"

```
1. /design-system 或 /quick-design（取决于范围）
2. /design-review 验证
3. /propagate-design-change 如果修改现有 GDD
4. /estimate 工作量和风险
5. /team-combat、/team-narrative、/team-ui 等（适当的团队技能）
6. 完成时 /story-done
7. 如果影响游戏平衡则 /balance-check
```

### 工作流程 4："生产中出了问题"

```
1. /hotfix "问题描述"
2. 修复在 hotfix 分支上实现
3. /code-review 修复
4. 运行测试
5. /release-checklist 用于 hotfix 构建
6. 部署和 backport
```

### 工作流程 5："我有现有项目想使用此系统"

```
1. /start（选择路径 D -- 现有工作）
2. /project-stage-detect（确定当前阶段）
3. /adopt（审计现有产物，构建迁移计划）
4. /design-system retrofit [path]（填补 GDD 差距）
5. /architecture-decision retrofit [path]（填补 ADR 差距）
6. /gate-check 在适当转换
```

### 工作流程 6："开始新的 sprint"

```
1. /retrospective（审查上一个 sprint）
2. /sprint-plan new（创建下一个 sprint）
3. /scope-check（确保范围可管理）
4. 领取前 /story-readiness 每个 story
5. 实现 stories
6. 每个完成 story 后 /story-done
7. /sprint-status 快速进度检查
```

### 工作流程 7："发布游戏"

```
1. /gate-check polish（验证 Polish 阶段完成）
2. /tech-debt（决定发布时可接受什么）
3. /localize（最终本地化通道）
4. /release-checklist v1.0.0
5. /launch-checklist（完整跨部门验证）
6. /team-release（协调发布）
7. /patch-notes 和 /changelog
8. 发货！
9. 发布后如有问题 /hotfix
10. 发布稳定后复盘
```

### 工作流程 8："我迷失了/不知道下一步做什么"

```
1. /help（读取你的阶段，检查产物，告诉你下一步）
2. 如果 /help 没帮助：/project-stage-detect（完整审计）
3. 如果阶段看起来不对：/gate-check 在你认为是阶段的转换
```

---

## 充分利用系统的技巧

1. **始终从设计开始，然后实现。** agent 系统构建在设计文档存在后才写代码的假设上。
   Agent 不断引用 GDD。

2. **对跨领域特性使用团队技能。** 不要试图手动协调 4 个 agent —
   让 `/team-combat`、`/team-narrative` 等处理编排。

3. **信任规则系统。** 当规则标记你代码中的东西时，修复它。
   规则编码了来之不易的游戏开发智慧（数据驱动值、增量时间、无障碍等）。

4. **主动压缩。** 在约 65-70% 上下文使用时压缩或 `/clear`。
   pre-compact hook 保存你的进度。不要等到你达到限制。

5. **使用正确层级的 agent。** 不要问 `creative-director` 写 shader。
   不要问 `qa-tester` 做设计决策。层级存在是有原因的。

6. **不确定时运行 /help。** 它读取你实际的项目状态并告诉你单一最重要的下一步。

7. **在将设计交给程序员前运行 `/design-review`。** 这及早捕获不完整的规格，节省返工。

8. **每个主要特性后运行 `/code-review`。** 在架构问题传播前捕获。

9. **首先原型化有风险的机制。** 一天的原型可以在不起作用的机制上节省一周的生产。

10. **保持你的 sprint 计划诚实。** 定期使用 `/scope-check`。
    范围蔓延是独立游戏的头号杀手。

11. **用 ADR 记录决策。** 未来的你会感谢现在的你记录了*为什么*事情以这种方式构建。

12. **虔诚地使用 story 生命周期。** 领取前 `/story-readiness`，完成后 `/story-done`。
    这及早捕获偏差并保持管线诚实。

13. **尽早经常写入文件。** 增量章节写入意味着你的设计决策在崩溃和压缩中存活。
    文件是记忆，不是对话。