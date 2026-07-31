---
name: day-one-patch
description: "Prepare a day-one patch for a game launch. Scopes, prioritises, implements, and QA-gates a focused patch addressing known issues discovered after gold master but before or immediately after public launch. Treats the patch as a mini-sprint with its own QA gate and rollback plan."
argument-hint: "[scope: known-bugs | cert-feedback | all]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion
model: sonnet
---

# Day-One Patch

每个已发布游戏都有 day-one patch。在发布日之前计划它可以防止混乱。此技能将 patch 范围限定为仅安全且必要的内容，通过轻量级 QA 通道进行门控，并确保在任何内容发布前存在回滚计划。它是一个 mini-sprint——不是 hotfix，不是完整 sprint。

**何时运行：**
- gold master 构建锁定后（cert 批准或 launch candidate 标记）
- 存在已知 bug，在 gold master 中修复风险太大
- cert 反馈需要在提交后进行小修复
- 发布门通过后，发布前试玩浮现了必须修复的问题

**Day-one patch 范围规则：**
- 仅 P1/P2 bug，且快速修复是安全的
- 无新功能——这仅限修复
- 无重构——最小可行更改
- 任何需要超过 4 小时开发时间的修复属于 patch 1.1，不属于 day-one

**输出：** `production/releases/day-one-patch-[version].md`

---

## 第 1 阶段：加载发布上下文

读取：
- `production/stage.txt` — 确认项目处于 Release 阶段
- `production/gate-checks/` 中最新的文件 — 读取发布门裁决
- `production/qa/bugs/*.md` — 加载所有状态为 Open 或 Fixed — Pending Verification 的 bug
- `production/sprints/` 最新的 — 了解已发布的内容
- `production/security/security-audit-*.md` 最新的 — 检查任何未解决的安全项目

如果 `production/stage.txt` 不是 `Release` 或 `Polish`：
> "Day-one patch 准备适用于 Release 阶段的项目。当前阶段：[stage]。在接近发布之前，此技能不适用。"

---

## 第 2 阶段：确定 Patch 范围

### 步骤 2a — 分类补丁包含的未解决 bug

对于每个未解决的 bug，评估：

| 标准 | 包含在 day-one 中？ |
|-----------|-------------------|
| S1 或 S2 严重性 | 是 — 如果安全修复则必须包含 |
| P1 优先级 | 是 |
| 修复估算 < 4 小时 | 是 |
| 修复需要架构变更 | 否 — 推迟到 1.1 |
| 修复引入新代码路径 | 否 — 太危险 |
| 修复仅数据/配置（无代码更改） | 是 — 风险极低 |
| Cert 反馈要求 | — 平台批准所需 |
| S3/S4 严重性 | 仅当琐碎的配置修复时；否则推迟 |

### 步骤 2b — 向用户展示补丁范围

使用 `AskUserQuestion`：
- 提示："根据未解决的 bug 和 cert 反馈，这是提议的 day-one patch 范围。这看起来对吗？"
- 展示：包含的 bug 表（ID、严重性、描述、估算工作量）
- 展示：推迟的 bug 表（ID、严重性、推迟原因）
- 选项：`[A] 批准此范围` / `[B] 调整 — 我想添加或删除项目` / `[C] 不需要 day-one patch`

如果 [C]：输出 "不需要 day-one patch。继续 `/launch-checklist`。" 停止。

### 步骤 2c — 检查总范围

汇总估算工作量。如果总计超过 1 天的工作量：
> "⚠️ Patch 范围为 [N 小时] — 这超出了安全的 day-one 窗口。考虑将低优先级项目推迟到 patch 1.1。臃肿的 day-one patch 引入的风险比消除的更多。"

使用 `AskUserQuestion` 确认继续或缩小范围。

---

## 第 3 阶段：回滚计划

在写入任何代码之前，定义回滚程序。这是不可协商的。

通过 Task 生成 `release-manager`。要求他们制定涵盖以下内容的回滚计划：
- 如何在每个目标平台上恢复到 gold master 构建
- 平台特定的回滚约束（某些平台无法回滚 cert 构建）
- 谁负责触发回滚
- 如果发生回滚，需要什么样的玩家沟通

展示回滚计划。询问："我可以将此回滚计划写入 `production/releases/rollback-plan-[version].md` 吗？"

在回滚计划写入之前，不要进入第 4 阶段。

---

## 第 4 阶段：实现修复

对于批准范围内的每个 bug，生成一个专注的实现循环：

1. 通过 Task 生成 `lead-programmer`，提供：
   - bug 报告（确切的复现步骤和根本原因，如果已知）
   - 约束：仅限最小可行修复，无清理
   - 受影响的文件（来自 bug 报告的 Technical Context 部分）

2. lead-programmer 实现并运行针对性测试。

3. 通过 Task 生成 `qa-tester` 验证：修复后 bug 是否重现？

对于仅配置/数据的修复：直接进行更改（不需要程序员代理）。确认值已更改并重新运行任何相关的冒烟测试。

---

## 第 5 阶段：Patch QA 门

这是一个轻量级的 QA 通道——不是完整的 `/team-qa`。Patch 已经从发布门获得了 QA 批准；我们仅重新验证更改的区域。

通过 Task 生成 `qa-lead`，提供：
- 所有更改文件的列表
- 已修复 bug 的列表（含第 4 阶段的验证状态）
- 受影响系统的冒烟检查范围

询问 qa-lead：**针对性冒烟检查是否足够，还是任何修复触及需要更广泛回归的系统？**

运行所需的 QA 范围：
- **针对性冒烟检查** — 运行 `/smoke-check [affected-systems]`
- **更广泛的回归** — 在受影响系统的 `tests/unit/` 和 `tests/integration/` 中运行针对性测试

在继续之前，QA 裁决必须为 PASS 或 PASS WITH WARNINGS。如果 FAIL：将失败的修复排除在 day-one patch 之外并推迟到 1.1。

---

## 第 6 阶段：生成 Patch 记录

```markdown
# Day-One Patch: [游戏名称] v[version]

**准备日期**：[date]
**目标发布**：[launch date 或 "发布当天"]
**基础构建**：[gold master tag 或 commit]
**Patch 构建**：[patch tag 或 commit]

---

## Patch Notes（内部）

### 已修复 Bug
| BUG-ID | 严重性 | 描述 | 修复摘要 |
|--------|----------|-------------|-------------|
| BUG-NNN | S[1-4] | [描述] | [一行修复] |

### 推迟到 1.1
| BUG-ID | 严重性 | 描述 | 推迟原因 |
|--------|----------|-------------|-----------------|
| BUG-NNN | S[1-4] | [描述] | [原因] |

---

## QA Sign-Off

**QA 范围**：[针对性冒烟 / 更广泛回归]
**裁决**：[PASS / PASS WITH WARNINGS]
**QA lead**：qa-lead agent
**日期**：[date]
**警告（如有）**：[列表或 "None"]

---

## 回滚计划

参见：`production/releases/rollback-plan-[version].md`

**触发条件**：如果在发布后 [X] 小时内报告 [N] 个或更多 S1 bug，执行回滚。
**回滚负责人**：[user / producer]

---

## 部署前所需批准
- [ ] lead-programmer：所有修复已审查
- [ ] qa-lead：QA 门 PASS 已确认
- [ ] producer：部署时间已批准
- [ ] release-manager：平台提交已确认

---

## 面向玩家的 Patch Notes

[供 community-manager 在发布前审查的草稿]

[用通俗语言列出面向玩家的更改]
```

询问："我可以将此 patch 记录写入 `production/releases/day-one-patch-[version].md` 吗？"

---

## 第 7 阶段：后续步骤

patch 记录写入后：

1. 运行 `/patch-notes` 生成面向玩家的 patch notes 版本
2. patch 上线后，对每个已修复的 bug 运行 `/bug-report verify [BUG-ID]`
3. 对每个已验证的修复运行 `/bug-report close [BUG-ID]`
4. 使用 `/retrospective launch` 在发布后 48-72 小时安排发布后审查

**如果 patch 后仍有任何 S1 bug 未解决：**
> "⚠️ S1 bug 仍未解决且未打补丁。这些是已接受的风险。将它们记录在回滚计划触发条件中——如果它们大规模发生，回滚可能比后续 patch 更可取。"

使用 `AskUserQuestion`：
- 提示："Day-one patch 完成。下一步？"
- 选项：
  - `[A] 运行 /patch-notes — 生成面向玩家的 patch notes`
  - `[B] 运行 /bug-report 记录部署后发现的问题`
  - `[C] 停在这里`

---

## 协作协议

- **范围纪律就是一切** — 抵制范围蔓延；每个添加都增加风险
- **回滚计划优先，始终** — 没有回滚计划的 patch 是不负责任的
- **推迟不等于遗忘** — 每个推迟的 bug 自动获得 1.1 工单
- **玩家沟通是 patch 的一部分** — `/patch-notes` 是必需的输出，不是可选的
