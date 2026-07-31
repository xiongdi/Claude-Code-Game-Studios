---
name: sprint-status
description: "快速 sprint 状态检查。读取当前 sprint 计划，扫描 story 文件状态，并生成包含 burndown 评估和新兴风险的简洁进度快照。在 sprint 期间随时运行以快速了解态势。当用户询问"sprint 进展如何"、"sprint 更新"、"显示 sprint 进度"时使用。"
argument-hint: "[sprint-number or blank for current]"
user-invocable: true
allowed-tools: Read, Glob, Grep
model: haiku
---

# Sprint Status

这是一个快速态势感知检查，而非 sprint 审查。它读取当前 sprint 计划和 story 文件，扫描状态标记，并在 30 行以内生成简洁的快照。对于详细的 sprint 管理，使用 `/sprint-plan update` 或 `/milestone-review`。

**此 skill 是只读的。** 它绝不提议更改，绝不请求写入文件，最多只做一项具体建议。

---

## 1. 查找 Sprint

**参数：** `$ARGUMENTS[0]`（空白 = 使用当前 sprint）

- 如果提供了参数（如 `/sprint-status 3`），在 `production/sprints/` 中搜索匹配 `sprint-03.md`、`sprint-3.md` 或类似的文件。报告找到了哪个文件。
- 如果没有参数，找到 `production/sprints/` 中最近修改的文件并将其视为当前 sprint。
- 如果 `production/sprints/` 不存在或为空，报告："No sprint files found. Start a sprint with `/sprint-plan new`." 然后停止。

完整读取 sprint 文件。提取：
- Sprint 编号和目标
- 开始日期和结束日期
- 所有 story 或任务条目及其优先级（Must Have / Should Have / Nice to Have）、负责人和估算

---

## 2. 计算剩余天数

使用今天的日期和 sprint 文件中的 sprint 结束日期，计算：
- sprint 总天数（结束减开始）
- 已过天数
- 剩余天数
- 已消耗时间百分比

如果 sprint 计划不包含明确日期，注明 "Sprint dates not
found — burndown assessment skipped."

---

## 3. 扫描 Story 状态

**首先：检查 `production/sprint-status.yaml`。**

如果存在，直接读取它 — 它是权威的真实来源。从 `status` 字段提取每个 story 的状态。无需 markdown 扫描。
使用其 `sprint`、`goal`、`start`、`end` 字段，而非重新解析 sprint 计划。

**如果 `sprint-status.yaml` 不存在**（旧版 sprint 或首次设置），回退到 markdown 扫描：

1. 如果条目引用了 story 文件路径，检查文件是否存在。
   读取文件并扫描状态标记：DONE、COMPLETE、IN PROGRESS、
   BLOCKED、NOT STARTED（不区分大小写）。
2. 如果条目没有文件路径（sprint 计划中的内联任务），扫描
   sprint 计划本身在该条目旁边的状态标记。
3. 如果未找到状态标记，分类为 NOT STARTED。
4. 如果引用了文件但文件不存在，分类为 MISSING 并注明。

使用回退方案时，在输出底部添加说明：
"⚠ No `sprint-status.yaml` found — status inferred from markdown. Run `/sprint-plan update` to generate one."

可选（仅快速检查 — 不要深度扫描）：grep `src/` 查找与 story 的 system slug 匹配的目录或文件名，以检查实现证据。这只是一个提示，不是确定性的状态。

### 陈旧 Story 检测

收集所有 stories 的状态后，检查每个 IN PROGRESS story 的陈旧性：

- 对于有引用文件的每个 story，读取文件并在 frontmatter 或头部查找 `Last Updated:` 字段（如 `Last Updated: 2026-04-01`
  或 `updated: 2026-04-01`）。接受任何合理的日期字段名：`Last Updated`、
  `Updated`、`last-updated`、`updated_at`。
- 使用今天日期计算自该日期以来的天数。
- 如果日期超过 4 天前，将 story 标记为 **STALE**。（4 天阈值考虑了周末 — 上周五最后处理的 story 要到下周三才会显得陈旧。）
- 如果 story 文件中未找到日期字段，注明 "no timestamp — cannot check staleness."
- 如果 story 没有引用文件（内联任务），注明 "inline task — cannot check staleness."

STALE stories 包含在输出表中并收集到 "Attention Needed"
章节（参见 Phase 5 输出格式）。

**陈旧 story 升级**：如果任何 IN PROGRESS story 被标记为 STALE（4+ 天无进展），burndown 裁决
至少升级为 **At Risk** — 即使完成百分比在正常的
On Track 窗口内。记录此升级原因："At Risk — [N] story(ies) with no progress in
[N] days."

---

## 4. Burndown 评估

计算：
- 已完成任务（DONE 或 COMPLETE）
- 进行中任务（IN PROGRESS）
- 已阻塞任务（BLOCKED）
- 未开始任务（NOT STARTED 或 MISSING）
- 完成百分比：(complete / total) * 100

通过比较完成百分比与已消耗时间百分比评估 burndown：

- **On Track**：完成百分比在已消耗时间百分比的 10 个百分点内或超前
- **At Risk**：完成百分比落后已消耗时间百分比 10-25 个百分点
- **Behind**：完成百分比落后已消耗时间百分比超过 25 个百分点

如果日期不可用，跳过 burndown 评估并报告 "On Track /
At Risk / Behind: unknown — sprint dates not found."

---

## 5. 输出

保持输出简洁。story 状态表是必需的 — 不要截断。目标总共 50 行以内；如果未发现值得注意的内容，省略 Emerging Risks 章节。使用以下格式：

```markdown
## Sprint [N] Status — [Today's Date]
**Sprint Goal**: [from sprint plan]
**Days Remaining**: [N] of [total] ([% time consumed])

### Progress: [complete/total] tasks ([%])

| Story / Task         | Priority   | Status      | Owner   | Blocker        |
|----------------------|------------|-------------|---------|----------------|
| [title]              | Must Have  | DONE        | [owner] |                |
| [title]              | Must Have  | IN PROGRESS | [owner] |                |
| [title]              | Must Have  | BLOCKED     | [owner] | [brief reason] |
| [title]              | Should Have| NOT STARTED | [owner] |                |

### Attention Needed
| Story / Task         | Status      | Last Updated   | Days Stale | Note           |
|----------------------|-------------|----------------|------------|----------------|
| [title]              | IN PROGRESS | [date or N/A]  | [N days]   | [STALE / no timestamp — cannot check staleness / inline task — cannot check staleness] |

*（如果没有 IN PROGRESS stories 陈旧或有时间戳问题，完全省略此章节。）*

### Burndown: [On Track / At Risk / Behind]
[1-2 sentences. If behind: which Must Haves are at risk. If on track: confirm
and note any Should Haves the team could pull.]

### Must-Haves at Risk
[List any Must Have stories that are BLOCKED or NOT STARTED with less than
40% of sprint time remaining. If none, write "None."]

### Emerging Risks
[Any risks visible from the story scan: missing files, cascading blockers,
stories with no owner. If none, write "None identified."]

### Recommendation
[One concrete action, or "Sprint is on track — no action needed."]
```

---

## 6. 快速升级规则

在输出前应用这些规则，如果触发则将标记放在输出的顶部
（在状态表上方）：

**Critical flag** — 如果 Must Have stories 为 BLOCKED 或 NOT STARTED 且
剩余 sprint 时间少于 40%：

```
SPRINT AT RISK: [N] Must Have stories are not complete with [X]% of sprint
time remaining. Recommend replanning with `/sprint-plan update`.
```

**Completion flag** — 如果所有 Must Have stories 都为 DONE：

```
All Must Haves complete. Team can pull from Should Have backlog.
```

**Missing stories flag** — 如果任何引用的 story 文件不存在：

```
NOTE: [N] story files referenced in the sprint plan are missing.
Run `/story-readiness sprint` to validate story file coverage.
```

---

## 协作协议

此 skill 是只读的。它报告磁盘上文件观察到的事实。

- 它不更新 sprint 计划
- 它不更改 story 状态
- 它不提议范围削减（那是 `/sprint-plan update`）
- 它每次运行最多做一项建议

要了解特定 story 的更多细节，用户可以直接读取 story 文件
或运行 `/story-readiness [path]`。

对于 sprint 重新计划，使用 `/sprint-plan update`。
对于 sprint 结束复盘，使用 `/retrospective`。
