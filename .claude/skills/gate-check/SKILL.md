---
name: gate-check
description: "Validate readiness to advance between development phases. Produces a PASS/CONCERNS/FAIL verdict with specific blockers and required artifacts. Use when user says 'are we ready to move to X', 'can we advance to production', 'check if we can start the next phase', 'pass the gate'."
argument-hint: "[target-phase: systems-design | technical-setup | pre-production | production | polish | release] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write, Task, AskUserQuestion
model: opus
---

# Phase Gate 验证

此技能验证项目是否准备好进入下一个开发阶段。它检查必需的工件、质量标准和阻塞项。

**与 `/project-stage-detect` 的区别**：那个技能是诊断性的（"我们在哪里？"）。此技能是规范性的（"我们准备好推进了吗？"，带有正式裁决）。

## 生产阶段（7 个）

项目通过这些阶段推进：

1. **Concept**——头脑风暴、游戏概念文档
2. **Systems Design**——映射系统、编写 GDD
3. **Technical Setup**——引擎配置、架构决策
4. **Pre-Production**——原型、垂直切片验证
5. **Production**——功能开发（Epic/Feature/Task 跟踪活跃）
6. **Polish**——性能、试玩、bug 修复
7. **Release**——发布准备、认证

当门通过时，将新阶段名称写入 `production/stage.txt`
（单行，例如 `Production`）。这会立即更新状态行。

---

## 1. 解析参数

**目标阶段：** `$ARGUMENTS[0]`（空白 = 自动检测当前阶段，然后验证下一个转换）

还解析审查模式（一次，存储供本次运行的所有门生成使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

注意：在 `solo` 模式下，主管生成（CD-PHASE-GATE、TD-PHASE-GATE、PR-PHASE-GATE、AD-PHASE-GATE）被跳过——gate-check 仅变为工件存在检查。在 `lean` 模式下，所有四个主管仍然运行（phase gates 是 lean 模式的目的）。

- **带参数**：`/gate-check production`——验证该特定阶段的准备情况
- **无参数**：使用与 `/project-stage-detect` 相同的启发式方法自动检测当前阶段，然后**在运行前向用户确认**：

  使用 `AskUserQuestion`：
  - 提示："检测到阶段：**[current stage]**。运行 [Current] → [Next] 转换的门。这正确吗？"
  - 选项：
    - `[A] 是——运行此门`
    - `[B] 否——选择不同的门`（如果选择，显示第二个 widget 列出所有门选项：Concept → Systems Design、Systems Design → Technical Setup、Technical Setup → Pre-Production、Pre-Production → Production、Production → Polish、Polish → Release）
  
  当未提供参数时，不要跳过此确认步骤。

---

## 2. Phase Gate 定义

### 门：Concept → Systems Design

**必需工件：**
- [ ] `design/gdd/game-concept.md` 存在且有内容
- [ ] 游戏支柱已定义（在概念文档或 `design/gdd/game-pillars.md` 中）
- [ ] `design/gdd/game-concept.md` 中存在 Visual Identity Anchor 部分（来自 brainstorm 第 4 阶段 art-director 输出）

**推荐（不阻塞）：**
- [ ] 概念原型存在于 `prototypes/` 中，带有显示 PROCEED 裁决的 REPORT.md
      （`/prototype [core-mechanic]`）——跳过此意味着 GDD 可能为未经试玩验证的想法编写。如果概念通过其他方式证明则可接受。

**质量检查：**
- [ ] 游戏概念已被审查（`/design-review` 裁决不是 MAJOR REVISION NEEDED）
- [ ] 核心循环已描述和理解
- [ ] 目标受众已识别
- [ ] Visual Identity Anchor 包含一行视觉规则和至少 2 个支持性视觉原则

---

### 门：Systems Design → Technical Setup

**必需工件：**
- [ ] Systems index 存在于 `design/gdd/systems-index.md`，至少枚举了 MVP 系统
- [ ] 所有 MVP 级 GDD 存在于 `design/gdd/` 并单独通过 `/design-review`
- [ ] 跨 GDD 审查报告存在于 `design/gdd/` 中（来自 `/review-all-gdds`）

**质量检查：**
- [ ] 所有 MVP GDD 通过单独设计审查（8 个必需部分，无 MAJOR REVISION NEEDED 裁决）
- [ ] `/review-all-gdds` 裁决不是 FAIL（跨 GDD 一致性和设计理论检查通过）
- [ ] `/review-all-gdds` 标记的所有跨 GDD 一致性问题已解决或明确接受
- [ ] 系统依赖关系在 systems index 中映射且双向一致
- [ ] MVP 优先级层级已定义
- [ ] 未标记过时的 GDD 引用（较旧的 GDD 已更新以反映在较新 GDD 中做出的决策）

---

### 门：Technical Setup → Pre-Production

**必需工件：**
- [ ] 引擎已选择（CLAUDE.md 技术栈不是 `[CHOOSE]`）
- [ ] 技术偏好已配置（`.claude/docs/technical-preferences.md` 已填充）
- [ ] Art bible 存在于 `design/art/art-bible.md`，至少包含部分 1-4（Visual Identity Foundation）
- [ ] `docs/architecture/` 中至少有 3 个架构决策记录涵盖
      Foundation 层系统（场景管理、事件架构、save/load）
- [ ] 引擎参考文档存在于 `docs/engine-reference/[engine]/`
- [ ] 测试框架已初始化：`tests/unit/` 和 `tests/integration/` 目录存在
- [ ] CI/CD 测试工作流存在于 `.github/workflows/tests.yml`（或等效文件）
- [ ] 至少存在一个示例测试文件以确认框架功能正常
- [ ] 主架构文档存在于 `docs/architecture/architecture.md`
- [ ] 架构可追溯性索引存在于 `docs/architecture/requirements-traceability.md`
- [ ] 已运行 `/architecture-review`（审查报告文件存在于 `docs/architecture/` 中）
- [ ] `design/accessibility-requirements.md` 存在，已承诺无障碍层级
- [ ] `design/ux/interaction-patterns.md` 存在（模式库已初始化，即使最小）

**质量检查：**
- [ ] 架构决策涵盖核心系统（渲染、输入、状态管理）
- [ ] 技术偏好设置了命名约定和性能预算
- [ ] 无障碍层级已定义并记录（即使是"基本"也可接受——未定义不可接受）
- [ ] 至少一个屏幕的 UX 规范已开始（通常在 Technical Setup 期间设计主菜单或核心 HUD）
- [ ] 所有 ADR 都有 **Engine Compatibility 部分**，盖有引擎版本章
- [ ] 所有 ADR 都有 **GDD Requirements Addressed 部分**，有明确的 GDD 链接
- [ ] 没有 ADR 引用 `docs/engine-reference/[engine]/deprecated-apis.md` 中列出的 API
- [ ] 所有 HIGH RISK 引擎领域（根据 VERSION.md）已在架构文档中明确解决或标记为未解决问题
- [ ] 架构可追溯性矩阵在 Foundation 层**零空白**
      （所有 Foundation 需求在 Pre-Production 之前必须有 ADR 覆盖）

**ADR 循环依赖检查：** 对于 `docs/architecture/` 中的所有 ADR，读取每个 ADR 的
"ADR Dependencies" / "Depends On" 部分。构建依赖图（ADR-A → ADR-B 表示
A 依赖于 B）。如果检测到任何循环（例如 A→B→A，或 A→B→C→A）：
- 标记为 **FAIL**："循环 ADR 依赖：[ADR-X] → [ADR-Y] → [ADR-X]。
  在循环存在的情况下，两者都无法达到 Accepted。移除一个 'Depends On' 边以
  打破循环。"

**引擎验证**（首先读取 `docs/engine-reference/[engine]/VERSION.md`）：
- [ ] 触及 cutoff 后引擎 API 的 ADR 标记为 Knowledge Risk: HIGH/MEDIUM
- [ ] `/architecture-review` 引擎审计显示无已弃用 API 使用
- [ ] 所有 ADR 同意相同的引擎版本（无过时版本引用）

---

### 门：Pre-Production → Production

**必需工件：**
- [ ] 垂直切片存在于 `prototypes/` 中，带有 REPORT.md（运行 `/vertical-slice`）——**推荐，不阻塞**；如果缺失，作为 CONCERNS 展示
- [ ] 第一个 sprint 计划存在于 `production/sprints/` 中
- [ ] Art bible 已完成（所有 9 个部分）且 AD-ART-BIBLE 签字确认裁决记录在 `design/art/art-bible.md` 中
- [ ] 实体清单存在于 `design/assets/entity-inventory.md`（推荐——运行 `/asset-spec` 不带参数以从 GDD + art bible 协作生成）
- [ ] Systems index 中的所有 MVP 级 GDD 已完成
- [ ] 主架构文档存在于 `docs/architecture/architecture.md`
- [ ] `docs/architecture/` 中至少有 3 个涵盖 Foundation 层决策的 ADR
- [ ] 所有 Foundation 和 Core 层 ADR 的状态为 `Accepted`（不是 `Proposed`）——story 无法解除阻塞，直到其管辖 ADR 被接受
- [ ] Control manifest 存在于 `docs/architecture/control-manifest.md`
      （由 `/create-control-manifest` 从 Accepted ADR 生成）
- [ ] Epic 在 `production/epics/` 中定义，至少存在 Foundation 和 Core
      层 epic（使用 `/create-epics layer: foundation` 和
      `/create-epics layer: core` 创建它们，然后为每个 epic 运行 `/create-stories [epic-slug]`）
- [ ] 垂直切片构建存在且可玩（不仅仅是范围定义）——**推荐，不阻塞**；如果缺失，作为 CONCERNS 展示
- [ ] 垂直切片已被试玩，至少有 1 次记录会话——**推荐，不阻塞**；如果缺失，作为 CONCERNS 展示
- [ ] 垂直切片试玩报告存在于 `production/playtests/` 或等效位置——**推荐，不阻塞**；如果缺失，作为 CONCERNS 展示
- [ ] 关键屏幕存在 UX 规范：主菜单、核心游戏 HUD（在 `design/ux/` 中）、暂停菜单
- [ ] HUD 设计文档存在于 `design/ux/hud.md` 中（如果游戏有游戏内 HUD）
- [ ] 所有关键屏幕 UX 规范已通过 `/ux-review`（裁决 APPROVED 或 NEEDS REVISION 已接受）

**质量检查：**
- [ ] **核心循环乐趣已验证**——试玩数据确认核心机制令人愉悦，不仅仅是功能性的。明确检查垂直切片试玩报告。
- [ ] UX 规范涵盖 MVP 级 GDD 的所有 UI Requirements 部分
- [ ] 交互模式库记录关键屏幕中使用的模式
- [ ] `design/accessibility-requirements.md` 中的无障碍层级在所有关键屏幕 UX 规范中得到解决
- [ ] Sprint 计划引用来自 `production/epics/` 的真实 story 文件路径
      （不仅仅是 GDD——story 必须嵌入 GDD req ID + ADR 引用）
- [ ] **垂直切片已完成**，不仅仅是范围定义——构建端到端展示完整核心循环。至少一个完整的 [开始 → 挑战 → 解决] 循环有效。
- [ ] 架构文档在 Foundation 或 Core 层没有未解决的未解决问题
- [ ] 所有 ADR 都有盖有引擎版本章的 Engine Compatibility 部分
- [ ] 所有 ADR 都有 ADR Dependencies 部分（即使所有字段都是 "None"）
- [ ] 手动验证确认 GDD + 架构 + epic 是连贯的
      （如果最近没有完成，运行 `/review-all-gdds` 和 `/architecture-review`）
- [ ] **核心幻想已交付**——至少有一名试玩者在没有被提示的情况下独立描述了与核心系统 GDD 的 Player Fantasy 部分相匹配的体验。

**垂直切片验证**（仅在构建了垂直切片时运行这些检查）：
- [ ] 有人在没有开发者指导的情况下玩完了核心循环
- [ ] 游戏在开始玩的前 2 分钟内传达了要做什么
- [ ] 垂直切片构建中不存在关键的"乐趣阻塞"bug
- [ ] 核心机制感觉良好互动（这是一个主观检查——询问用户）

> **垂直切片的裁决规则：**
> - **切片已构建且任何验证项为否** → 裁决自动为 FAIL。损坏的
>   或无聊的垂直切片不应推进到 Production。
> - **切片未构建（跳过）** → 仅降级为 CONCERNS，不是 FAIL。清楚地展示风险：
>   "在没有经过验证的垂直切片的情况下推进会增加后期设计
>   转变的风险。在承诺完整生产范围之前推荐。"用户决定。
>   跳过是 solo 开发或时间限制的合理选择。发布损坏的不是。

---

### 门：Production → Polish

**必需工件：**
- [ ] `src/` 有组织成子系统的活动代码
- [ ] GDD 中的所有核心机制已实现（将 `design/gdd/` 与 `src/` 交叉引用）
- [ ] 主要游戏路径端到端可玩
- [ ] 测试文件存在于 `tests/unit/` 和 `tests/integration/` 中，涵盖 Logic 和 Integration story
- [ ] 此 sprint 的所有 Logic story 在 `tests/unit/` 中有对应的单元测试文件
- [ ] 冒烟检查已运行，裁决为 PASS 或 PASS WITH WARNINGS——报告存在于 `production/qa/` 中
- [ ] QA 计划存在于 `production/qa/` 中（由 `/qa-plan` 生成）涵盖此 sprint 或最终生产 sprint
- [ ] `production/qa/` 中至少存在一个 QA 计划涵盖此生产阶段——如果缺失则运行 `/qa-plan`（CONCERNS——建议性，不阻塞）
- [ ] QA 签字确认报告存在于 `production/qa/` 中（由 `/team-qa` 生成），裁决为 APPROVED 或 APPROVED WITH CONDITIONS
- [ ] 至少 3 次不同的试玩会话记录在 `production/playtests/` 中
- [ ] 试玩报告涵盖：新玩家体验、中期系统和难度曲线
- [ ] 游戏概念中的乐趣假设已明确验证或修订

**质量检查：**
- [ ] 测试通过（通过 Bash 运行测试套件）
- [ ] 任何 bug 跟踪器或已知问题中没有关键/阻塞 bug
- [ ] 核心循环按设计进行（与 GDD 验收标准比较）
- [ ] 性能在预算内（检查 technical-preferences.md 目标）
- [ ] 试玩发现已审查且关键乐趣问题已解决（不仅仅是记录）
- [ ] 未识别"困惑循环"——游戏中没有超过 50% 的试玩者卡住而不知道为什么的点
- [ ] 难度曲线与 Difficulty Curve 设计文档匹配（如果存在于 `design/difficulty-curve.md`）
- [ ] 所有实现的屏幕都有对应的 UX 规范（无"代码中设计"的屏幕）
- [ ] 交互模式库与实现中使用的所有模式保持最新
- [ ] 无障碍合规性对照 `design/accessibility-requirements.md` 中承诺的层级进行验证

---

### 门：Polish → Release

**必需工件：**
- [ ] 里程碑计划中的所有功能已实现
- [ ] 内容完整（设计文档中引用的所有关卡、资产、对话存在）
- [ ] 本地化字符串已外部化（`src/` 中无硬编码的玩家面向文本）
- [ ] QA 测试计划存在（`/qa-plan` 输出在 `production/qa/` 中）
- [ ] QA 签字确认报告存在（`/team-qa` 输出——APPROVED 或 APPROVED WITH CONDITIONS）
- [ ] 所有 Must Have story 测试证据存在（Logic/Integration：测试文件通过；Visual/Feel/UI：`production/qa/evidence/` 中的签字确认文档）
- [ ] 冒烟检查在候选发布构建上干净通过（PASS 裁决）
- [ ] 与先前 sprint 无测试回归（测试套件完全通过）
- [ ] 平衡数据已审查（运行了 `/balance-check`）
- [ ] 发布清单已完成（运行了 `/release-checklist` 或 `/launch-checklist`）
- [ ] 商店元数据已准备（如适用）
- [ ] Changelog / patch notes 已起草

**质量检查：**
- [ ] 完整的 QA 通过由 `qa-lead` 签字确认
- [ ] 所有测试通过
- [ ] 性能目标在所有目标平台上达到
- [ ] 无已知的关键、高或中严重性 bug
- [ ] 无障碍基础覆盖（重映射、文本缩放，如适用）
- [ ] 本地化针对所有目标语言验证
- [ ] 法律要求满足（EULA、隐私政策、年龄评级，如适用）
- [ ] 构建干净编译和打包

---

## 3. 运行门检查

**在运行工件检查之前**，如果存在，读取 `docs/consistency-failures.md`。
提取 Domain 与目标阶段匹配的条目（例如，如果检查
Systems Design → Technical Setup，拉取 Economy、Combat 或任何 GDD 领域的条目；
如果检查 Technical Setup → Pre-Production，拉取 Architecture、Engine 中的条目）。
将这些作为上下文携带——目标领域中的反复冲突模式值得
对这些特定检查加大审查力度。

对于目标门中的每个项目：

### 工件检查
- 使用 `Glob` 和 `Read` 验证文件存在且有有意义的内容
- 不要只检查存在——验证文件有真实内容（不仅仅是模板头部）
- 对于代码检查，验证目录结构和文件计数

**Systems Design → Technical Setup 门——跨 GDD 审查检查**：
使用 `Glob('design/gdd/gdd-cross-review-*.md')` 查找 `/review-all-gdds` 报告。
如果没有文件匹配，将"跨 GDD 审查报告存在"工件标记为 **FAIL** 并
突出展示："在 `design/gdd/` 中未找到 `/review-all-gdds` 报告。在推进到 Technical Setup 之前运行
`/review-all-gdds`。"
如果找到文件，读取它并检查裁决行：FAIL 裁决意味着跨 GDD 一致性检查失败，必须在推进前解决。

### 质量检查
- 对于测试检查：如果配置了测试运行器，通过 `Bash` 运行测试套件
- 对于设计审查检查：`Read` GDD 并检查 8 个必需部分
- 对于性能检查：`Read` technical-preferences.md 并与 `tests/performance/` 中的任何分析数据或最近的 `/perf-profile` 输出进行比较
- 对于本地化检查：`Grep` `src/` 中的硬编码字符串

### 交叉引用检查
- 将 `design/gdd/` 文档与 `src/` 实现进行比较
- 检查架构文档中引用的每个系统都有对应的代码
- 验证 sprint 计划引用真实的工作项

---

## 4. 协作评估

对于无法自动验证的项目，**询问用户**：

- "我无法自动验证核心循环是否玩得好。它被试玩了吗？"
- "未找到试玩报告。是否进行了非正式测试？"
- "性能分析数据不可用。你想运行 `/perf-profile` 吗？"

**永远不要对无法验证的项目假设 PASS。** 将它们标记为 MANUAL CHECK NEEDED。

---

## 4b. 主管小组评估

**在生成任何主管之前应用审查模式：**
- `solo` → 跳过所有四个主管。在输出中注明："主管小组已跳过——Solo 模式。门裁决仅基于工件和质量检查。" 进入第 5 阶段。
- `lean` → 生成所有四个主管（phase gates 始终在 lean 模式下运行——这是它们的目的）。
- `full` → 正常生成所有四个主管。

（审查模式在第 1 阶段已解析。在此使用存储的值。）

在生成最终裁决之前，通过 Task 使用 `.claude/docs/director-gates.md` 中的并行门协议将所有四个主管作为**并行子代理**生成。同时发出所有四个 Task 调用——不要在开始下一个之前等待一个。

**并行生成：**

1. **`creative-director`**——门 **CD-PHASE-GATE**（`.claude/docs/director-gates.md`）
2. **`technical-director`**——门 **TD-PHASE-GATE**（`.claude/docs/director-gates.md`）
3. **`producer`**——门 **PR-PHASE-GATE**（`.claude/docs/director-gates.md`）
4. **`art-director`**——门 **AD-PHASE-GATE**（`.claude/docs/director-gates.md`）

传递给每个：目标阶段名称、存在的工件列表以及该门定义中列出的上下文字段。

**收集所有四个响应，然后展示主管小组摘要：**

```
## 主管小组评估

Creative Director：  [READY / CONCERNS / NOT READY]
  [反馈]

Technical Director： [READY / CONCERNS / NOT READY]
  [反馈]

Producer：           [READY / CONCERNS / NOT READY]
  [反馈]

Art Director：       [READY / CONCERNS / NOT READY]
  [反馈]
```

**应用于裁决：**
- 任何主管返回 NOT READY → 裁决最低为 FAIL（用户可以用明确确认覆盖）
- 任何主管返回 CONCERNS → 裁决最低为 CONCERNS
- 所有四个 READY → 有资格获得 PASS（仍受第 3 节的工件和质量检查约束）

---

## 5. 输出裁决

```
## 门检查：[当前阶段] → [目标阶段]

**日期**：[date]
**检查者**：gate-check skill

### 必需工件：[X/Y 存在]
- [x] design/gdd/game-concept.md——存在，2.4KB
- [ ] docs/architecture/——缺失（未找到 ADR）
- [x] production/sprints/——存在，1 个 sprint 计划

### 质量检查：[X/Y 通过]
- [x] GDD 有 8/8 必需部分
- [ ] 测试——失败（tests/unit/ 中有 3 个失败）
- [?] 核心循环已试玩——MANUAL CHECK NEEDED

### 阻塞项
1. **无架构决策记录**——在进入生产之前运行 `/architecture-decision` 创建一个
   涵盖核心系统架构。
2. **3 个测试失败**——在推进之前修复 tests/unit/ 中失败的测试。

### 建议
- [解决阻塞项目的优先操作]
- [不阻塞的可选改进]

### 裁决：[PASS / CONCERNS / FAIL]
- **PASS**：所有必需工件存在，所有质量检查通过
- **CONCERNS**：存在小空白，但可以在下一阶段解决
- **FAIL**：在推进之前必须解决关键阻塞项
```

---

## 5a. 验证链

在第 5 阶段起草裁决后，在最终确定之前挑战它。

**步骤 1 — 生成 5 个挑战问题**，旨在反驳裁决：

> **工具操作要求**：以下 5 个挑战问题中至少 2 个必须通过重新读取特定文件（Read 工具）或重新运行特定检查（Grep 工具）来回答——不仅仅是反思。用 [TOOL ACTION] 标记这些以指示使用了工具。

对于 **PASS** 草稿：
- "我通过实际读取文件验证了哪些质量检查，与推断它们通过？"
- "是否有我标记为 PASS 的 MANUAL CHECK NEEDED 项目而没有用户确认？[TOOL ACTION] 重新扫描清单中的任何 [?] 或 MANUAL CHECK 项目。"
- "我是否确认所有列出的工件都有真实内容，而不仅仅是空头部？[TOOL ACTION] 重新读取文件并检查它是否有非占位符内容。"
- "我作为次要驳回的任何阻塞项是否实际上可能阻止阶段成功？"
- "我最不确定的单一检查是什么，为什么？"

对于 **CONCERNS** 草稿：
- "鉴于项目的当前状态，列出的任何 CONCERN 是否可以升级为阻塞项？"
- "关注点是否可以在下一阶段内解决，还是会随时间复合？"
- "我是否将任何 FAIL 条件软化为 CONCERNS 以避免更难的裁决？"
- "是否有我没有检查的工件可能揭示额外的阻塞项？"
- "所有 CONCERNS 在一起是否创建了阻塞问题，即使每个单独都很小？"

对于 **FAIL** 草稿：
- "我是否准确地将硬阻塞项与强烈建议分开？"
- "是否有任何 PASS 项目我过于宽松？"
- "我是否遗漏了用户应该知道的任何额外阻塞项？"
- "我可以提供通往 PASS 的最小路径——必须改变的特定 3 件事吗？"
- "失败条件是否可解决，还是它表明更深的设计问题？"

**步骤 2 — 独立回答每个问题。**
不要参考裁决草稿文本——重新检查特定文件或询问用户。

**步骤 3 — 如需修订：**
- 如果任何答案揭示遗漏的阻塞项 → 升级裁决（PASS→CONCERNS 或 CONCERNS→FAIL）
- 如果任何答案揭示过度陈述的阻塞项 → 仅在引用具体证据时降级
- 如果答案一致 → 确认裁决不变

**步骤 4 — 在最终报告输出中注明验证：**
`验证链：[N] 个问题已检查——裁决 [未更改 | 从 X 修订为 Y]`

---

## 6. 通过时更新阶段

当裁决为 **PASS** 且用户确认他们想要推进时：

1. 将新阶段名称写入 `production/stage.txt`（单行，无尾随换行符）
2. 这会立即更新所有未来会话的状态行

示例：如果通过 "Pre-Production → Production" 门：
```bash
echo -n "Production" > production/stage.txt
```

**始终在写入前询问**："门已通过。我可以将 `production/stage.txt` 更新为 'Production' 吗？"

---

## 7. 关闭后续步骤 Widget

在裁决呈现且任何 stage.txt 更新完成后，使用 `AskUserQuestion` 以结构化的后续步骤提示关闭。

**为刚刚运行的门定制选项：**

对于 **systems-design 通过**：
```
门已通过。你想下一步做什么？
[A] 运行 /create-architecture — 生成你的主架构蓝图和 ADR 工作计划（推荐的下一步）
[B] 先设计更多 GDD —— 当所有 MVP 系统完成时回到这里
[C] 本次会话停在这里
```

> **systems-design 通过的注意事项**：`/create-architecture` 是在写任何 ADR 之前必需的下一步。它生成主架构文档和要写的 ADR 的优先列表。没有此步骤运行 `/architecture-decision` 意味着在没有蓝图的情况下写 ADR——风险自负。

对于 **technical-setup 通过**：
```
门已通过。你想下一步做什么？
[A] 运行 /create-control-manifest — 从你的 Accepted ADR 生成层级规则 manifest（先做这个）
[B] 运行 /vertical-slice —— 构建垂直切片（在写 epic 之前做这个——先验证乐趣）
[C] 先写更多 ADR —— 运行 /architecture-decision [next-system]
[D] 本次会话停在这里
```

> **technical-setup 通过的注意事项**：Pre-Production 序列是故意排序的，
> 以在承诺详细规划之前验证乐趣：
>
> 1. `/create-control-manifest` — 从 Accepted ADR 提取技术规则（epic 之前必需）
> 2. `/vertical-slice` — **首先**构建垂直切片，在写 epic 或 story 之前
> 3. 试玩 → `/playtest-report` — 通过 Pre-Production 门至少需要 1 次会话；在承诺完整团队之前推荐 3+ 次
> 4. `/ux-design [screen]` — 主菜单、核心 HUD、暂停菜单的 UX 规范（如果未完成）
> 5. `/create-epics layer:foundation` 然后 `/create-epics layer:core` — 乐趣验证后计划
> 6. `/create-stories [epic-slug]` 为每个 epic
> 7. `/sprint-plan new`
>
> **为什么在 epic 之前原型？** 如果原型揭示核心循环需要改变，
> 在那次发现之前写的 epic 将部分错误。先便宜地验证乐趣，
> 然后详细计划。这是 GDC 事后分析数据中的 #1 教训。

对于所有其他门，为该阶段提供两个最合乎逻辑的下一步加上"停在这里"。

---

## 8. 后续操作

根据裁决，建议具体的后续步骤：

- **没有 art bible？** → `/art-bible` 创建视觉身份规范
- **Art bible 存在但没有资产规范？** → `/asset-spec system:[name]` 从批准的 GDD 生成每个资产的视觉规范和生成提示
- **没有游戏概念？** → `/brainstorm` 创建一个
- **没有 systems index？** → `/map-systems` 将概念分解为系统
- **缺少设计文档？** → `/reverse-document` 或委托给 `game-designer`
- **需要小设计更改？** → `/quick-design` 用于约 ~4 小时以下的更改（绕过完整 GDD 管道）
- **没有 UX 规范？** → `/ux-design [screen name]` 编写规范，或 `/team-ui [feature]` 用于完整管道
- **UX 规范未审查？** → `/ux-review [file]` 或 `/ux-review all` 验证
- **没有无障碍需求文档？** → 运行 `/ux-design`，它一步创建 `design/accessibility-requirements.md` 和 `design/ux/interaction-patterns.md`
- **没有交互模式库？** → `/ux-design patterns` 初始化它
- **GDD 未交叉审查？** → `/review-all-gdds`（在所有 MVP GDD 单独批准后运行）
- **跨 GDD 一致性问题？** → 修复标记的 GDD，然后重新运行 `/review-all-gdds`
- **没有测试框架？** → `/test-setup` 为你的引擎搭建框架
- **当前 sprint 没有 QA 计划？** → `/qa-plan sprint` 在实现开始前生成一个
- **缺少 ADR？** → `/architecture-decision` 用于单个决策
- **没有主架构文档？** → `/create-architecture` 用于完整蓝图
- **ADR 缺少引擎兼容性部分？** → 重新运行 `/architecture-decision`
  或手动将 Engine Compatibility 部分添加到现有 ADR
- **缺少 control manifest？** → `/create-control-manifest`（需要 Accepted ADR）
- **缺少 epic？** → `/create-epics layer: foundation` 然后 `/create-epics layer: core`（需要 control manifest）
- **Epic 缺少 story？** → `/create-stories [epic-slug]`（在每个 epic 创建后运行）
- **Story 未准备好实现？** → `/story-readiness` 在开发者接手之前验证 story
- **测试失败？** → 委托给 `lead-programmer` 或 `qa-tester`
- **没有试玩数据？** → `/playtest-report`
- **超过最低限度的试玩会话？** → 额外会话提供更可靠信号。在承诺完整团队之前推荐总共 3+ 次。使用 `/playtest-report` 构建发现。
- **没有 Difficulty Curve 文档？** → 从 `.claude/docs/templates/difficulty-curve.md` 中的模板创建 `design/difficulty-curve.md` —— 或使用 `/quick-design "difficulty curve"` 进行引导会话。
- **没有玩家旅程地图？** → 从 `.claude/docs/templates/player-journey.md` 中的模板创建 `design/player-journey.md` —— 或使用 `/ux-design` 第 2b 阶段协作编写。
- **需要快速 sprint 检查？** → `/sprint-status` 获取当前 sprint 进度快照
- **性能未知？** → `/perf-profile`
- **未本地化？** → `/localize`
- **准备发布？** → `/launch-checklist`

---

## 协作协议

此技能遵循协作设计原则：

1. **先扫描**：检查所有工件和质量门
2. **询问未知数**：对你无法验证的事情不要假设 PASS
3. **展示发现**：显示完整清单及状态
4. **用户决定**：裁决是建议——用户做最终决定
5. **获得批准**："我可以将此门检查报告写入 production/gate-checks/ 吗？"
6. **永远不要自动修复**：如果必需工件缺失，报告 FAIL 裁决并
   命名要运行的技能（例如"运行 `/test-setup`"）。不要创建缺失的文件或
   自动重新运行门。创建文件来制造 PASS 会破坏门的
   目的。

**永远不要**阻止用户推进——裁决是建议性的。记录风险并
让用户决定是否在有顾虑的情况下继续。
