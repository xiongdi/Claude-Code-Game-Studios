---
name: bug-report
description: "Creates a structured bug report from a description, or analyzes code to identify potential bugs. Ensures every bug report has full reproduction steps, severity assessment, and context."
argument-hint: "[description] | analyze [path-to-file]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write
model: sonnet
---

## 阶段 1：解析参数

从参数确定模式：

- 无关键字 → **描述模式**：从提供的描述生成结构化 bug 报告
- `analyze [path]` → **分析模式**：读取目标文件并识别潜在 bug
- `verify [BUG-ID]` → **验证模式**：确认报告的修复实际解决了 bug
- `close [BUG-ID]` → **关闭模式**：将已验证的 bug 标记为关闭并记录解决情况

如果未提供参数，在继续之前要求用户提供 bug 描述。

---

## 阶段 2A：描述模式

1. **解析描述**以获取关键信息：什么坏了、何时、如何重现以及预期行为是什么。

2. **搜索代码库**以使用 Grep/Glob 获取相关文件以添加上下文（受影响的系统、可能的文件）。

3. **起草 bug 报告**：

```markdown
# Bug Report

## 摘要
**标题**: [简洁、描述性的标题]
**ID**: BUG-[NNNN]
**严重性**: [S1-Critical / S2-Major / S3-Minor / S4-Trivial]
**优先级**: [P1-Immediate / P2-Next Sprint / P3-Backlog / P4-Wishlist]
**状态**: Open
**报告日期**: [日期]
**报告人**: [姓名]

## 分类
- **类别**: [Gameplay / UI / Audio / Visual / Performance / Crash / Network]
- **系统**: [受影响的哪个游戏系统]
- **频率**: [Always / Often (>50%) / Sometimes (10-50%) / Rare (<10%)]
- **回归**: [Yes/No/Unknown —— 这个以前工作过吗？]

## 环境
- **构建**: [版本或提交哈希]
- **平台**: [操作系统，如果相关则包括硬件]
- **场景/关卡**: [游戏中的位置]
- **游戏状态**: [相关状态 —— 库存、任务进度等]

## 重现步骤
**前置条件**: [开始前的必需状态]

1. [确切步骤 1]
2. [确切步骤 2]
3. [确切步骤 3]

**预期结果**: [应该发生什么]
**实际结果**: [实际发生什么]

## 技术上下文
- **可能受影响的文件**: [基于代码库搜索的文件列表]
- **相关系统**: [可能涉及的其他系统]
- **可能的根本原因**: [如果从描述中可识别]

## 证据
- **日志**: [如果可用，相关日志输出]
- **视觉**: [视觉证据的描述]

## 相关问题
- [链接到相关 bug 或设计文档]

## 备注
[任何其他上下文或观察]
```

---

## 阶段 2B：分析模式

1. **读取目标文件**（参数中指定的）。

2. **识别潜在 bug**：空引用、差一错误、竞争条件、未处理的边缘情况、资源泄漏、不正确的状态转换。

3. **对于每个潜在 bug**，使用上述模板生成 bug 报告，填入可能的触发场景和建议的修复。

---

## 阶段 2C：验证模式

读取 `production/qa/bugs/[BUG-ID].md`。提取重现步骤和预期结果。

1. **重新运行重现步骤** —— 使用 Grep/Glob 检查根本原因代码路径是否如描述般仍然存在。如果修复删除或更改了它，记录更改。
2. **运行相关测试** —— 如果 bug 的系统在 `tests/` 中有测试文件，通过 Bash 运行它并报告通过/失败。
3. **检查回归** —— grep 代码库中导致 bug 的模式的任何新出现。

生成验证裁决：

- **VERIFIED FIXED** —— 重现步骤不再产生 bug；相关测试通过
- **STILL PRESENT** —— bug 如描述般重现；修复未解决问题
- **CANNOT VERIFY** —— 自动化检查不确定；需要手动试玩

询问："我可以更新 `production/qa/bugs/[BUG-ID].md` 以设置 Status: Verified Fixed / Still Present / Cannot Verify 吗？"

如果 STILL PRESENT：重新打开 bug，将状态设置回 Open，并建议重新运行 `/hotfix [BUG-ID]`。

---

## 阶段 2D：关闭模式

读取 `production/qa/bugs/[BUG-ID].md`。在关闭前确认状态为 `Verified Fixed`。如果状态是其他任何内容，停止："Bug [ID] 必须 Verified Fixed 才能关闭。先运行 `/bug-report verify [BUG-ID]`。"

将关闭记录追加到 bug 文件：

```markdown
## 关闭记录
**关闭日期**: [日期]
**解决方式**: Fixed —— [更改内容的一句话描述]
**修复提交/PR**: [如果已知]
**验证人**: qa-tester
**关闭人**: [用户]
**回归测试**: [测试文件路径，或"Manual verification"]
**状态**: Closed
```

更新顶层的 `**Status**: Open` 字段为 `**Status**: Closed`。

询问："我可以更新 `production/qa/bugs/[BUG-ID].md` 以标记为 Closed 吗？"

关闭后，检查 `production/qa/bug-triage-*.md` —— 如果 bug 出现在开放的 triage 报告中，注意："Bug [ID] 在 triage 报告中被引用。运行 `/bug-triage` 刷新开放 bug 计数。"

---

## 阶段 3：保存报告

向用户展示完成的 bug 报告。

询问："我可以将此写入 `production/qa/bugs/BUG-[NNNN].md` 吗？"

如果是，写入文件，如果需要创建目录。裁决：**COMPLETE** —— bug 报告已提交。

如果否，在这里停止。裁决：**BLOCKED** —— 用户拒绝写入。

---

## 阶段 4：下一步

保存后，根据模式建议：

**提交后（描述/分析模式）：**
- 运行 `/bug-triage` 以与现有开放 bug 一起优先处理
- 如果 S1 或 S2：运行 `/hotfix [BUG-ID]` 进行紧急修复工作流

**修复 bug 后（开发者确认修复已就位）：**
- 运行 `/bug-report verify [BUG-ID]` —— 在关闭前确认修复实际有效
- 永远不要在没有验证的情况下标记 bug 关闭 —— 未验证的修复仍然是 Open

**验证返回 VERIFIED FIXED 后：**
- 运行 `/bug-report close [BUG-ID]` —— 写入关闭记录并更新状态
- 运行 `/bug-triage` 刷新开放 bug 计数并将其从活动列表中移除
