<p align="center">
  <h1 align="center">Claude Code Game Studios</h1>
  <p align="center">
    将一个 Claude Code 会话转变为完整的游戏开发工作室。
    <br />
    49 个 Agent。73 项技能。一个协调的 AI 团队。
  </p>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
  <a href=".claude/agents"><img src="https://img.shields.io/badge/agents-49-blueviolet" alt="49 Agents"></a>
  <a href=".claude/skills"><img src="https://img.shields.io/badge/skills-73-green" alt="73 Skills"></a>
  <a href=".claude/hooks"><img src="https://img.shields.io/badge/hooks-12-orange" alt="12 Hooks"></a>
  <a href=".claude/rules"><img src="https://img.shields.io/badge/rules-11-red" alt="11 Rules"></a>
  <a href="https://docs.anthropic.com/en/docs/claude-code"><img src="https://img.shields.io/badge/built%20for-Claude%20Code-f5f5f5?logo=anthropic" alt="Built for Claude Code"></a>
  <a href="https://www.buymeacoffee.com/donchitos3"><img src="https://img.shields.io/badge/Buy%20Me%20a%20Coffee-Support%20this%20project-FFDD00?logo=buymeacoffee&logoColor=black" alt="Buy Me a Coffee"></a>
  <a href="https://github.com/sponsors/Donchitos"><img src="https://img.shields.io/badge/GitHub%20Sponsors-Support%20this%20project-ea4aaa?logo=githubsponsors&logoColor=white" alt="GitHub Sponsors"></a>
</p>

---

## 为什么存在这个项目

用 AI 单独开发游戏很强大——但单一聊天会话没有结构。没有人阻止你硬编码魔法数字、跳过设计文档或写出意大利面条代码。没有 QA 审查，没有设计评审，没有人问"这真的符合游戏愿景吗？"

**Claude Code Game Studios** 通过给你的 AI 会话一个真实工作室的结构来解决这个问题。不是单一通用助手，而是 49 个专业 Agent，按照工作室层级组织——守护愿景的导演、负责各自领域的主管、以及进行实际工作的专家。每个 Agent 都有明确的职责、升级路径和质量门控。

结果是：你仍然做每个决定，但现在你有一个团队来提出正确的问题、尽早发现错误，并在从第一次头脑风暴到发布的整个过程中保持项目有序。

---

## 目录

- [包含内容](#包含内容)
- [工作室层级](#工作室层级)
- [Slash Commands](#slash-commands)
- [开始使用](#开始使用)
- [升级](#升级)
- [项目结构](#项目结构)
- [工作原理](#工作原理)
- [设计理念](#设计理念)
- [自定义](#自定义)
- [平台支持](#平台支持)
- [社区](#社区)
- [支持这个项目](#支持这个项目)
- [许可证](#许可证)

---

## 包含内容

| 类别 | 数量 | 描述 |
|----------|-------|-------------|
| **Agents** | 49 | 跨越设计、编程、美术、音频、叙事、QA 和制作的专职子 Agent |
| **Skills** | 73 | 覆盖每个工作流程阶段的 Slash 命令（`/start`、`/design-system`、`/create-epics`、`/create-stories`、`/dev-story`、`/story-done` 等） |
| **Hooks** | 12 | 在提交、推送、资源更改、会话生命周期、Agent 审计追踪和差距检测上自动验证 |
| **Rules** | 11 | 编辑游戏玩法、引擎、AI、UI、网络代码等时执行的路径作用域编码标准 |
| **Templates** | 41 | GDD、UX 规范、ADR、Sprint 计划、HUD 设计、无障碍等文档模板 |

## 工作室层级

Agent 按三个层级组织，与真实工作室的运作方式一致：

```
第一层 — 导演（Opus）
  creative-director    technical-director    producer

第二层 — 部门主管（Sonnet）
  game-designer        lead-programmer       art-director
  audio-director       narrative-director    qa-lead
  release-manager      localization-lead

第三层 — 专家（Sonnet/Haiku）
  gameplay-programmer  engine-programmer     ai-programmer
  network-programmer   tools-programmer     ui-programmer
  systems-designer     level-designer       economy-designer
  technical-artist     sound-designer       writer
  world-builder        ux-designer          prototyper
  performance-analyst  devops-engineer      analytics-engineer
  security-engineer   qa-tester            accessibility-specialist
  live-ops-designer   community-manager
```

### 引擎专家

模板包含三个主流引擎的 Agent 集合。使用与你的项目匹配的集合：

| 引擎 | 主管 Agent | 子专家 |
|--------|-----------|-----------------|
| **Godot 4** | `godot-specialist` | GDScript、Shaders、GDExtension |
| **Unity** | `unity-specialist` | DOTS/ECS、Shaders/VFX、Addressables、UI Toolkit |
| **Unreal Engine 5** | `unreal-specialist` | GAS、Blueprints、Replication、UMG/CommonUI |

## Slash Commands

在 Claude Code 中输入 `/` 访问全部 73 项技能：

**入门与导航**
`/start` `/help` `/project-stage-detect` `/setup-engine` `/adopt`

**游戏设计**
`/brainstorm` `/map-systems` `/design-system` `/quick-design` `/review-all-gdds` `/propagate-design-change`

**美术与资源**
`/art-bible` `/asset-spec` `/asset-audit`

**UX 与界面设计**
`/ux-design` `/ux-review`

**架构**
`/create-architecture` `/architecture-decision` `/architecture-review` `/create-control-manifest`

**Stories 与 Sprints**
`/create-epics` `/create-stories` `/dev-story` `/sprint-plan` `/sprint-status` `/story-readiness` `/story-done` `/estimate`

**评审与分析**
`/design-review` `/code-review` `/balance-check` `/content-audit` `/scope-check` `/perf-profile` `/tech-debt` `/gate-check` `/consistency-check` `/security-audit`

**QA 与测试**
`/qa-plan` `/smoke-check` `/soak-test` `/regression-suite` `/test-setup` `/test-helpers` `/test-evidence-review` `/test-flakiness` `/skill-test` `/skill-improve`

**制作**
`/milestone-review` `/retrospective` `/bug-report` `/bug-triage` `/reverse-document` `/playtest-report`

**发布**
`/release-checklist` `/launch-checklist` `/changelog` `/patch-notes` `/hotfix` `/day-one-patch`

**创意与内容**
`/prototype` `/onboard` `/localize`

**团队编排**（协调单个功能上的多个 Agent）
`/team-combat` `/team-narrative` `/team-ui` `/team-release` `/team-polish` `/team-audio` `/team-level` `/team-live-ops` `/team-qa`

## 开始使用

### 前提条件

- [Git](https://git-scm.com/)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)（`npm install -g @anthropic-ai/claude-code`）
- **推荐**: [jq](https://jqlang.github.io/jq/)（用于 Hook 验证）和 Python 3（用于 JSON 验证）

所有 Hook 在缺少可选工具时会优雅失败——不会破坏任何东西，只是失去验证功能。

### 设置

1. **克隆或用作模板**:
   ```bash
   git clone https://github.com/Donchitos/Claude-Code-Game-Studios.git my-game
   cd my-game
   ```

2. **打开 Claude Code** 并启动会话:
   ```bash
   claude
   ```

3. **运行 `/start`** — 系统询问你目前处于哪个阶段（没有想法、模糊概念、清晰设计、现有工作）并引导你到正确的工作流程。没有假设。

   或者直接跳转到特定的技能（如果你已经知道需要什么）：
   - `/brainstorm` — 从头探索游戏创意
   - `/setup-engine godot 4.6` — 如果你已经知道则配置你的引擎
   - `/project-stage-detect` — 分析现有项目

## 升级

已经在使用旧版本这个模板？参见 [UPGRADING.md](UPGRADING.md)
获取逐步迁移说明、版本间变更明细，以及哪些文件可以安全覆盖、哪些需要手动合并。

## 项目结构

```
CLAUDE.md                           # 主配置
.claude/
  settings.json                     # Hooks、权限、安全规则
  agents/                           # 49 个 Agent 定义（Markdown + YAML frontmatter）
  skills/                           # 73 个 Slash 命令（每个技能一个子目录）
  hooks/                            # 12 个 Hook 脚本（Bash，跨平台）
  rules/                            # 11 个路径作用域编码标准
  statusline.sh                     # 状态行脚本（上下文%、模型、阶段、Epic 面包屑）
  docs/
    workflow-catalog.yaml           # 7 阶段管线定义（由 /help 读取）
    templates/                      # 41 个文档模板
src/                                # 游戏源代码
assets/                             # 美术、音频、VFX、Shader、数据文件
design/                             # GDD、叙事文档、关卡设计
docs/                               # 技术文档和 ADR
tests/                              # 测试套件（单元、集成、性能、试玩）
tools/                              # 构建和管线工具
prototypes/                         # 一次性原型（与 src/ 隔离）
production/                         # Sprint 计划、里程碑、发布追踪
```

## 工作原理

### Agent 协调

Agent 遵循结构化委托模型：

1. **垂直委托** — 导演委托给主管，主管委托给专家
2. **水平协商** — 同级 Agent 可以互相协商但不能做出跨域约束性决策
3. **冲突解决** — 分歧升级到共同上级（设计问题升级到 `creative-director`，技术问题升级到 `technical-director`）
4. **变更传播** — 跨部门变更由 `producer` 协调
5. **领域边界** — Agent 未经明确委托不得修改其领域外的文件

### 协作，而非自主

这不是自动导航系统。每个 Agent 遵循严格的协作协议：

1. **提问** — Agent 在提出解决方案前先提问
2. **展示选项** — Agent 展示 2-4 个带优缺点的选项
3. **你来决定** — 用户始终做决定
4. **草稿** — Agent 在定稿前展示工作
5. **审批** — 未经你的签字不会写入任何内容

你保持控制。Agent 提供结构和专业知识，而非自主性。

### 自动化安全

**Hooks** 在每个会话上自动运行：

| Hook | 触发条件 | 功能 |
|------|---------|--------------|
| `validate-commit.sh` | PreToolUse (Bash) | 检查硬编码值、TODO 格式、JSON 有效性、设计文档章节 — 如果命令不是 `git commit` 则提前退出 |
| `validate-push.sh` | PreToolUse (Bash) | 警告推送到受保护分支 — 如果命令不是 `git push` 则提前退出 |
| `validate-assets.sh` | PostToolUse (Write/Edit) | 验证命名约定和 JSON 结构 — 如果文件不在 `assets/` 中则提前退出 |
| `session-start.sh` | 会话打开 | 显示当前分支和最近提交以供定向 |
| `detect-gaps.sh` | 会话打开 | 检测新项目（建议 `/start`）以及代码或原型存在时缺少设计文档的情况 |
| `pre-compact.sh` | 压缩前 | 保存会话进度笔记 |
| `post-compact.sh` | 压缩后 | 提醒 Claude 从 `active.md` 恢复会话状态 |
| `notify.sh` | 通知事件 | 通过 PowerShell 显示 Windows Toast 通知 |
| `session-stop.sh` | 会话关闭 | 将 `active.md` 归档到会话日志并记录 git 活动 |
| `log-agent.sh` | Agent 启动 | 审计追踪开始 — 记录子 Agent 调用 |
| `log-agent-stop.sh` | Agent 停止 | 审计追踪停止 — 完成子 Agent 记录 |
| `validate-skill-change.sh` | PostToolUse (Write/Edit) | 建议在任何 `.claude/skills/` 更改后运行 `/skill-test` |

> **注意**: `validate-commit.sh`、`validate-assets.sh` 和 `validate-skill-change.sh` 在每个 Bash/Write 工具调用时触发，当命令或文件路径不相关时立即退出（exit 0）。这是正常的 Hook 行为——不是性能问题。

`settings.json` 中的**权限规则**自动允许安全操作（git status、测试运行）并阻止危险操作（force push、`rm -rf`、读取 `.env` 文件）。

### 路径作用域规则

编码标准根据文件位置自动执行：

| 路径 | 强制执行 |
|------|----------|
| `src/gameplay/**` | 数据驱动值、增量时间使用、无 UI 引用 |
| `src/core/**` | 热路径零分配、线程安全、API 稳定性 |
| `src/ai/**` | 性能预算、可调试性、数据驱动参数 |
| `src/networking/**` | 服务器权威、版本化消息、安全性 |
| `src/ui/**` | 无游戏状态所有权、本地化就绪、无障碍 |
| `design/gdd/**` | 必需 8 个章节、公式格式、边缘情况 |
| `tests/**` | 测试命名、覆盖率要求、fixture 模式 |
| `prototypes/**` | 放宽标准、必需 README、记录假设 |

## 设计理念

本模板基于专业游戏开发现践：

- **MDA 框架** — 用于游戏设计的力学、动态、美学分析
- **自我决定理论** — 用于玩家动机的自主性、能力、关联性
- **心流设计** — 用于玩家参与的能力-挑战平衡
- **Bartle 玩家类型** — 用于受众定位和验证
- **验证驱动开发** — 测试优先，然后实现

## 自定义

这是**模板**，不是锁定框架。一切都可以自定义：

- **添加/删除 Agent** — 删除你不需要的 Agent 文件，为你的领域添加新的
- **编辑 Agent 提示** — 调整 Agent 行为，添加项目特定知识
- **修改技能** — 调整工作流程以匹配你团队的流程
- **添加规则** — 为你的项目目录结构创建新的路径作用域规则
- **调整 Hooks** — 调整验证严格度，添加新的检查
- **选择你的引擎** — 使用 Godot、Unity 或 Unreal Agent 集合（或不使用）
- **设置评审强度** — `full`（所有导演门）、`lean`（仅阶段门）或 `solo`（无）。在 `/start` 期间设置或编辑 `production/review-mode.txt`。使用 `--review solo` 在任何技能上覆盖本次运行。

## 平台支持

已在 **Windows 10** 和 Git Bash 上测试。所有 Hook 使用 POSIX 兼容模式（`grep -E`，不是 `grep -P`）并在缺少工具时包含回退，因此应该能在 macOS 和 Linux 上运行。`notify.sh` hook 使用 PowerShell 显示 Windows Toast 通知，在其他平台上是空操作 — macOS/Linux 的桌面通知尚未接入。跨平台测试正在进行中；如遇任何平台特定的问题请提交 issue。

## 社区

- **讨论区** — [GitHub Discussions](https://github.com/Donchitos/Claude-Code-Game-Studios/discussions) 用于提问、想法和展示你构建的内容
- **问题** — [Bug 报告和功能请求](https://github.com/Donchitos/Claude-Code-Game-Studios/issues)

---

## 支持这个项目

Claude Code Game Studios 是免费开源的。如果它为你节省了时间或帮助你发布游戏，请考虑支持持续开发：

<p>
  <a href="https://www.buymeacoffee.com/donchitos3"><img src="https://img.shields.io/badge/Buy%20Me%20a%20Coffee-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black" alt="Buy Me a Coffee"></a>
  &nbsp;
  <a href="https://github.com/sponsors/Donchitos"><img src="https://img.shields.io/badge/GitHub%20Sponsors-ea4aaa?style=for-the-badge&logo=githubsponsors&logoColor=white" alt="GitHub Sponsors"></a>
</p>

- **[Buy Me a Coffee](https://www.buymeacoffee.com/donchitos3)** — 一次性支持
- **[GitHub Sponsors](https://github.com/sponsors/Donchitos)** — 通过 GitHub 的持续支持

赞助帮助资助维护技能、添加新 Agent、跟上 Claude Code 和引擎 API 变化，以及回应社区问题的投入。

---

*为 Claude Code 构建。通过 [GitHub Discussions](https://github.com/Donchitos/Claude-Code-Game-Studios/discussions) 欢迎贡献。*

## 许可证

MIT License。详见 [LICENSE](LICENSE)。