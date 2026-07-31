# Agent 协调规则

1. **垂直委托**：领导 Agent 委托给部门主管，部门主管委托给专家。
   复杂决策绝不跳过层级。
2. **水平协商**：同层级的 Agent 可以互相协商，但不能在其领域外做出约束性决策。
3. **冲突解决**：当两个 Agent 不同意时，升级到共同上级。
   如果没有共同上级，设计冲突升级到 `creative-director`，技术冲突升级到 `technical-director`。
4. **变更传播**：当设计变更影响多个领域时，`producer` Agent 协调传播。
5. **禁止单方面跨域变更**：Agent 未经明确委托不得修改其指定目录外的文件。

## 模型层级分配

技能和 Agent 根据任务复杂度分配到模型层级：

| 层级 | 模型 | 使用场景 |
|------|-------|-------------|
| **Haiku** | `claude-haiku-4-5-20251001` | 只读状态检查、格式化、简单查询 — 不需要创意判断 |
| **Sonnet** | `claude-sonnet-4-6` | 实现、设计创作、单个系统分析 — 大多数工作的默认值 |
| **Opus** | `claude-opus-4-6` | 多文档综合、高风险阶段门裁决、跨系统整体审查 |

`model: haiku` 的技能：`/help`、`/sprint-status`、`/story-readiness`、`/scope-check`、
`/project-stage-detect`、`/changelog`、`/patch-notes`、`/onboard`

`model: opus` 的技能：`/review-all-gdds`、`/architecture-review`、`/gate-check`

所有其他技能默认为 Sonnet。创建新技能时，如果技能只读取和格式化则分配 Haiku；
如果必须综合 5+ 个文档且输出高风险则分配 Opus；否则不设置（Sonnet）。

## Subagents 与 Agent Teams

本项目使用两种不同的多 Agent 模式：

### Subagents（当前，始终活跃）
通过单个 Claude Code 会话中的 `Task` 派生。被所有 `team-*` 技能和编排技能使用。
Subagent 共享会话的权限上下文，在会话内顺序或并行运行，并返回结果给父级。

**何时并行派生**：如果两个 subagent 的输入是独立的（都不需要对方输出才能开始），
同时派生两个 Task 调用，而不是等待。例如：`/review-all-gdds` 第 1 阶段（一致性）
和第 2 阶段（设计理论）是独立的 — 同时派生两者。

### Agent Teams（实验性 — 选择加入）
多个独立的 Claude Code *会话* 同时运行，通过共享任务列表协调。
每个会话有自己的上下文窗口和 token 预算。
需要 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 环境变量。

**使用 Agent Teams 时**：
- 工作跨越多个不会触碰相同文件的子系统
- 每个工作流需要 >30 分钟并从真正并行中受益
- 一个高级 Agent（technical-director、producer）需要协调 3+ 个同时在不同 epic 上工作的专家会话

**不要使用 Agent Teams 时**：
- 一个会话的输出需要作为另一个的输入（使用顺序 subagent）
- 任务适合单个会话的上下文（使用 subagent）
- 成本是问题 — 每个团队成员独立消耗 token

**当前状态**：通过 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 环境变量选择加入。首次采用时在此记录。

## 并行任务协议

当编排技能派生多个独立 Agent 时：

1. 在等待任何结果前发出所有独立 Task 调用
2. 在进入依赖阶段前收集所有结果
3. 如果任何 Agent 被 BLOCKED，立即暴露 — 不要默默跳过
4. 如果一些 Agent 完成而其他被阻塞，总是产生部分报告