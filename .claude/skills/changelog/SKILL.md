---
name: changelog
description: "Auto-generates a changelog from git commits, sprint data, and design documents. Produces both internal 和玩家面向的版本。"
argument-hint: "[version|sprint-number]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write
context: |
  !git log --oneline -30 2>/dev/null
  !git tag --list --sort=-v:refname 2>/dev/null | head -5
model: haiku
---

## 阶段 1：解析参数

读取参数以获取目标版本或 sprint 编号。如果给定版本，使用相应的 git tag。如果给定 sprint 编号，使用 sprint 日期范围。

验证存储库已初始化：运行 `git rev-parse --is-inside-work-tree` 以确认 git 可用。如果不是 git 仓库，通知用户并优雅地中止。

---

## 阶段 2：收集变更数据

读取自上次 tag 或发布以来的 git 日志：

```
git log --oneline [last-tag]..HEAD
```

如果不存在 tag，读取完整日志或合理的最近范围（最近 100 次提交）。

从 `production/sprints/` 读取相关期间的 sprint 报告以了解计划工作和变更背后的上下文。

从 `design/gdd/` 读取已完成的设计文档以了解此期间实现的任何新功能。

---

## 阶段 3：分类变更

将每个变更分类为以下类别之一：

- **New Features**：全新的游戏系统、模式或内容
- **Improvements**：现有功能的增强、UX 改进、性能提升
- **Bug Fixes**：对损坏行为的纠正
- **Balance Changes**：游戏值、难度、经济的调整
- **Known Issues**：团队已知但尚未解决的问题
- **Miscellaneous**：不符合上述类别的变更，或提交消息太模糊而无法自信分类的变更

对于每个提交，检查消息是否包含任务 ID 或故事引用
（例如 `[STORY-123]`、`TR-`、`#NNN` 或类似）。计算缺少任何任务引用的提交数量
并在阶段 4 指标部分包含此计数为：`Commits without task reference: [N]`。

---

## 阶段 4：生成内部 Changelog

```markdown
# Internal Changelog: [Version]
Date: [Date]
Sprint(s): [Sprint numbers covered]
Commits: [Count] ([first-hash]..[last-hash])

## New Features
- [Feature Name] —— [技术描述，受影响的系统]
  - Commits: [hash1], [hash2]
  - Owner: [实现者]
  - Design doc: [如果适用，链接]

## Improvements
- [Improvement] —— [技术上改变了什么以及为什么]
  - Commits: [hashes]
  - Owner: [who]

## Bug Fixes
- [BUG-ID] [bug 和根本原因的描述]
  - Fix: [改变了什么]
  - Commits: [hashes]
  - Owner: [who]

## Balance Changes
- [调整了什么] —— [旧值 -> 新值] —— [设计意图]
  - Owner: [who]

## Technical Debt / Refactoring
- [清理了什么以及为什么]
  - Commits: [hashes]

## Miscellaneous
- [不符合其他类别的变更，或模糊的提交消息]
  - Commits: [hashes]

## Known Issues
- [问题描述] —— [严重性] —— [如果已知，修复的 ETA]

## Metrics
- Total commits: [N]
- Files changed: [N]
- Lines added: [N]
- Lines removed: [N]
- Commits without task reference: [N]
```

---

## 阶段 5：生成玩家面向的 Changelog

```markdown
# What is New in [Version]

## New Features
- **[Feature Name]**: [玩家可以做什么以及为什么令人兴奋的玩家友好描述。
  专注于体验，而不是实现。]

## Improvements
- **[改进了什么]**: [这对玩家来说如何让游戏更好。
  具体但避免行话。]

## Bug Fixes
- Fixed an issue where [描述玩家体验了什么，而不是
  代码中什么错了]
- Fixed [玩家可见的症状]

## Balance Changes
- [以玩家可理解的术语改变了什么以及设计意图。
  示例："Healing potions now restore 50 HP (up from 30) —— we felt
  players needed more recovery options in late-game encounters."]

## Known Issues
- We are aware of [以玩家术语描述的问题] and are working on a
  fix. [如果存在解决方法。]

---
Thank you for playing! Your feedback helps us make the game better.
Report issues at [link].
```

---

## 阶段 6：输出

向用户输出两个 changelog。内部 changelog 是主要工作文档。玩家面向的 changelog 在审查后准备好供社区发布。

---

## 阶段 7：提供文件写入

展示 changelog 后，询问用户：

> "我可以将此 changelog 写入 `docs/CHANGELOG.md` 吗？
> [A] 是，追加此条目（如果文件已存在则推荐）
> [B] 是，完全覆盖文件
> [C] 不 —— 我会手动复制"

- 在询问之前检查 `docs/CHANGELOG.md` 是否存在。如果存在，默认推荐
  为 **[A] 追加**。
- 如果用户选择 [A]：将新的内部 changelog 条目追加到现有文件的顶部
  （最新条目在前）。
- 如果用户选择 [B]：用新的 changelog 覆盖文件。
- 如果用户选择 [C]: 在此停止，不写入。

成功写入后：裁决：**CHANGELOG WRITTEN** —— changelog 已保存到 `docs/CHANGELOG.md`。
如果用户拒绝：裁决：**COMPLETE** —— changelog 已生成。

---

## 阶段 7：下一步

- 使用 `/patch-notes [version]` 生成用于公开发布的样式化、保存版本。
- 在外部发布 changelog 之前使用 `/release-checklist`。

### 指南

- 永远不要在玩家面向的 changelog 中暴露内部代码引用、文件路径或开发者名称
- 将相关变更分组在一起，而不是列出单个提交
- 如果提交消息不清楚，检查相关文件和 sprint 数据以获取上下文
- 平衡变更应始终包括设计推理，而不仅仅是数字
- 已知问题应该诚实 —— 玩家欣赏透明度
- 如果 git 历史混乱（合并提交、恢复、修复提交），清理叙事而不是逐字列出每个提交
