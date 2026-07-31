---
name: hotfix
description: "Emergency fix workflow that bypasses normal sprint processes with a full audit trail. Creates hotfix branch, tracks approvals, and ensures the fix is backported correctly."
argument-hint: "[bug-id or description]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion
model: sonnet
---

> **仅显式调用**：此技能仅在用户使用 `/hotfix` 显式请求时运行。不要基于上下文匹配自动调用。

## 第 1 阶段：评估严重性

读取 bug 描述或 ID。使用以下标准评估严重性：

- **S1（关键）**：游戏无法玩、数据丢失、安全漏洞
- **S2（重大）**：重要功能损坏、存在变通方法
- **S3 或更低**：小问题——适用正常 bug 修复工作流

使用 `AskUserQuestion` 确认：
- 提示："我将其评估为 **[评估的严重性]** —— [简要理由]。确认严重性以继续："
- 选项：
  - `[A] S1（关键）——游戏无法玩、数据丢失或安全问题`
  - `[B] S2（重大）——重要功能损坏、存在变通方法`
  - `[C] S3 或更低——重定向到正常 bug 修复工作流`

如果 [C]：停止。裁决：**REDIRECTED**——对 S3 及以下使用正常 bug 修复工作流。

---

## 第 2 阶段：创建 Hotfix 记录

起草 hotfix 记录：

```markdown
## Hotfix：[简短描述]
日期：[日期]
严重性：[S1/S2]
报告者：[谁发现的]
状态：IN PROGRESS

### 问题
[清楚描述什么坏了以及对玩家的影响]

### 根本原因
[在调查期间填写]

### 修复
[在实现期间填写]

### 测试
[测试了什么以及如何测试]

### 批准
- [ ] 修复由 lead-programmer 审查
- [ ] 回归测试通过（qa-tester）
- [ ] 发布已批准（producer）

### 回滚计划
[如果修复导致新问题如何恢复]
```

询问："我可以将此写入 `production/hotfixes/hotfix-[date]-[short-name].md` 吗？"

如果同意，写入文件，必要时创建目录。

---

## 第 3 阶段：创建 Hotfix 分支

检查这是否是 git 仓库：

`Bash: git rev-parse --is-inside-work-tree 2>/dev/null`

如果此命令失败或返回空：注明"不是 git仓库——手动创建分支。"并跳过分支创建。

如果检查通过，在创建分支之前使用 `AskUserQuestion`：
- 提示："准备好从 [base-ref] 创建 hotfix 分支 'hotfix/[short-name]' 了吗？"
- 选项：
  - `[A] 是——创建分支`
  - `[B] 使用不同的基础 ref——我会指定它`
  - `[C] 跳过——我自己创建分支`

仅当用户选择 [A] 时运行 `git checkout -b hotfix/[short-name] [base-ref]`。如果 [B]：询问用户基础 ref，然后使用该 ref 运行命令。如果 [C]：跳过分支创建并进入第 4 阶段。

---

## 第 4 阶段：调查和实现

专注于解决问题的最小更改。不要在 hotfix 旁边重构、清理或添加功能。

通过运行受影响系统的针对性测试来验证修复。检查相邻系统的回归。

用根本原因、修复细节和测试结果更新 hotfix 记录。

---

## 第 5 阶段：收集批准

使用 Task 工具并行请求签字确认：

- `subagent_type: lead-programmer` —— 审查修复的正确性和副作用
- `subagent_type: qa-tester` —— 在受影响系统上运行针对性回归测试
- `subagent_type: producer` —— 批准部署时间和沟通计划

所有三个必须在继续之前返回 APPROVE。如果任何返回 CONCERNS 或 REJECT，不要部署——先展示问题并解决。

---

## 第 5b 阶段：QA 重新进入门

在批准后，确定部署 hotfix 之前所需的 QA 范围。通过 Task 生成 `qa-lead`，提供：
- hotfix 描述和受影响系统
- 第 5 阶段的回归测试结果
- 触及更改文件的所有系统的列表（使用 Grep 查找调用者）

询问 qa-lead：**完整的冒烟检查是否足够，还是此修复需要针对性的 team-qa 通道？**

应用裁决：
- **冒烟检查足够** —— 对 hotfix 构建运行 `/smoke-check`。如果 PASS，进入第 6 阶段。
- **需要针对性 QA 通道** —— 仅在更改的系统范围内运行 `/team-qa [affected-system]`。如果 QA 返回 APPROVED 或 APPROVED WITH CONDITIONS，进入第 6 阶段。
- **需要完整 QA** —— 触及核心系统的 S1 修复可能需要完整的 `/team-qa sprint`。这会延迟部署但防止不良补丁。

不要跳过此门。破坏其他东西的 hotfix 比原来的 bug 更糟。

---

## 第 6 阶段：更新 Bug 状态并部署

如果存在，更新原始 bug 文件：

```markdown
## 修复记录
**修复于**：hotfix/[branch-name] —— [commit hash 或描述]
**修复日期**：[date]
**状态**：Fixed — Pending Verification
```

在 bug 文件头部设置 `**Status**: Fixed — Pending Verification`。

输出部署摘要：

```
## Hotfix 准备部署：[short-name]

**严重性**：[S1/S2]
**根本原因**：[一行]
**修复**：[一行]
**QA 门**：[冒烟检查 PASS / Team-QA APPROVED]
**批准**：lead-programmer ✓ / qa-tester ✓ / producer ✓
**回滚计划**：[来自第 2 阶段记录]

合并到：release 分支 AND 开发分支
下一步：部署后运行 /bug-report verify [BUG-ID] 确认解决
```

### 规则
- Hotfix 必须是解决问题的最小更改——无清理、无重构
- 每个 hotfix 在部署前必须有记录的回滚计划
- Hotfix 分支合并到 release 分支 AND 开发分支
- 所有 hotfix 需要在 48 小时内进行事后审查
- 如果修复复杂到需要超过 4 小时，升级到 `technical-director`

---

## 第 7 阶段：部署后验证

部署后，运行 `/bug-report verify [BUG-ID]` 确认修复在已部署构建中解决了问题。

如果 VERIFIED FIXED：运行 `/bug-report close [BUG-ID]` 正式关闭它。
如果 STILL PRESENT：hotfix 失败——立即重新打开、评估回滚并升级。

使用 `/retrospective hotfix` 在 48 小时内安排事后审查。

使用 `AskUserQuestion`：
- 提示："Hotfix 完成。下一步？"
- 选项：
  - `[A] 运行 /smoke-check 验证修复`
  - `[B] 运行 /patch-notes 记录此 hotfix`
  - `[C] 停在这里`
