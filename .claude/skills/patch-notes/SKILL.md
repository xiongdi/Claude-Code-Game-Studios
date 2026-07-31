---
name: patch-notes
description: "Generate player-facing patch notes from git history, sprint data, and internal changelogs. Translates developer language into clear, engaging player communication."
argument-hint: "[version] [--style brief|detailed|full]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Bash
model: haiku
agent: community-manager
---

## Phase 1: 解析参数

- `version`: 要为其生成说明的发布版本（例如 `1.2.0`）
- `--style`: 输出风格 — `brief`（要点列表）、`detailed`（含上下文）、`full`（含开发者评论）。默认值：`detailed`。

如果未提供版本，请在继续前询问用户。

---

## Phase 2: 收集变更数据

- 如果存在，读取 `production/releases/[version]/changelog.md` 中的内部 changelog
- 同时检查 `docs/CHANGELOG.md` 中是否有相关版本条目
- 作为备选方案，运行 `git log` 获取上一个发布标签与当前标签/HEAD 之间的提交
- 读取 `production/sprints/` 中的 sprint 回顾以获取上下文
- 读取 `design/balance/` 中的任何平衡性变更文档
- 如果可用，读取 QA 的 bug 修复记录

**如果没有可用的 changelog 数据**（既不存在 `production/releases/[version]/changelog.md`，
也不存在 `docs/CHANGELOG.md` 中该版本的条目，且 git log 为空或不可用）：

> "未找到 [version] 的 changelog 数据。请先运行 `/changelog [version]` 生成内部 changelog，
> 然后重新运行 `/patch-notes [version]`。"

判定：**BLOCKED** — 停止，不生成说明。

---

## Phase 2b: 检测语气指南和模板

**语气指南检测** — 在起草说明之前，检查写作风格指引：

1. 检查 `.claude/docs/technical-preferences.md` 中是否有 "tone"、"voice" 或 "style"
   字段或部分。
2. 如果存在，检查 `docs/PATCH-NOTES-STYLE.md`。
3. 如果存在，检查 `design/community/tone-guide.md`。
4. 如果任何来源包含 tone/voice/style 指令，提取它们并将其应用于生成说明的语言和框架。
5. 如果在任何地方都找不到语气指引，则默认使用：
   玩家友好的、非技术性的语言；热情但不过度夸张；
   关注玩家的体验，而不是开发者的改动。

**模板检测** — 检查是否存在 patch notes 模板：

1. Glob 搜索 `docs/patch-notes-template.md` 和 `.claude/docs/templates/patch-notes-template.md`。
2. 如果在任一位置找到，读取它并将其用作 Phase 4 的输出结构，
   而不是内置的风格模板（Brief / Detailed / Full）。用分类后的数据填充模板的各部分。
3. 如果未找到，使用 Phase 4 中定义的内置风格模板。

---

## Phase 3: 分类与翻译

将所有变更分类为面向玩家的类别：

- **新内容**: 新功能、地图、角色、物品、模式
- **玩法变更**: 平衡性调整、机制变更、进度变化
- **体验优化**: UI 改进、便利功能、无障碍性
- **Bug 修复**: 按系统分组（战斗、UI、网络等）
- **性能**: 玩家可能注意到的优化改进
- **已知问题**: 对未解决问题的透明说明

将开发者语言翻译为玩家语言：

- "Refactored damage calculation pipeline" → "Improved hit detection accuracy"
- "Fixed null reference in inventory manager" → "Fixed a crash when opening inventory"
- "Reduced GC allocations in combat loop" → "Improved combat performance"
- 移除不影响玩家的纯内部变更
- 保留平衡性变更的具体数值（伤害：50 → 45）

---

## Phase 4: 生成 Patch Notes

### Brief 风格
```markdown
# Patch [Version] — [Title]

**新内容**
- [功能 1]
- [功能 2]

**变更**
- [平衡性/机制变更，含变更前 → 变更后数值]

**修复**
- [Bug 修复 1]
- [Bug 修复 2]

**已知问题**
- [问题 1]
```

### Detailed 风格
```markdown
# Patch [Version] — [Title]
*[日期]*

## 亮点
[1-2 句话总结最令人兴奋的变更]

## 新内容
### [功能名称]
[2-3 句话描述该功能以及为什么玩家应该感到兴奋]

## 玩法变更
### 平衡性
| 变更 | 变更前 | 变更后 | 原因 |
| ---- | ---- | ---- | ---- |
| [物品/技能] | [旧值] | [新值] | [简要理由] |

### 机制
- **[变更]**: [解释发生了什么变更以及为什么]

## 体验优化
- [含上下文的改进]

## Bug 修复
### 战斗
- 修复了 [玩家所经历问题的描述]

### UI
- 修复了 [描述]

### 网络
- 修复了 [描述]

## 性能
- [玩家会注意到的改进]

## 已知问题
- [问题及变通方案（如有）]
```

### Full 风格
包含 Detailed 的所有内容，另加：
```markdown
## 开发者评论
### [主题]
> [对重大变更的开发者洞察 — 为什么做出这个决定，考虑了什么，
> 团队学到了什么。以第一人称团队声音撰写。]
```

---

## Phase 5: 审查输出

检查生成的说明是否存在以下问题：

- 无内部术语（用玩家友好的语言替换技术术语）
- 无对内部系统、工单或 sprint 编号的引用
- 平衡性变更包含变更前/变更后数值
- Bug 修复描述的是玩家体验，而非技术原因
- 语气与游戏的声音一致（根据游戏风格调整正式程度）

---

## Phase 6: 保存 Patch Notes

向用户展示完成的 patch notes，同时提供：按类别统计的变更数量，以及被排除的任何内部变更（供审查）。

询问："我可以将这些 patch notes 写入 `docs/patch-notes/[version].md` 吗？"

如果同意，将文件写入 `docs/patch-notes/[version].md`，必要时创建目录。
同时写入 `production/releases/[version]/patch-notes.md` 作为内部存档副本。

---

## Phase 7: 后续步骤

判定：**COMPLETE** — patch notes 已生成并保存。

- 运行 `/release-checklist` 以验证发布前是否满足所有其他发布门槛。
- 在公开发布前，将 patch notes 草稿分享给 community-manager 进行语气审查。
