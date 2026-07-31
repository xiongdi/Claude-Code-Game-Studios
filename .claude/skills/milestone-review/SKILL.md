---
name: milestone-review
description: "Generates a comprehensive milestone progress review including feature completeness, quality metrics, risk assessment, and go/no-go recommendation. Use at milestone checkpoints or when evaluating readiness for a milestone deadline."
argument-hint: "[milestone-name|current] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Task, AskUserQuestion
model: sonnet
---

## 第 0 阶段：解析参数

提取里程碑名称（`current` 或特定名称）并解析审查模式（一次，存储供本次运行的所有门生成使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

参见 `.claude/docs/director-gates.md` 了解完整的检查模式。

---

## 第 1 阶段：加载里程碑数据

从 `production/milestones/` 读取里程碑定义。如果参数是 `current`，使用最近修改的里程碑文件。

从 `production/sprints/` 读取此里程碑内所有 sprint 的 sprint 报告。

---

## 第 2 阶段：扫描代码库健康

- 扫描指示未完成工作的 `TODO`、`FIXME`、`HACK` 标记
- 检查 `production/risk-register/` 中的风险登记册

---

## 第 3 阶段：生成里程碑审查

```markdown
# 里程碑审查：[里程碑名称]

## 概述
- **目标日期**：[日期]
- **当前日期**：[今天]
- **剩余天数**：[N]
- **已完成的 Sprint**：[X/Y]

## 功能完整性

### 完全完成
| 功能 | 验收标准 | 测试状态 |
|---------|-------------------|-------------|

### 部分完成
| 功能 | 完成百分比 | 剩余工作 | 对里程碑的风险 |
|---------|--------|-----------|------------------|

### 未开始
| 功能 | 优先级 | 可以削减吗？ | 削减的影响 |
|---------|----------|----------|------------------|

## 质量指标
- **未解决的 S1 Bug**：[N] -- [列表]
- **未解决的 S2 Bug**：[N]
- **未解决的 S3 Bug**：[N]
- **测试覆盖率**：[X%]
- **性能**：[在预算内？详情]

## 代码健康
- **TODO 计数**：[代码库中 N 个]
- **FIXME 计数**：[N]
- **HACK 计数**：[N]
- **技术债务项目**：[列出关键的]

## 风险评估
| 风险 | 状态 | 如果实现的影响 | 缓解状态 |
|------|--------|-------------------|------------------|

## 速度分析
- **计划 vs 完成**（跨所有 sprint）：[X/Y 任务 = Z%]
- **趋势**：[改善 / 稳定 / 下降]
- **剩余工作的调整后估算**：[当前速度下需要的天数]

## 范围建议
### 保护（必须与里程碑一起发布）
- [功能和原因]

### 有风险（可能需要削减或简化）
- [功能和风险]

### 削减候选（可以推迟而不影响里程碑）
- [功能和削减的影响]

## Go/No-Go 评估

**建议**：[GO / CONDITIONAL GO / NO-GO]

**条件**（如果是条件性的）：
- [必须满足的条件 1]
- [必须满足的条件 2]

**理由**：[建议的解释]

## 操作项
| # | 操作 | 负责人 | 截止日期 |
|---|--------|-------|----------|
```

---

## 第 3b 阶段：Producer 风险评估

**审查模式检查** — 在生成 PR-MILESTONE 之前应用：
- `solo` → 跳过。注意："PR-MILESTONE 已跳过 — Solo 模式。" 在没有 producer 裁决的情况下展示 Go/No-Go 部分。
- `lean` → 跳过（不是 PHASE-GATE）。注意："PR-MILESTONE 已跳过 — Lean 模式。" 在没有 producer 裁决的情况下展示 Go/No-Go 部分。
- `full` → 正常生成。

在生成 Go/No-Go 建议之前，通过 Task 使用门 **PR-MILESTONE**（`.claude/docs/director-gates.md`）生成 `producer`。

传递：里程碑名称和目标日期、当前完成百分比、受阻 story 计数、来自 sprint 报告的速度数据（如果可用）、削减候选列表。

在 Go/No-Go 部分内联展示 producer 的评估。producer 的裁决（ON TRACK / AT RISK / OFF TRACK）通知整体建议。

如果 OFF TRACK，在生成建议之前使用 `AskUserQuestion`：
- 提示："Producer 裁决：OFF TRACK。里程碑处于危险中。此审查将推荐 NO-GO。你想如何继续？"
- 选项：
  - `[A] 接受 NO-GO — 生成含该建议的完整审查`
  - `[B] 覆盖为 CONDITIONAL GO — 我将自己记录接受的风险`
  - `[C] 停止 — 我想在生成审查之前解决阻塞项`

如果 AT RISK，使用 `AskUserQuestion`：
- 提示："Producer 裁决：AT RISK。里程碑可能滑脱。Go/No-Go 部分应如何构建？"
- 选项：
  - `[A] CONDITIONAL GO — 在审查中包含 producer 的条件`
  - `[B] NO-GO — 条件无法及时满足`
  - `[C] GO — 接受风险并想继续`

除非用户明确选择上述 [B]，否则不要针对 OFF TRACK 裁决发出 GO。

---

## 第 4 阶段：保存审查

向用户展示审查。

询问："我可以将此写入 `production/milestones/[milestone-name]-review.md` 吗？"

如果同意，写入文件，必要时创建目录。裁决：**COMPLETE**——里程碑审查已保存。

如果不同意，停在这里。裁决：**BLOCKED**——用户拒绝写入。

---

## 第 5 阶段：后续步骤

- 如果此里程碑标志着开发阶段边界，运行 `/gate-check` 获得正式的 phase gate 裁决。
- 基于上述范围建议运行 `/sprint-plan` 调整下一个 sprint。
