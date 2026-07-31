# 为 Claude Code Game Studios 做贡献

CCGS 是一个使用 Claude Code 进行独立游戏开发的协调框架。
欢迎贡献 — bug 修复、填补真正缺口的新技能、agent 改进和 hook 修复。
不符合框架方向的 PR 将被关闭，不做详细解释。

## 什么是好的 PR

- **Bug 修复** — 某个东西坏了，这是修复
- **新技能**，解决尚未覆盖的工作流缺口
- **对现有 agent、skill 或 hook 的改进**
- **文档更正** — 错误信息、损坏的引用、过时的步骤

作为 PR 提交的功能请求将被关闭。请改为开一个 issue。

**这个仓库不是什么：**
CCGS 是帮助你构建游戏的系统，不是存储你用 CCGS 构建的游戏的地方。
GDD、ADR、PRD、游戏概念、关卡设计、叙事文档，
或 CCGS 为你的项目生成的任何其他输出都不会被合并到这里 — 请保留在你自己的仓库中。

## 不可协商的技术规则

这些是你如果遗漏就会被 PR 拒绝的东西。

**Skill 文件**
- Skills 位于 `.claude/skills/<name>/SKILL.md` — 子目录格式是必需的。
  扁平的 `.md` 文件会被 Claude Code 静默忽略。
- SKILL.md 必须包含 YAML frontmatter：`name`、`description`、
  `argument-hint`、`allowed-tools` 和 `model`
- 模型层级：`haiku` 用于只读状态检查，`opus` 用于多文档综合和阶段门控，
  `sonnet` 用于其他所有情况

**Hooks**
- 使用 `grep -E` — 永远不要用 `grep -P`（Perl 正则表达式在 Windows Git Bash 上会坏）
- 为没有安装 `jq` 或 `python` 的系统提供回退
- Hooks 在每个会话开始时运行 — 它们必须快速且优雅地退出（`exit 0`）
  当不适用时

**Agents**
- 新 agent 必须包含一个**协作协议**部分，描述 agent 如何提问并将决策交给用户
- Agent 不得在没有明确用户委托的情况下修改其文档化领域之外的文件

**参考文档**
- 如果你的 PR 添加或更改了 skill、agent 或 hook，更新匹配的参考文档
  （agent-roster、skills-reference、hooks-reference 或 rules-reference）。
  添加内容但不更新索引的 PR 将被退回。

## 协作原则

CCGS 不是自主系统。每个工作流遵循：
**提问 → 选项 → 决策 → 草稿 → 审批 → 写入**

Skills 和 agents 必须在行动前询问。没有任何东西在未经用户明确确认的情况下写入文件。
如果你的贡献让 agent 单方面做出决策或写入文件，它将不会被合并。

## 测试你的更改

在 Claude Code 会话中运行它并确认端到端工作。对于 skills，
调用技能并验证输出与技能声称的一致。对于 hooks，
触发相关事件并确认 hook 正确触发并干净退出。

在你的 PR 描述中包含一个简短的说明，描述你测试了什么以及输出是什么样的。

## 提交格式

使用 [Conventional Commits](https://www.conventionalcommits.org/)：

```
feat: add /retrospective skill for end-of-sprint reviews
fix: correct grep -P usage in session-start hook
docs: update skills-reference with new /qa-plan entry
```

类型：`feat`、`fix`、`docs`、`chore`、`refactor`、`test`

## PR 流程

- 你的 PR 将通过 CODEOWNERS 自动分配给维护者
- 审查在发生时进行 — 这是一个单人维护的项目
- 如果你的 PR 在几周内没有反馈，可以发一个提醒评论
- 被合并的贡献者将在发布说明中被署名

## 平台兼容性

CCGS 必须在 Windows（Git Bash）、macOS 和 Linux 上工作。如果你的 hook 或
脚本使用了任何平台特定的内容，它将被拒绝。有疑问时，
在 Windows 上测试。
