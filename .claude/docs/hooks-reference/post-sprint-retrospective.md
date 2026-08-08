# Hook: post-sprint-retrospective

## Trigger（触发）

在每个 sprint 结束时手动触发（通常由 producer agent 或人类开发者调用）。

## Purpose（目的）

通过分析 sprint 数据自动生成回顾起点：计划 vs 完成的内容、速度变化、bug 趋势和常见阻塞项。这不是一个 git hook，而是一个通过 `producer` agent 调用的工作流 hook。

## Implementation（实现）

这是一个工作流 hook，不是 git hook。通过运行以下命令调用：

```
@producer Generate sprint retrospective for Sprint [N]
```

producer agent 应：

1. **读取 sprint 计划**，从 `production/sprints/sprint-[N].md`
2. **计算指标**：
   - 计划任务 vs 完成任务
   - 计划 story points vs 完成 story points（如使用）
   - 从上个 sprint 结转的项目
   - sprint 中期新增的任务
   - 平均任务完成时间
3. **分析模式**：
   - 最常见的阻塞项
   - 哪个 agent/领域未完成的工作最多
   - 哪些估算最不准确
4. **生成回顾**：

```markdown
# Sprint [N] Retrospective

## Metrics
| Metric | Value |
|--------|-------|
| Tasks Planned | [N] |
| Tasks Completed | [N] |
| Completion Rate | [X%] |
| Carryover from Previous | [N] |
| New Tasks Added | [N] |
| Bugs Found | [N] |
| Bugs Fixed | [N] |

## Velocity Trend
[Sprint N-2]: [X] | [Sprint N-1]: [Y] | [Sprint N]: [Z]
Trend: [Improving / Stable / Declining]

## What Went Well
- [Automatically detected: tasks completed ahead of estimate]
- [Facilitator adds team observations]

## What Went Poorly
- [Automatically detected: tasks that were carried over or cut]
- [Automatically detected: areas with significant estimate overruns]
- [Facilitator adds team observations]

## Blockers
| Blocker | Frequency | Resolution Time | Prevention |
|---------|-----------|----------------|-----------|

## Action Items for Next Sprint
| # | Action | Owner | Priority |
|---|--------|-------|----------|

## Estimation Accuracy
| Area | Avg Planned | Avg Actual | Accuracy |
|------|------------|-----------|----------|
```

5. **保存**到 `production/sprints/sprint-[N]-retro.md`
