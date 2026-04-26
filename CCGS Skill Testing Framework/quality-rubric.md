# 技能质量规则

由 `/skill-test category [name|all]` 使用，以评估超出结构合规性的技能。
每个类别定义 4-5 个特定于技能工作的二进制 PASS/FAIL 指标。

当技能的书面指令清楚满足标准时，指标为 PASS。
当指令缺失、模糊或矛盾时，指标为 FAIL。
当指令部分解决标准时，指标为 WARN。

---

## 技能类别

### `gate`

**技能**: gate-check

Gate 技能控制阶段转换。它们必须在不自动推进 stage 的情况下强制正确性，并且必须尊重三种审查模式。

| 指标 | PASS 标准 |
|---|---|
| **G1 — 审查模式已读** | 技能在决定派生哪些 directors 之前读取 `production/session-state/review-mode.txt`（或等效文件） |
| **G2 — Full 模式：所有 4 个 directors 派生** | 在 `full` 模式中，所有 4 个 Tier-1 directors（CD、TD、PR、AD）的 PHASE-GATE 提示并行调用 |
| **G3 — Lean 模式：仅 PHASE-GATE** | 在 `lean` 模式中，仅运行 `*-PHASE-GATE` gates；内联 gates（CD-PILLARS、TD-ARCHITECTURE 等）被跳过 |
| **G4 — Solo 模式：无 directors** | 在 `solo` 模式中，不派生任何 director gates；每个被记为"skipped — Solo mode" |
| **G5 — 无自动推进** | 技能在未通过"May I write"获得用户明确确认的情况下从不写入 `production/stage.txt` |

---

### `review`

**技能**: design-review, architecture-review, review-all-gdds

Review 技能读取文档并产生结构化裁决。它们主要是只读的，在分析阶段不得触发 director gates。

| 指标 | PASS 标准 |
|---|---|
| **R1 — 只读强制** | 技能在未获得用户明确批准的情况下不修改被审查的文档；任何写操作（审查日志、索引更新）都在"May I write"之后 |
| **R2 — 8章节检查** | 技能明确评估所有 8 个必需的 GDD 章节（或等效的架构章节）|
| **R3 — 正确的裁决词汇** | 裁决正好是以下之一：APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED（设计）或 PASS / CONCERNS / FAIL（架构）|
| **R4 — 分析期间无 director gates** | 技能在其分析阶段不派生 director gates；后分析 director 审查（如 architecture-review）在技能的scope和stakes warrant 时可接受 |
| **R5 — 结构化发现** | 输出在最终裁决之前包含每个部分的状态表或检查清单 |

> **例外：**
> - `design-review`：在 allowed-tools 中有 `Write, Edit` 以支持可选的"现在修订"路径（所有写操作在用户批准后进行）并写入审查日志。R1 被满足，因为被审查的文档从未被静默修改。
> - `architecture-review`：在其分析完成后派生 TD-ARCHITECTURE 和 LP-FEASIBILITY gates。这是故意的 — 架构审查是高风险的，从 director sign-off 中受益。R4 被满足，因为 gates 在分析后运行，而不是在分析期间。

---

### `authoring`

**技能**: design-system, quick-design, architecture-decision, ux-design, ux-review, art-bible, create-architecture

Authoring 技能协作创建或更新设计文档。完整的 GDD/UX 创作技能使用逐节循环；轻量级创作技能使用适合其较小范围的单一草稿模式。

| 指标 | PASS 标准 |
|---|---|
| **A1 — 逐节循环** | 完整创作技能（design-system、ux-design、art-bible）一次创作一个章节，在继续下一个之前呈现内容供批准。轻量级技能（quick-design、architecture-decision、create-architecture）可以起草完整文档然后请求批准 — 对于实施范围小于约4小时的文档，单一草稿可接受。 |
| **A2 — 每个章节的 May-I-write** | 完整创作技能在每个章节写入前询问"可以写入 [filepath] 吗？"。轻量级技能对完整文档询问一次。 |
| **A3 — Retrofit 模式** | 技能检测目标文件是否已存在，并提供更新特定章节而不是覆盖整个文档。始终创建新文件的轻量级技能（quick-design）豁免。 |
| **A4 — 正确层级的 Director gate** | 如果为此技能定义了 director gate（如 CD-GDD-ALIGN、TD-ADR），它以正确的模式阈值（full/lean）运行 — 不在 solo 中 |
| **A5 — 骨架优先** | 完整创作技能在填充内容之前创建带所有章节标题的文件骨架，以在会话中断时保留进度。轻量级技能豁免。 |

> **完整创作技能**（必须通过所有5个指标）：`design-system`、`ux-design`、`art-bible`
> **轻量级创作技能**（A1、A2、A5 使用单一草稿模式；A3 对仅新文件技能豁免）：`quick-design`、`architecture-decision`、`create-architecture`
> **Review 模式技能**（对照 review 指标评估）：`ux-review`

---

### `readiness`

**技能**: story-readiness, story-done

Readiness 技能在实施之前或之后验证 stories。它们必须产生多维度裁决并正确集成 director gate 模式。

| 指标 | PASS 标准 |
|---|---|
| **RD1 — 多维度检查** | 技能检查 ≥3 个独立维度（如 Design、Architecture、Scope、DoD）并分别报告 |
| **RD2 — 三个裁决级别** | 裁决层次明确定义：READY/COMPLETE > NEEDS WORK/COMPLETE WITH NOTES > BLOCKED |
| **RD3 — BLOCKED 需要外部操作** | BLOCKED 裁决保留给无法由 story 作者单独解决的问题（如 Proposed ADR、不可解决的依赖）|
| **RD4 — 正确模式的 Director gate** | QL-STORY-READY 或 LP-CODE-REVIEW gate 在 `full` 模式中派生，在 `lean`/`solo` 中跳过并带有记为跳过的消息 |
| **RD5 — 下一个 story 交接** | 完成后，技能从 active sprint 中 surfaced 下一个 READY story |

---

### `pipeline`

**技能**: create-epics, create-stories, dev-story, create-control-manifest, propagate-design-change, map-systems

Pipeline 技能产生其他技能消耗的产物。它们必须以正确的 schema 写入文件，尊重层级/优先级顺序，并在写入前设门。

| 指标 | PASS 标准 |
|---|---|
| **P1 — 正确的输出 schema** | 每个生成的文件遵循项目模板（EPIC.md、story frontmatter 等）；技能引用模板路径 |
| **P2 — 层级/优先级顺序** | 生成 epics 或 stories 的技能尊重层级顺序（core → extended → meta）和优先级字段 |
| **P3 — 每个产物前 May-I-write** | 技能在创建每个输出文件前询问"可以写入 [artifact] 吗？"，而不是一次批准所有文件 |
| **P4 — 正确层级的 Director gate** | 范围内的 gates（PR-EPIC、QL-STORY-READY、LP-CODE-REVIEW 等）在 `full` 中运行，在 `lean`/`solo` 中跳过并带有记为跳过的消息 |
| **P5 — 写入前读取** | 技能在生成产物之前读取相关 GDD/ADR/manifest 以确保一致性 |

---

### `analysis`

**技能**: consistency-check, balance-check, content-audit, code-review, tech-debt,
scope-check, estimate, perf-profile, asset-audit, security-audit, test-evidence-review, test-flakiness

Analysis 技能扫描项目并 surfaced 发现。它们在分析期间是只读的，必须在推荐任何文件写入之前询问。

| 指标 | PASS 标准 |
|---|---|
| **AN1 — 只读扫描** | 分析阶段仅使用 Read/Glob/Grep 工具；在扫描本身期间无 Write 或 Edit |
| **AN2 — 结构化发现表** | 输出包含发现表或检查清单（不仅仅是散文），每个发现有严重性/优先级 |
| **AN3 — 无自动写入** | 任何建议的文件写入（如 tech-debt register、fix patches）都在"May I write"之后 |
| **AN4 — 分析期间无 director gates** | Analysis 技能不派生 director gates；它们产生发现供人工审查 |

---

### `team`

**技能**: team-combat, team-narrative, team-audio, team-level, team-ui, team-qa,
team-release, team-polish, team-live-ops

Team 技能为部门编排多个专家 agents。它们必须派生正确的 agents，在独立的 agents 上并行运行，并立即 surfaced 阻塞。

| 指标 | PASS 标准 |
|---|---|
| **T1 — 命名 agent 列表** | 技能明确命名它派生的 agents 及其顺序 |
| **T2 — 在独立的地方并行** | 输入不相互依赖的 agents 并行派生（单个消息，多个 Task 调用）|
| **T3 — BLOCKED surfacing** | 如果任何派生的 agent 返回 BLOCKED 或失败，技能立即 surfaced 并停止依赖工作 — 从不静默跳过 |
| **T4 — 在继续之前收集所有裁决** | 依赖阶段等待所有并行 agents 完成后再继续 |
| **T5 — 参数缺失时使用错误** | 如果缺少必需参数（如 feature 名称），技能输出使用提示并停止而不派生 agents |

---

### `sprint`

**技能**: sprint-plan, sprint-status, milestone-review, retrospective, changelog, patch-notes

Sprint 技能读取生产状态并产生报告或规划产物。它们在特定模式阈值有 PR-SPRINT 或 PR-MILESTONE gate。

| 指标 | PASS 标准 |
|---|---|
| **SP1 — 读取 sprint/milestone 状态** | 技能在生成输出之前读取 `production/sprints/` 或 `production/milestones/` |
| **SP2 — 正确的 sprint gate** | PR-SPRINT（用于规划）或 PR-MILESTONE（用于里程碑审查）gate 在 `full` 模式中运行，在 `lean`/`solo` 中跳过 |
| **SP3 — 结构化输出** | 输出使用一致的结构（velocity 表、风险列表、行动项目）而不是自由散文 |
| **SP4 — 无自动提交** | 技能在未获得"May I write"的情况下从不写入 sprint 文件或里程碑记录 |

---

### `utility`

**技能**: start, help, brainstorm, onboard, adopt, hotfix, prototype, localize,
launch-checklist, release-checklist, smoke-check, soak-test, test-setup, test-helpers,
regression-suite, qa-plan, bug-triage, bug-report, playtest-report, asset-spec,
reverse-document, project-stage-detect, setup-engine, skill-test, skill-improve,
day-one-patch 以及上述类别之外的其他技能

Utility 技能通过7个标准静态检查。如果它们恰好派生 director gates，gate 模式逻辑也必须正确。

| 指标 | PASS 标准 |
|---|---|
| **U1 — 通过所有7个静态检查** | `/skill-test static [name]` 返回 COMPLIANT，0 个 FAIL |
| **U2 — Gate 模式正确（如适用）** | 如果技能派生任何 director gate，它读取 review-mode 并正确应用 full/lean/solo 逻辑 |

---

## Agent 类别

用于验证 `tests/agents/` 中的 agent 规格文件。

### `director`

**Agents**: creative-director, technical-director, art-director, producer

| 指标 | PASS 标准 |
|---|---|
| **D1 — 正确的裁决词汇** | 返回 APPROVE / CONCERNS / REJECT（或领域等效：producer 的 REALISTIC/CONCERNS/UNREALISTIC）|
| **D2 — 尊重领域边界** | 不在其声明的领域外做出约束性决策 |
| **D3 — 冲突升级** | 当两个部门冲突时，升级到正确的父级（creative-director 或 technical-director）而不是单方面决定 |
| **D4 — Opus 模型层级** | Agent per coordination-rules.md 被分配 Opus 模型 |

### `lead`

**Agents**: lead-programmer, qa-lead, narrative-director, audio-director, game-designer,
systems-designer, level-designer

| 指标 | PASS 标准 |
|---|---|
| **L1 — 领域裁决** | 返回领域特定的裁决（如 lead-programmer 的 FEASIBLE/INFEASIBLE，qa-lead 的 PASS/FAIL）|
| **L2 — 升级到共享父级** | 领域外冲突升级到 creative-director（设计）或 technical-director（技术）|
| **L3 — Sonnet 模型层级** | Agent per coordination-rules.md 被分配 Sonnet 模型（默认）|

### `specialist`

**Agents**: gameplay-programmer, ai-programmer, technical-artist, sound-designer,
engine-programmer, tools-programmer, network-programmer, security-engineer,
accessibility-specialist, ux-designer, ui-programmer, performance-analyst, prototyper,
qa-tester, writer, world-builder

| 指标 | PASS 标准 |
|---|---|
| **S1 — 保持在领域内** | 明确限定自己在其声明的领域内；推迟领域外的请求 |
| **S2 — 无约束性跨域决策** | 不单方面决定由另一个 specialist 拥有的事项 |
| **S3 — 正确推迟** | 领域外的请求被重定向到正确的 agent，而不是静默拒绝 |

### `engine`

**Agents**: godot-specialist, godot-gdscript-specialist, godot-csharp-specialist,
godot-shader-specialist, godot-gdextension-specialist, unity-specialist, unity-ui-specialist,
unity-shader-specialist, unity-dots-specialist, unity-addressables-specialist,
unreal-specialist, ue-blueprint-specialist, ue-gas-specialist, ue-umg-specialist,
ue-replication-specialist

| 指标 | PASS 标准 |
|---|---|
| **E1 — 版本感知** | 在建议 API 调用之前从 `docs/engine-reference/` 引用引擎版本；标记 post-cutoff 风险 |
| **E2 — 文件路由** | 将文件类型路由到正确的子专家（如 `.gdshader` → godot-shader-specialist，而不是 godot-gdscript-specialist）|
| **E3 — 引擎特定模式** | 强制引擎特定习语（如 GDScript 静态类型、C# 属性导出、Blueprint 函数库）|

### `qa`

**Agents**: qa-tester, qa-lead, security-engineer, accessibility-specialist

| 指标 | PASS 标准 |
|---|---|
| **Q1 — 产生产物而非代码** | 主要输出是测试用例、bug 报告或覆盖差距 — 而不是实现代码 |
| **Q2 — 证据格式** | 测试用例遵循项目的测试证据格式（coding-standards.md 中的 unit/integration/visual/UI）|
| **Q3 — 无范围蔓延** | 不提议新功能；标记差距供人工决定 |

### `operations`

**Agents**: devops-engineer, release-manager, live-ops-designer, community-manager,
analytics-engineer, economy-designer, localization-lead

| 指标 | PASS 标准 |
|---|---|
| **O1 — 领域所有权清晰** | Agent 描述清楚说明它拥有什么（管道、发布、经济等）|
| **O2 — 推迟实施** | 不写游戏逻辑或引擎代码；委托给适当的专家 |
| **O3 — 工具集匹配角色** | frontmatter 中的 `allowed-tools` 匹配角色的操作（而非编码）性质 |
