# 升级 Claude Code Game Studios

本指南涵盖将现有游戏项目仓库从模板的一个版本升级到下一个版本。

**在你的 git log 中找到当前版本：**
```bash
git log --oneline | grep -i "release\|setup"
```
或检查 `README.md` 中的版本徽章。

---

## 目录

- [升级策略](#升级策略)
- [v1.0.0-beta → v1.0](#v100-beta--v10)
- [v0.4.x → v1.0](#v04x--v10)
- [v0.4.0 → v0.4.1](#v040--v041)
- [v0.3.0 → v0.4.0](#v030--v040)
- [v0.2.0 → v0.3.0](#v020--v030)
- [v0.1.0 → v0.2.0](#v010--v020)

---

## 升级策略

有三种方式拉取模板更新。根据你的仓库设置方式选择。

### 策略 A — Git Remote Merge（推荐）

最适合：你克隆了模板并在之上有自己的 commits。

```bash
# 将模板添加为 remote（一次性设置）
git remote add template https://github.com/Donchitos/Claude-Code-Game-Studios.git

# 获取新版本
git fetch template main

# 合并到你的分支
git merge template/main --allow-unrelated-histories
```

Git 仅在模板*和你*都更改过的文件中标记冲突。解决每一个 — 你的游戏内容保留，结构性改进随之而来。然后提交合并。

**提示：** 最可能冲突的文件是 `CLAUDE.md` 和 `.claude/docs/technical-preferences.md`，因为你已经在其中填入了引擎和项目设置。保留你的内容；接受结构性更改。

---

### 策略 B — Cherry-pick 特定 commits

最适合：你只需要一个特定功能（如仅新技能，而不是完整更新）。

```bash
git remote add template https://github.com/Donchitos/Claude-Code-Game-Studios.git
git fetch template main

# Cherry-pick 你想要的具体 commit(s)
git cherry-pick <commit-sha>
```

每个版本的 commit SHA 列在下面的版本部分中。

---

### 策略 C — 手动文件复制

最适合：你没有用 git 设置模板（只是下载了 zip）。

1. 下载或克隆新版本到你的仓库旁边。
2. 直接复制**"可安全覆盖"**下列出的文件。
3. 对于**"仔细合并"**下的文件，两边打开两个版本并手动合并结构性更改，同时保留你的内容。

---

## v0.4.1

**发布日期：** 2026-04-02
**关键主题：** 美术方向集成、资产规格管道

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新技能** | `/art-bible` — 引导式逐节视觉身份创作（9个章节）。每章节强制 art-director Task 派生。AD-ART-BIBLE sign-off gate。技术设置阶段必需。 |
| **新技能** | `/asset-spec` — 每资产视觉规格和 AI 生成提示生成器。读取 art bible + GDD/level/character 文档。写入 `design/assets/specs/` 文件和 `design/assets/asset-manifest.md`。Full/lean/solo 模式。 |
| **新 director gates（3个）** | `AD-CONCEPT-VISUAL`（brainstorm 阶段4）、`AD-ART-BIBLE`（art bible sign-off）、`AD-PHASE-GATE`（gate-check 面板） |
| **`/brainstorm` 更新** | 在 allowed-tools 中添加 `Task`（之前缺失 — 阻止所有 director 派生）。Pillars 锁定后 art-director 与 creative-director 并行派生。视觉身份锚点写入 game-concept.md。 |
| **`/gate-check` 更新** | Art-director 添加为第4个并行 director（AD-PHASE-GATE）。视觉产物检查：视觉身份锚点（Concept gate）、art bible（Technical Setup gate）、AD-ART-BIBLE sign-off + 角色视觉档案（Pre-Production gate）。 |
| **`/team-level` 更新** | Art-director 添加到步骤1并行派生（视觉方向在布局之前）。Level-designer 现在接收 art-director 目标作为明确约束。步骤4 art-director 角色更正为仅 production-concepts。 |
| **`/team-narrative` 更新** | Art-director 添加到阶段2并行派生（角色视觉设计、环境叙事、电影色调）。 |
| **`/design-system` 更新** | 路由表扩展了 art-director + technical-artist 用于战斗、UI、对话、动画/VFX、角色类别。视觉/音频章节现在对7个系统类别强制（带 art-director Task 派生）。 |
| **`workflow-catalog.yaml`** | `/art-bible` 添加到 Technical Setup（必需）。`/asset-spec` 添加到 Pre-Production（可选，可重复）。 |

### 文件：可安全覆盖

**新增文件：**
```
.claude/skills/art-bible/SKILL.md
.claude/skills/asset-spec/SKILL.md
.claude/docs/director-gates.md
```

**现有文件覆盖（无用户内容）：**
```
.claude/skills/brainstorm/SKILL.md
.claude/skills/gate-check/SKILL.md
.claude/skills/team-level/SKILL.md
.claude/skills/team-narrative/SKILL.md
.claude/skills/design-system/SKILL.md
.claude/docs/workflow-catalog.yaml
README.md
UPGRADING.md
```

### 文件：仔细合并

无 — 所有变更都是基础设施文件，无用户内容。

---

## v1.0.0-beta → v1.0

**Released:** 2026-05-13
**Commit range:** `49d1e45..HEAD`
**Key themes:** New `/vertical-slice` gate, skill polish & bug fixes, contributor docs

### What Changed

| Category | Changes |
|----------|---------|
| **New skill** | `/vertical-slice` — Pre-Production gate that validates the full game loop with a production-quality end-to-end build before Production. Pairs with the overhauled `/prototype` (concept validation right after `/brainstorm`). |
| **New flow** | Entity inventory step in `/map-systems` — surfaces all named entities up front for cleaner downstream GDD authoring. |
| **UX polish** | Added missing `AskUserQuestion` widgets to 7 skills; comprehensive skill audit for consistency, prompts, and flow gaps; exposed `--review` flag in `argument-hints` for all `team-*` skills. |
| **Bug fixes** | `#21` log-agent hooks logged "unknown" `agent_type`; `#36` missing `allowed-tools` in `/architecture-decision` and `/story-done`; `#42` `rg --type gdscript` is invalid (now uses `--glob *.gd`); `#43` session-start preview showed oldest state instead of newest; `#45` duplicate `## 0.` heading and broken step numbering in `/architecture-decision`. |
| **Project docs** | Added `CONTRIBUTING.md` (framework contribution guidelines) and `SECURITY.md` (coordinated disclosure policy). |
| **Counts/refs** | Synced agent/skill/hook counts across `WORKFLOW-GUIDE.md`, `README.md`, and agent rosters; fixed stale agent names and skill model-tier fields. |

---

### Files: Safe to Overwrite

**New files to add:**
```
.claude/skills/vertical-slice/SKILL.md
CONTRIBUTING.md
SECURITY.md
```

**Existing files to overwrite (no user content):**
- All files under `.claude/skills/` modified in the commit range (skill audit + AskUserQuestion widgets + `--review` argument-hints)
- `.claude/hooks/log-agent.sh` (fix #21)
- `README.md`, `docs/WORKFLOW-GUIDE.md`, `docs/examples/skill-flow-diagrams.md`
- `UPGRADING.md`

---

### Files: Merge Carefully

None — all changes are to infrastructure files with no user content.

---

## v0.4.x → v1.0

**发布日期：** 2026-03-29
**Commit 范围：** `6c041ac..HEAD`
**关键主题：** Director gates 系统、gate 强度模式、Godot C# 专家

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新系统** | Director gates — 跨所有工作流技能共享的命名审查检查点。定义在 `.claude/docs/director-gates.md` |
| **新功能** | Gate 强度模式：`full`（所有 director gates）、`lean`（仅阶段 gates）、`solo`（无 directors）。通过 `/start` 期间全局设置（写入 `production/review-mode.txt`），或在任何 gate 使用技能上用 `--review [mode]` 覆盖每次运行 |
| **新 agent** | `godot-csharp-specialist` — Godot 4 项目中的 C# 代码质量 |
| **技能更新（13个）** | 所有 gate 使用技能现在解析 `--review [full|lean|solo]` 并将其包含在 argument-hint 中：`brainstorm`、`map-systems`、`design-system`、`architecture-decision`、`create-architecture`、`create-epics`、`create-stories`、`sprint-plan`、`milestone-review`、`playtest-report`、`prototype`、`story-done`、`gate-check` |
| **`/start` 更新** | 添加阶段3b — 在 onboarding 期间设置审查模式，写入 `production/review-mode.txt` |
| **`/setup-engine` 更新** | Godot 的语言选择步骤（GDScript vs C#） |
| **文档** | `director-gates.md` — 完整 gate 目录；`WORKFLOW-GUIDE.md` — Director 审查模式章节；`README.md` — 审查强度自定义 |

---

### 文件：可安全覆盖

**新增文件：**
```
.claude/agents/godot-csharp-specialist.md
.claude/docs/director-gates.md
```

**现有文件覆盖（无用户内容）：**
```
.claude/skills/brainstorm/SKILL.md
.claude/skills/map-systems/SKILL.md
.claude/skills/design-system/SKILL.md
.claude/skills/architecture-decision/SKILL.md
.claude/skills/create-architecture/SKILL.md
.claude/skills/create-epics/SKILL.md
.claude/skills/create-stories/SKILL.md
.claude/skills/sprint-plan/SKILL.md
.claude/skills/milestone-review/SKILL.md
.claude/skills/playtest-report/SKILL.md
.claude/skills/prototype/SKILL.md
.claude/skills/story-done/SKILL.md
.claude/skills/gate-check/SKILL.md
.claude/skills/start/SKILL.md
.claude/skills/quick-design/SKILL.md
.claude/skills/setup-engine/SKILL.md
README.md
docs/WORKFLOW-GUIDE.md
UPGRADING.md
```

---

### 文件：仔细合并

此版本中无文件需要手动合并。所有变更都是基础设施文件，无用户内容。

---

### 新功能

#### Director Gates 系统

所有主要工作流技能现在引用定义在 `.claude/docs/director-gates.md` 中的命名 gate 检查点。Gates 通过领域前缀和名称识别（如 `CD-CONCEPT`、`TD-ARCHITECTURE`、`LP-CODE-REVIEW`）。每个 gate 定义派生哪个 director、传递什么输入、裁决意味着什么，以及 lean/solo 模式如何影响它。

技能使用 `Task` 派生 gates，带 gate ID 和文档化输入，而不是内联嵌入 director 提示。这保持技能体干净并使 gate 行为在所有工作流阶段中一致。

#### Gate 强度模式

三种模式让你控制你获得多少 director 审查：

- **`full`**（默认）— 每个审查检查点运行所有 director gates
- **`lean`** — 跳过每个技能的 director 审查；`/gate-check` 的阶段 gates 仍然运行
- **`solo`** — 任何地方都没有 director gates；`/gate-check` 仅检查产物存在

在 `/start` 期间全局设置（写入 `production/review-mode.txt`）。在任何 gate 使用技能上用 `--review [mode]` 覆盖任何单独运行：

```
/design-system combat --review lean
/gate-check concept --review full
/brainstorm my-game-idea --review solo
```

---

### 升级后

1. 运行 `/start` 一次以设置你首选的审查模式 — 或者手动创建 `production/review-mode.txt`，内容为 `full`、`lean` 或 `solo`。
2. 如果你在项目中期，审查 `.claude/docs/director-gates.md` 以了解哪些 gates 适用于你当前阶段。
3. 运行 `/skill-test static all` 验证所有技能通过结构性检查。

---

## v0.4.0 → v0.4.1

**发布日期：** 2026-03-26
**Commit 范围：** `04ed5d5..HEAD`
**关键主题：** 领域无关 agents、新技能、技能修复

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新技能（1个）** | `/consistency-check` — 跨 GDD 实体一致性扫描器 |
| **技能修复（所有 team-*）** | 添加无参数守卫、正式 `Verdict: COMPLETE / BLOCKED` 关键词、每步骤 AskUserQuestion gates、相邻区域依赖检查（team-level）、伦理 enforcement（team-live-ops）、NO-GO 路径带阶段跳过（team-release） |
| **Agent 修复（4个）** | game-designer、systems-designer、economy-designer、live-ops-designer 中的领域无关语言 — 移除 RPG 特定术语 |

---

### 文件：可安全覆盖

**新增文件：**
```
.claude/skills/consistency-check/SKILL.md
```

**现有文件覆盖（无用户内容）：**
```
.claude/skills/team-combat/SKILL.md      ← 无参数守卫、verdict 关键词、gate 改进
.claude/skills/team-narrative/SKILL.md   ← 无参数守卫、verdict 关键词、gate 改进
.claude/skills/team-ui/SKILL.md          ← 无参数守卫、verdict 关键词、gate 改进
.claude/skills/team-release/SKILL.md     ← 无参数守卫、verdict 关键词、NO-GO 路径
.claude/skills/team-polish/SKILL.md      ← 无参数守卫、verdict 关键词、gate 改进
.claude/skills/team-audio/SKILL.md       ← 无参数守卫、verdict 关键词、gate 改进
.claude/skills/team-level/SKILL.md       ← 无参数守卫、verdict 关键词、相邻区域检查
.claude/skills/team-live-ops/SKILL.md    ← 无参数守卫、verdict 关键词、伦理 enforcement
.claude/skills/team-qa/SKILL.md          ← 无参数守卫、verdict 关键词、gate 改进
.claude/skills/map-systems/SKILL.md      ← verdict 关键词
.claude/skills/create-epics/SKILL.md     ← "May I write" 协议修复、verdict 关键词
.claude/skills/create-stories/SKILL.md   ← verdict 关键词
.claude/agents/game-designer.md          ← 领域无关语言
.claude/agents/systems-designer.md       ← 领域无关语言
.claude/agents/economy-designer.md       ← 领域无关语言
.claude/agents/live-ops-designer.md      ← 领域无关语言
```

---

### 文件：仔细合并

此版本中无文件需要手动合并。所有变更都是基础设施文件，无用户内容。

---

### 升级后

1. 运行 `/skill-test catalog` 验证所有技能已索引。
2. 任何技能编辑后运行 `/skill-test lint [skill-name]` 检查结构性合规性。
3. 如果你自定义了任何 team-* 技能，审查更新后的版本 — 无参数守卫和 `Verdict:` 关键词现在是所有 team-* 技能必需的。

---

## v0.3.0 → v0.4.0

**发布日期：** 2026-03-21
**Commit 范围：** `b1cad29..HEAD`
**关键主题：** 完整 UX/UI 管线、完整 story 生命周期、brownfield 采用、综合 QA/测试框架、管道完整性、29个新技能

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新技能（17个）** | `/ux-design`、`/ux-review`、`/help`、`/quick-design`、`/review-all-gdds`、`/story-readiness`、`/story-done`、`/sprint-status`、`/adopt`、`/create-architecture`、`/create-control-manifest`、`/create-epics`、`/create-stories`、`/dev-story`、`/propagate-design-change`、`/content-audit`、`/architecture-review` |
| **新 QA 技能（12个）** | `/qa-plan`、`/smoke-check`、`/soak-test`、`/regression-suite`、`/test-setup`、`/test-helpers`、`/test-evidence-review`、`/test-flakiness`、`/skill-test`、`/bug-triage`、`/team-live-ops`、`/team-qa` |
| **新 hooks（4个）** | `log-agent-stop.sh` — agent 审计追踪停止；`notify.sh` — Windows toast 通知；`post-compact.sh` — 压缩后会话恢复提醒；`validate-skill-change.sh` — 技能编辑后 advise `/skill-test` |
| **新模板（8个）** | `ux-spec.md`、`hud-design.md`、`accessibility-requirements.md`、`interaction-pattern-library.md`、`player-journey.md`、`difficulty-curve.md`，以及2个 adoption 计划模板 |
| **新基础设施** | `workflow-catalog.yaml`（7阶段管道，被 `/help` 读取）、`docs/architecture/tr-registry.yaml`（稳定 TR-ID）、`production/sprint-status.yaml` schema |
| **技能更新** | `/gate-check` — 3个 gates 现在需要 UX 产物；Pre-Production gate 需要垂直切片（HARD gate） |
| **技能更新** | `/sprint-plan` — 写入 `sprint-status.yaml`；`/sprint-status` 读取它 |
| **技能更新** | `/story-done` — 8阶段完成审查，更新 story 文件，surfaces 下一个就绪 story |
| **技能更新** | `/design-review` — 移除架构差距检查（错误的阶段） |
| **技能更新** | `/team-ui` — 完整 UX 管道（ux-design → ux-review → team 阶段） |
| **Agent 更新** | 14个专家 agents — 添加 `memory: project` |
| **Agent 更新** | `prototyper` — `isolation: worktree`（在隔离的 git 分支中丢弃工作） |
| **模型路由** | Haiku/Sonnet/Opus 层级分配在协调规则中记录；技能在 frontmatter 中声明其层级 |
| **目录 CLAUDE.md** | 搭建 `design/CLAUDE.md`、`src/CLAUDE.md`、`docs/CLAUDE.md` — 每个目录的路径作用域指令 |
| **管道完整性** | TR-ID 稳定性、manifest 版本控制、ADR 状态 gates、TR-ID 引用而非引用 |
| **GDD 模板** | 添加 `## Game Feel` 章节（输入响应性、动画目标、冲击时刻） |

---

### 文件：可安全覆盖

**新增文件：**
```
.claude/skills/ux-design/SKILL.md
.claude/skills/ux-review/SKILL.md
.claude/skills/help/SKILL.md
.claude/skills/quick-design/SKILL.md
.claude/skills/review-all-gdds/SKILL.md
.claude/skills/story-readiness/SKILL.md
.claude/skills/story-done/SKILL.md
.claude/skills/sprint-status/SKILL.md
.claude/skills/adopt/SKILL.md
.claude/skills/create-architecture/SKILL.md
.claude/skills/create-control-manifest/SKILL.md
.claude/skills/create-epics/SKILL.md
.claude/skills/create-stories/SKILL.md
.claude/skills/dev-story/SKILL.md
.claude/skills/propagate-design-change/SKILL.md
.claude/skills/content-audit/SKILL.md
.claude/skills/architecture-review/SKILL.md
.claude/skills/qa-plan/SKILL.md
.claude/skills/smoke-check/SKILL.md
.claude/skills/soak-test/SKILL.md
.claude/skills/regression-suite/SKILL.md
.claude/skills/test-setup/SKILL.md
.claude/skills/test-helpers/SKILL.md
.claude/skills/test-evidence-review/SKILL.md
.claude/skills/test-flakiness/SKILL.md
.claude/skills/skill-test/SKILL.md
.claude/skills/bug-triage/SKILL.md
.claude/skills/team-live-ops/SKILL.md
.claude/skills/team-qa/SKILL.md
.claude/hooks/log-agent-stop.sh
.claude/hooks/notify.sh
.claude/hooks/post-compact.sh
.claude/hooks/validate-skill-change.sh
.claude/docs/workflow-catalog.yaml
.claude/docs/templates/ux-spec.md
.claude/docs/templates/hud-design.md
.claude/docs/templates/accessibility-requirements.md
.claude/docs/templates/interaction-pattern-library.md
.claude/docs/templates/player-journey.md
.claude/docs/templates/difficulty-curve.md
design/CLAUDE.md
src/CLAUDE.md
docs/CLAUDE.md
```

**现有文件覆盖（无用户内容）：**
```
.claude/skills/gate-check/SKILL.md
.claude/skills/sprint-plan/SKILL.md
.claude/skills/sprint-status/SKILL.md
.claude/skills/design-review/SKILL.md
.claude/skills/team-ui/SKILL.md
.claude/skills/story-readiness/SKILL.md
.claude/skills/story-done/SKILL.md
.claude/docs/templates/game-design-document.md    ← 添加 Game Feel 章节
README.md
docs/WORKFLOW-GUIDE.md
UPGRADING.md
```

**Agent 文件覆盖**（如果你没有写入自定义提示）：
```
.claude/agents/prototyper.md         ← 添加 isolation: worktree
.claude/agents/art-director.md       ← 添加 memory: project
.claude/agents/audio-director.md     ← 添加 memory: project
.claude/agents/economy-designer.md   ← 添加 memory: project
.claude/agents/game-designer.md      ← 添加 memory: project
.claude/agents/gameplay-programmer.md ← 添加 memory: project
.claude/agents/lead-programmer.md    ← 添加 memory: project
.claude/agents/level-designer.md     ← 添加 memory: project
.claude/agents/narrative-director.md ← 添加 memory: project
.claude/agents/systems-designer.md   ← 添加 memory: project
.claude/agents/technical-artist.md   ← 添加 memory: project
.claude/agents/ui-programmer.md      ← 添加 memory: project
.claude/agents/ux-designer.md        ← 添加 memory: project
.claude/agents/world-builder.md      ← 添加 memory: project
```

---

### 文件：仔细合并

#### `.claude/settings.json`

此版本注册了四个新 hooks。如果你没有自定义 `settings.json`，覆盖是安全的。否则，手动添加以下 hook 条目：

- `log-agent-stop.sh` — `SubagentStop` 事件（agent 审计追踪停止）
- `notify.sh` — `Notification` 事件（Windows toast 通知）
- `post-compact.sh` — `PostCompact` 事件（会话恢复提醒）
- `validate-skill-change.sh` — `PostToolUse` 事件过滤到 `.claude/skills/` 写入

#### 自定义 agent 文件

如果你向 agent `.md` 文件添加了项目特定知识，做 diff 并手动将 `memory: project` 行添加到适当的 YAML frontmatter。Creative 和 technical director agents 故意保留 `memory: user` — 只有专家 agents 获取 `memory: project`。

---

### 新功能

#### 完整 Story 生命周期

Stories 现在有由两个技能强制执行的正式生命周期：

- **`/story-readiness`** — 在开发者领取之前验证 story 是否可实施。检查 Design（GDD req 链接）、Architecture（ADR 已接受）、Scope（标准可测试）和 DoD（manifest 版本当前）。裁决：READY / NEEDS WORK / BLOCKED。
- **`/story-done`** — 实施后的8阶段完成审查。验证每个验收标准，检查 GDD/ADR 偏差，提示代码审查，将 story 文件更新为 `Status: Complete`，并 surfaces 下一个就绪 story。

流程：`/story-readiness` → 实施 → `/story-done` → 下一个 story

#### 完整 UX/UI 管道

- **`/ux-design`** — 引导式逐节 UX 规格创作。三种模式：screen/flow、HUD 或交互模式库。读取 GDD UI 需求和玩家旅程。输出到 `design/ux/`。
- **`/ux-review`** — 根据 GDD 对齐、无障碍层级和模式库验证 UX 规格。裁决：APPROVED / NEEDS REVISION / MAJOR REVISION。
- **`/team-ui` 更新：** 阶段1现在运行 `/ux-design` + `/ux-review` 作为硬 gate，在视觉设计开始之前。

#### Brownfield 采用

**`/adopt`** 使现有项目适应模板格式。审计 GDD、ADR、stories、systems-index 和 infra 的内部结构。分类差距（BLOCKING/HIGH/MEDIUM/LOW）。构建有序迁移计划。永不重新生成现有产物 — 仅填补差距。

参数模式：`full | gdds | adrs | stories | infra`

还有：`/design-system retrofit [path]` 和 `/architecture-decision retrofit [path]` 检测现有文件并仅添加缺失的章节。

#### Sprint 追踪 YAML

`production/sprint-status.yaml` 现在是权威的 story 追踪格式：
- 由 `/sprint-plan`（初始化所有 stories）和 `/story-done`（设置状态为 `done`）写入
- 由 `/sprint-status`（快速快照）和 `/help`（生产阶段每个 story 状态）读取
- 状态值：`backlog | ready-for-dev | in-progress | review | done | blocked`
- 如果文件不存在，优雅降级到 markdown 扫描

#### `/help` — 上下文感知下一步

`/help` 读取你当前阶段和进行中的工作，检查哪些产物已完成，并准确告诉你下一步做什么 — 一个主要必需步骤，加上可选机会。区别于 `/start`（仅首次）和 `/project-stage-detect`（完整审计）。

#### 综合 QA 和测试框架

涵盖完整测试生命周期的9个新 QA/测试技能：

- **`/test-setup`** — 为你的引擎搭建测试框架和 CI/CD 管道
- **`/test-helpers`** — 生成引擎特定测试辅助库（GDUnit4、NUnit 等）
- **`/qa-plan`** — 为 sprint 或 feature 生成 QA 测试计划，按测试类型分类 stories
- **`/smoke-check`** — 在 QA hand-off 之前运行关键路径冒烟测试 gate
- **`/soak-test`** — 生成扩展游戏会话的浸泡测试协议（稳定性、内存泄漏）
- **`/regression-suite`** — 将测试覆盖率映射到 GDD 关键路径，识别缺少回归测试的已修复 bugs
- **`/test-evidence-review`** — 测试文件和手动证据文档的质量审查
- **`/test-flakiness`** — 通过读取 CI 运行日志检测非确定性测试
- **`/skill-test`** — 验证技能文件的结构性合规性和行为正确性（三种模式：lint、spec、catalog）

还有新功能：**`/bug-triage`** 重新评估所有开放 bugs 的优先级、严重性和所有权。

#### 技能验证器（`/skill-test`）

`/skill-test` 是一个用于验证工具本身的 meta-skill。在编辑任何技能文件后运行它。三种模式：
- `lint` — 验证 YAML frontmatter 和必需字段
- `spec [skill-name]` — 针对特定技能运行行为规格测试
- `catalog` — 检查 `.claude/skills/` 中的所有技能是否在目录中索引

新 `validate-skill-change.sh` hook 在技能文件被修改时自动提醒你运行 `/skill-test`。

#### Team Live-Ops 和 Team QA 编排

- **`/team-live-ops`** — 协调 live-ops-designer + economy-designer + community-manager + analytics-engineer 进行发布后内容规划（季节性活动、战斗通行证、留存）
- **`/team-qa`** — 编排 qa-lead + qa-tester + gameplay-programmer + producer 完成完整 QA 循环：策略、执行、覆盖率和 sign-off

#### 模型层级路由

技能现在根据任务复杂度明确分配到 Haiku、Sonnet 或 Opus 层级。只读状态检查使用 Haiku；复杂多文档综合使用 Opus；其他所有内容默认为 Sonnet。层级分配记录在 `.claude/docs/coordination-rules.md` 中。

#### 目录 CLAUDE.md 文件

三个新的目录作用域 CLAUDE.md 文件（`design/`、`src/`、`docs/`）为在这些目录工作的 agents 提供路径特定指令。这些在 Claude Code 读取该目录中的文件时自动加载。

---

### 升级后

1. **验证新 hooks** 已在 `.claude/settings.json` 中注册 — 检查所有四个：`log-agent-stop.sh`、`notify.sh`、`post-compact.sh`、`validate-skill-change.sh`。

2. **测试审计追踪** 通过派生任何 subagent — 开始和停止事件都应出现在 `production/session-logs/` 中。

3. **如果你在积极生产中，生成 sprint-status.yaml：**
   ```
   /sprint-plan status
   ```

4. **如果你有早于本模板版本且存在的 GDD 或 ADR，运行 `/adopt`** — 它将识别需要添加哪些章节而不覆盖你的内容。

5. **任何技能编辑后用 `/skill-test` 验证你的技能** — 新 `validate-skill-change.sh` hook 将自动提醒你这样做。

---

## v0.2.0 → v0.3.0

**发布日期：** 2026-03-09
**Commit 范围：** `e289ce9..HEAD`
**关键主题：** `/design-system` GDD 创作、`/map-systems` 重命名、自定义状态行

### 破坏性变更

#### `/design-systems` 重命名为 `/map-systems`

`/design-systems` 技能被重命名为 `/map-systems` 以提高清晰度（分解 = *mapping*，不是 *designing*）。

**需要操作：** 更新任何调用 `/design-systems` 的文档、笔记或脚本。新的调用是 `/map-systems`。

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新技能** | `/design-system`（引导式 GDD 创作，逐节） |
| **重命名的技能** | `/design-systems` → `/map-systems`（破坏性重命名） |
| **新文件** | `.claude/statusline.sh`、`.claude/settings.json` statusline 配置 |
| **技能更新** | `/gate-check` — 在 PASS 时写入 `production/stage.txt`，新阶段定义 |
| **技能更新** | `brainstorm`、`start`、`design-review`、`project-stage-detect`、`setup-engine` — 交叉引用修复 |
| **Bug 修复** | `log-agent.sh`、`validate-commit.sh` — hook 执行修复 |
| **文档** | 添加 `UPGRADING.md`、`README.md` 更新、`WORKFLOW-GUIDE.md` 更新 |

---

### 文件：可安全覆盖

**新增文件：**
```
.claude/skills/design-system/SKILL.md
.claude/statusline.sh
```

**现有文件覆盖（无用户内容）：**
```
.claude/skills/map-systems/SKILL.md      ← 原 design-systems/SKILL.md
.claude/skills/gate-check/SKILL.md
.claude/skills/brainstorm/SKILL.md
.claude/skills/start/SKILL.md
.claude/skills/design-review/SKILL.md
.claude/skills/project-stage-detect/SKILL.md
.claude/skills/setup-engine/SKILL.md
.claude/hooks/log-agent.sh
.claude/hooks/validate-commit.sh
README.md
docs/WORKFLOW-GUIDE.md
UPGRADING.md
```

**删除（被重命名替换）：**
```
.claude/skills/design-systems/   ← 整个目录；被 map-systems/ 替换
```

---

### 文件：仔细合并

#### `.claude/settings.json`

新版本添加了指向 `.claude/statusline.sh` 的 `statusLine` 配置块。如果你没有自定义 `settings.json`，覆盖是安全的。否则，手动添加此块：

```json
"statusLine": {
  "script": ".claude/statusline.sh"
}
```

---

### 新功能

#### 自定义状态行

`.claude/statusline.sh` 在终端状态行显示7阶段生产管道面包屑：

```
ctx: 42% | claude-sonnet-4-6 | Systems Design
```

在 Production/Polish/Release 阶段，如果 `<!-- STATUS -->` 块存在，它还显示来自 `production/session-state/active.md` 的活动 Epic/Feature/Task：

```
ctx: 42% | claaude-sonnet-4-6 | Production | Combat System > Melee Combat > Hitboxes
```

当前阶段从项目产物自动检测，或者可以通过将阶段名称写入 `production/stage.txt` 来固定。

#### `/gate-check` 阶段推进

当 gate PASS 裁决被确认时，`/gate-check` 现在将新阶段名称写入 `production/stage.txt`。这立即为所有未来会话更新状态行，无需手动文件编辑。

---

### 升级后

1. **删除旧的技能目录：**
   ```bash
   rm -rf .claude/skills/design-systems/
   ```

2. **测试状态行** 通过启动 Claude Code 会话 — 你应在终端 footer 中看到阶段面包屑。

3. **验证 hook 执行仍然有效：**
   ```bash
   bash .claude/hooks/log-agent.sh '{}' '{}'
   bash .claude/hooks/validate-commit.sh '{}' '{}'
   ```

---

## v0.1.0 → v0.2.0

**发布日期：** 2026-02-21
**Commit 范围：** `ad540fe..e289ce9`
**关键主题：** 上下文弹性、AskUserQuestion 集成、`/map-systems` 技能

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新技能** | `/start`（onboarding）、`/map-systems`（系统分解）、`/design-system`（引导式 GDD 创作） |
| **新 hooks** | `session-start.sh`（恢复）、`detect-gaps.sh`（差距检测） |
| **新模板** | `systems-index.md`、3个协作协议模板 |
| **上下文管理** | 主要重写 — 添加文件支持的状态策略 |
| **Agent 更新** | 14个设计/创意 agents — AskUserQuestion 集成 |
| **技能更新** | 所有7个 `team-*` 技能 + `brainstorm` — 阶段转换时的 AskUserQuestion |
| **CLAUDE.md** | 从约159行精简到约60行；5个文档导入而不是10个 |
| **Hook 更新** | 所有8个 hooks — Windows 兼容性修复、新功能 |
| **文档移除** | `docs/IMPROVEMENTS-PROPOSAL.md`、`docs/MULTI-STAGE-DOCUMENT-WORKFLOW.md` |

---

### 文件：可安全覆盖

这些是纯基础设施 — 你没有自定义它们。直接复制新版本，不会有项目内容风险。

**新增文件：**
```
.claude/skills/start/SKILL.md
.claude/skills/map-systems/SKILL.md
.claude/skills/design-system/SKILL.md
.claude/docs/templates/systems-index.md
.claude/docs/templates/collaborative-protocols/design-agent-protocol.md
.claude/docs/templates/collaborative-protocols/implementation-agent-protocol.md
.claude/docs/templates/collaborative-protocols/leadership-agent-protocol.md
.claude/hooks/detect-gaps.sh
.claude/hooks/session-start.sh
production/session-state/.gitkeep
docs/examples/README.md
.github/ISSUE_TEMPLATE/bug_report.md
.github/ISSUE_TEMPLATE/feature_request.md
.github/PULL_REQUEST_TEMPLATE.md
```

**现有文件覆盖（无用户内容）：**
```
.claude/skills/brainstorm/SKILL.md
.claude/skills/design-review/SKILL.md
.claude/skills/gate-check/SKILL.md
.claude/skills/project-stage-detect/SKILL.md
.claude/skills/setup-engine/SKILL.md
.claude/skills/team-audio/SKILL.md
.claude/skills/team-combat/SKILL.md
.claude/skills/team-level/SKILL.md
.claude/skills/team-narrative/SKILL.md
.claude/skills/team-polish/SKILL.md
.claude/skills/team-release/SKILL.md
.claude/skills/team-ui/SKILL.md
.claude/hooks/log-agent.sh
.claude/hooks/pre-compact.sh
.claude/hooks/session-stop.sh
.claude/hooks/validate-assets.sh
.claude/hooks/validate-commit.sh
.claude/hooks/validate-push.sh
.claude/rules/design-docs.md
.claude/docs/hooks-reference.md
.claude/docs/skills-reference.md
.claude/docs/quick-start.md
.claude/docs/directory-structure.md
.claude/docs/context-management.md
docs/COLLABORATIVE-DESIGN-PRINCIPLE.md
docs/WORKFLOW-GUIDE.md
README.md
```

**Agent 文件覆盖**（如果你没有写入自定义提示）：
```
.claude/agents/art-director.md
.claude/agents/audio-director.md
.claude/agents/creative-director.md
.claude/agents/economy-designer.md
.claude/agents/game-designer.md
.claude/agents/level-designer.md
.claude/agents/live-ops-designer.md
.claude/agents/narrative-director.md
.claude/agents/producer.md
.claude/agents/systems-designer.md
.claude/agents/technical-director.md
.claude/agents/ux-designer.md
.claude/agents/world-builder.md
.claude/agents/writer.md
```

如果你*已*自定义 agent 提示，见下面的"仔细合并"。

---

### 文件：仔细合并

这些文件同时包含模板结构和你的项目特定内容。
**不要**覆盖它们 — 手动合并更改。

#### `CLAUDE.md`

模板版本从约159行精简到约60行。关键结构性变更：移除了5个文档导入，因为它们被 Claude Code 自动加载（agent-roster、skills-reference、hooks-reference、rules-reference、review-workflow）。

**从你的版本保留：**
- `## Technology Stack` 章节（你的引擎/语言选择）
- 你做的任何项目特定添加

**从新版本采用：**
- 更精简的导入列表（如果存在，删除5个冗余的 `@` 导入）
- 更新的协作协议措辞

#### `.claude/docs/technical-preferences.md`

如果你运行了 `/setup-engine`，此文件有你的引擎配置、命名约定和性能预算。保留所有内容。模板版本只是空占位符。

#### `.claude/docs/templates/game-concept.md`

微小结构性更新 — 添加了指向 `/map-systems` 的 `## Next Steps` 章节。如果你想获得更新后的指导，将该章节添加到你自己的副本，但不是必需的。

#### `.claude/settings.json`

检查新版本是否添加了你想要的任何权限规则。变更很小（schema 更新）。如果你没有自定义你的 `settings.json`，覆盖是安全的。

#### 自定义 agent 文件

如果你向任何 agent `.md` 文件添加了项目特定知识或自定义行为，做 diff 并手动添加新的 AskUserQuestion 集成章节，而不是覆盖。每个 agent 中的变更是系统提示末尾标准化的协作协议块。

---

### 文件：删除

这些文件在 v0.2.0 中被移除。如果存在于你的仓库中，你可以安全删除它们 — 它们被更好的组织替代方案替换。

```
docs/IMPROVEMENTS-PROPOSAL.md      → 被 WORKFLOW-GUIDE.md 取代
docs/MULTI-STAGE-DOCUMENT-WORKFLOW.md → 内容合并到 context-management.md
```

---

### 升级后

1. **运行 `/project-stage-detect`** 验证系统用新的检测逻辑正确读取你的项目。

2. **如果你没有使用过，运行 `/start` 一次** — 它现在正确识别你的阶段并跳过你已完成的上线步骤。

3. **检查 `production/session-state/` 存在且被 gitignored：**
   ```bash
   ls production/session-state/
   cat .gitignore | grep session-state
   ```

4. **测试 hook 执行** — 如果你在 Windows 上，验证新 hooks 在 Git Bash 中运行无错误：
   ```bash
   bash .claude/hooks/detect-gaps.sh '{}' '{}'
   bash .claude/hooks/session-start.sh '{}' '{}'
   ```

---

*每个未来版本将在本文件中拥有自己的章节。*
