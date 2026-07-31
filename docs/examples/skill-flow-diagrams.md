# 技能流程图

跨7个开发阶段技能如何链接的可视化地图。
这些显示每个技能之前和之后运行什么，以及哪些产物在它们之间流动。

---

## 完整管线概览（从零到发布）

```
阶段 1: 概念
  /start ──────────────────────────────────────────────────────► 路由到 A/B/C/D
  /brainstorm ──────────────────────────────────────────────────► design/gdd/game-concept.md
  /setup-engine ────────────────────────────────────────────────► CLAUDE.md + technical-preferences.md
  /prototype [核心机制] ────────────────────────────────────────► prototypes/[name]-concept/REPORT.md
        │ PROCEED                                                  (在写 GDD 之前验证想法)
        ▼
  /design-review [game-concept.md] ────────────────────────────► 概念已验证
  /gate-check ─────────────────────────────────────────────────► PASS → 推进到系统设计
        │
        ▼
阶段 2: 系统设计
  /map-systems ────────────────────────────────────────────────► design/gdd/systems-index.md
        │
        ▼ (按依赖顺序，对每个系统)
  /design-system [name] ──────────────────────────────────────► design/gdd/[system].md
  /design-review [system].md ─────────────────────────────────► 每个 GDD 审查评论
        │
        ▼ (所有 MVP GDD 完成后)
  /review-all-gdds ────────────────────────────────────────────► design/gdd/gdd-cross-review-[date].md
  /gate-check ─────────────────────────────────────────────────► PASS → 推进到技术设置
        │
        ▼
阶段 3: 技术设置
  /create-architecture ────────────────────────────────────────► docs/architecture/master.md
  /architecture-decision (×N) ─────────────────────────────────► docs/architecture/[adr-nnn].md
  /architecture-review ────────────────────────────────────────► 审查报告 + docs/architecture/tr-registry.yaml
  /create-control-manifest ────────────────────────────────────► docs/architecture/control-manifest.md
  /gate-check ─────────────────────────────────────────────────► PASS → 推进到预生产
        │
        ▼
阶段 4: 预生产
  [UX — 在 epic 之前，所以规格存在时故事被编写]
  /ux-design [screen/hud/patterns] ────────────────────────────► design/ux/*.md
  /ux-review ─────────────────────────────────────────────────► UX 规格已批准（/team-ui 的硬门）

  [测试基础设施 — 在引用测试的故事之前搭建]
  /test-setup ─────────────────────────────────────────────────► 测试框架 + CI/CD 管道
  /test-helpers ───────────────────────────────────────────────► tests/helpers/[engine-specific].gd

  [Vertical slice — 在 epic 之前，验证完整游戏循环]
  /vertical-slice ─────────────────────────────────────────────► prototypes/[name]-vertical-slice/REPORT.md
  /playtest-report ────────────────────────────────────────────► production/playtests/

  [Stories + sprint plan — 仅在 vertical slice PROCEED 之后]
  /create-epics [layer] ───────────────────────────────────────► production/epics/*/EPIC.md
  /create-stories [epic-slug] ─────────────────────────────────► production/epics/*/story-*.md
  /sprint-plan new ────────────────────────────────────────────► production/sprints/sprint-01.md
  /gate-check ─────────────────────────────────────────────────► PASS → 推进到生产
        │
        ▼
阶段 5: 生产（重复 sprint 循环）
  /sprint-status ──────────────────────────────────────────────► sprint 快照
  /story-readiness [story] ────────────────────────────────────► story 已验证 READY
        │
        ▼ (领取并实施)
  /dev-story [story] ─────────────────────────────────────────► 路由到正确的 programmer agent
        │
        ▼ (实施期间，按需)
  /code-review ────────────────────────────────────────────────► 代码审查报告
  /scope-check ────────────────────────────────────────────────► 范围蔓延检测 / 清晰
  /content-audit ──────────────────────────────────────────────► GDD 内容差距识别
  /bug-report ─────────────────────────────────────────────────► production/qa/bugs/bug-NNN.md
  /bug-triage ─────────────────────────────────────────────────► bugs 重新优先排序 + 分配

  [功能区域的团队技能 — 处理完整功能时派生]
  /team-combat / /team-narrative / /team-ui / /team-level / /team-audio

  [每个 sprint 的 QA 循环]
  /qa-plan ───────────────────────────────────────────────────► production/qa/qa-plan-sprint-NN.md
  /smoke-check ────────────────────────────────────────────────► 冒烟测试门 (PASS/FAIL)
  /regression-suite ───────────────────────────────────────────► 覆盖率差距 + 缺失的回归测试
  /test-evidence-review ───────────────────────────────────────► 证据质量报告
  /test-flakiness ─────────────────────────────────────────────► 不稳定测试报告
        │
        ▼
  /story-done [story] ─────────────────────────────────────────► story 关闭 + surfaced 下一个
  /sprint-plan [next] ─────────────────────────────────────────► 下一个 sprint
        │
        ▼ (生产里程碑之后)
  /milestone-review ───────────────────────────────────────────► 里程碑报告
  /gate-check ─────────────────────────────────────────────────► PASS → 推进到打磨
        │
        ▼
阶段 6: 打磨
  /perf-profile ───────────────────────────────────────────────► 性能报告 + 修复
  /balance-check ──────────────────────────────────────────────► 平衡报告 + 修复
  /asset-audit ────────────────────────────────────────────────► 资产合规报告
  /tech-debt ─────────────────────────────────────────────────► docs/tech-debt-register.md
  /soak-test ─────────────────────────────────────────────────► 浸泡测试协议 + 结果
  /localize ───────────────────────────────────────────────────► 本地化就绪报告
  /team-polish ───────────────────────────────────────────────► 打磨 sprint 编排
  /team-qa ───────────────────────────────────────────────────► 完整 QA 循环 sign-off
  /gate-check ─────────────────────────────────────────────────► PASS → 推进到发布
        │
        ▼
阶段 7: 发布
  /launch-checklist ───────────────────────────────────────────► 发布就绪报告
  /release-checklist ──────────────────────────────────────────► 平台特定检查清单
  /changelog ──────────────────────────────────────────────────► CHANGELOG.md
  /patch-notes ────────────────────────────────────────────────► 面向玩家的说明
  /team-release ───────────────────────────────────────────────► 发布管道编排
        │
        ▼ (发布后，持续)
  /hotfix ─────────────────────────────────────────────────────► 带审计追踪的紧急修复
  /team-live-ops ──────────────────────────────────────────────► 持续运营内容计划
```

---

## 技能链：/design-system 详解

单个 GDD 如何被创作、审查并移交给架构：

```
systems-index.md (输入)
game-concept.md (输入)
上游 GDD (输入，如果有的话)
        │
        ▼
/design-system [name]
        │
        ├── 预检查：可行性表 + 引擎风险标记
        │
        ├── 章节循环 × 8:
        │     提问 → 选项 → 决策 → 草稿 → 批准 → 写入
        │     [每个章节在批准后立即写入文件]
        │
        └── 输出: design/gdd/[system].md (完整，8个章节)
                │
                ▼
        /design-review design/gdd/[system].md
                │
                ├── APPROVED → 在 systems-index 标记 DONE，进行下一个系统
                ├── NEEDS REVISION → agent 显示具体问题，重新进入章节循环
                └── MAJOR REVISION → 需要重大重新设计才能进行下一个系统
                        │
                        ▼ (所有 MVP GDD + 交叉审查之后)
                /review-all-gdds
                        │
                        └── 输出: gdd-cross-review-[date].md
```

---

## 技能链：UX/UI 管线详解

UX 规格在阶段 4（预生产）创作，在 epic 被编写之前，
以便 story 验收标准可以引用具体的 UX 产物。

```
design/gdd/*.md (提取的 UI/UX 需求)
design/player-journey.md (情绪弧线，如果已创作)
        │
        ▼
/ux-design hud              → design/ux/hud.md
/ux-design screen [name]    → design/ux/screens/[name].md
/ux-design patterns         → design/ux/interaction-patterns.md
        │
        ▼
/ux-review design/ux/
        │
        ├── APPROVED → UX 规格就绪，进行 /create-epics
        ├── NEEDS REVISION → 阻塞问题列出 → 修复 → 重新运行审查
        └── MAJOR REVISION → 根本性 UX 问题 → epic 前重新设计
                │
                ▼ (APPROVED 之后 — 在阶段5实施 UI 功能时)
        /team-ui
                │
                ├── 阶段1: /ux-design (如果有规格仍然缺失) + /ux-review
                ├── 阶段2: 视觉设计 (art-director)
                ├── 阶段3: 布局实施 (ui-programmer)
                ├── 阶段4: 无障碍审计 (accessibility-specialist)
                └── 阶段5: 最终审查

注意: /ux-design 和 /ux-review 属于阶段4（预生产）。
      /team-ui 属于阶段5（生产），当 UI 功能正在构建时。
```

---

## 技能链：开发 Story 流程详解

Story 如何从 backlog 移动到关闭：

```
/story-readiness [story]
        │
        ├── READY → 状态: ready-for-dev → 领取实施
        ├── NEEDS WORK → agent 显示具体差距 → 解决 → 重新运行就绪检查
        └── BLOCKED → ADR 仍是 Proposed，或上游 story 未完成
                │
                ▼ (READY 之后)
        /dev-story [story]
                │
                ├── 读取: story 文件、链接的 GDD 需求、ADR 决策、控制清单
                ├── 路由到: gameplay-programmer / engine-programmer / ui-programmer 等
                │
                └── 实施开始
                        │
                        ▼ (可选，在/实施期间或之后)
                /code-review          → 变更集的架构审查
                /scope-check          → 验证与原始 story 标准相比无范围蔓延
                /test-evidence-review → 验证测试文件和手动证据质量
                        │
                        ▼
                /story-done [story]
                        │
                        ├── COMPLETE → 状态: Complete，sprint-status.yaml 更新，surfaced 下一个 story
                        ├── COMPLETE WITH NOTES → 完成但某些标准推迟（已记录）
                        └── BLOCKED → 验收标准无法验证 → 调查阻塞
```

---

## 技能链：Story 生命周期（Backlog 到关闭）

Story 如何从 backlog 到关闭（摘要视图）：

```
/create-epics [layer]
        │
        └── 输出: production/epics/[slug]/EPIC.md
                │
                ▼
        /create-stories [epic-slug]
                │
                └── 输出: production/epics/[slug]/story-NNN-[slug].md
                            (状态: Ready 或如果 ADR 是 Proposed 则 Blocked)
                │
                ▼
        /story-readiness [story]
                │
                ├── READY → /dev-story → 实施 → /story-done
                ├── NEEDS WORK → 解决差距 → 重新运行
                └── BLOCKED → 先修复上游依赖
```

---

## 技能链：QA 管线详解

```
[阶段4 — 一次性基础设施设置]
/test-setup ────────────────────────────────────────────────────► 测试框架搭建 + CI/CD 接线
/test-helpers ──────────────────────────────────────────────────► tests/helpers/[engine].gd (GDUnit4, NUnit 等)

[阶段5 — 每 sprint QA 循环]
/qa-plan [sprint 或 feature]
        │
        ├── 读取: story 文件、GDD、验收标准
        ├── 将每个 story 按测试类型分类:
        │     Logic → 自动化单元测试 (BLOCKING)
        │     Integration → 集成测试或记录的试玩 (BLOCKING)
        │     Visual/Feel → 截图 + 主管 sign-off (ADVISORY)
        │     UI → 手动演练或交互测试 (ADVISORY)
        │     Config/Data → 冒烟检查 (ADVISORY)
        └── 输出: production/qa/qa-plan-sprint-NN.md
                │
                ▼
        /smoke-check
                │
                ├── PASS → QA hand-off 已清除
                └── FAIL → 阻止 sprint 关闭 → 先修复关键路径
                        │
                        ▼
                /regression-suite
                        │
                        └── 覆盖率差距 + 修复了 bug 但没有回归测试的列表
                                │
                                ▼
                        /test-evidence-review
                                │
                                └── 验证证据质量，不只是存在
                                        │
                                        ▼ (如果有 CI 运行历史)
                        /test-flakiness
                                │
                                └── 不稳定测试报告 + 修复建议

[阶段6 — 扩展稳定性测试]
/soak-test ─────────────────────────────────────────────────────► 浸泡测试协议 + 观察结果
/team-qa ───────────────────────────────────────────────────────► 完整 QA 循环 sign-off 用于发布门

[持续 — bug 管理]
/bug-report ────────────────────────────────────────────────────► production/qa/bugs/bug-NNN.md
/bug-triage ────────────────────────────────────────────────────► 开放 bugs 重新优先排序 + 分配

[元 — 工具验证]
/skill-test [lint|spec|catalog] ────────────────────────────────► 技能文件结构 + 行为检查
```

---

## 技能链：UX 管线详解（遗留参考）

对于有现有工作的项目（使用 `/start` 选项 D 或直接运行）：

```
/project-stage-detect    → 阶段检测报告
        │
        ▼
/adopt
        │
        ├── 阶段1: 检测存在什么
        ├── 阶段2: FORMAT 审计（不只是存在）
        ├── 阶段3: 分类差距 (BLOCKING / HIGH / MEDIUM / LOW)
        ├── 阶段4: 有序迁移计划
        ├── 阶段5: 写 docs/adoption-plan-[date].md
        └── 阶段6: 内联修复最紧急的差距（可选）
                │
                ▼
        /design-system retrofit [path]    → 填补缺失的 GDD 章节
        /architecture-decision retrofit [path] → 填补缺失的 ADR 章节
        /gate-check                       → 你在管线的哪个位置？
```

---

## 如何阅读这些图表

| 符号 | 含义 |
|--------|---------|
| `──►` | 生成此产物 |
| `│ ▼` | 流入下一步 |
| `├──` | 分支（多种可能结果）|
| `×N` | 运行 N 次（每个系统、story 等一次）|
| `(input)` | 被技能读取但不在这里生成 |
| `[optional]` | 对门通过不是必需的 |
| `WRITE` (大写) | 文件立即写入磁盘 |

---

## 常见入口点

| 你所在位置 | 运行这个 |
|---------------|---------|
| 全新，不知道 | `/start` → `/brainstorm` |
| 有概念，没有引擎 | `/setup-engine` |
| 有概念 + 引擎 | `/map-systems` |
| 系统设计中期 | `/design-system [下一个系统]` 或 `/map-systems next` |
| 所有 GDD 完成 | `/review-all-gdds` → `/gate-check` |
| 技术设置中 | `/create-architecture` → `/architecture-decision` |
| 开始 UX 设计 | `/ux-design screen [name]` 或 `/ux-design hud` |
| 搭建测试 | `/test-setup` → `/test-helpers` |
| 有 stories，准备编码 | `/story-readiness [story]` → `/dev-story [story]` |
| Story 完成 | `/story-done [story]` |
| 运行 sprint QA | `/qa-plan` → `/smoke-check` → `/regression-suite` |
| Bug backlog 需要排序 | `/bug-triage` |
| 扩展稳定性测试 | `/soak-test` |
| 不确定 | `/help` |
| 现有项目 | `/adopt` |
