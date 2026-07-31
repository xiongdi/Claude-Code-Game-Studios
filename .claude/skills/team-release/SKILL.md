---
name: team-release
description: "编排发布团队：协调 release-manager、qa-lead、devops-engineer 和 producer，执行从候选版本到部署的发布。"
argument-hint: "[version number or 'next'] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
model: sonnet
---
**参数检查：** 如果未提供版本号：
1. 读取 `production/session-state/active.md` 和 `production/milestones/` 中最近的文件（如果存在）来推断目标版本。
2. 如果找到版本：报告 "No version argument provided — inferred [version] from milestone data. Proceeding." 然后用 `AskUserQuestion` 确认："Releasing [version]. Is this correct?"
3. 如果无法发现版本：使用 `AskUserQuestion` 询问 "What version number should be released? (e.g., v1.0.0)" 并等待用户输入后再继续。不要默认为硬编码的版本字符串。

当此 skill 被调用时，通过结构化流水线编排发布团队。

**决策点：** 在每个阶段转换时，使用 `AskUserQuestion` 向用户展示 subagent 的提案作为可选项。先在对话中写出 agent 的完整分析，然后用简洁的标签捕获决策。用户必须批准才能进入下一阶段。

## 阶段 0：解析审查模式

1. 如果传入了 `--review [mode]` 参数，使用该模式。
2. 否则读取 `production/review-mode.txt` — 使用其中写入的内容。
3. 否则默认为 `lean`。

模式：
- `full` — 生成所有 director 和 lead 门控，如下所述
- `lean` — 跳过 director 门控，除非它们是 PHASE-GATE 类型（CD-PHASE-GATE, TD-PHASE-GATE, PR-PHASE-GATE, AD-PHASE-GATE）
- `solo` — 完全跳过所有 director 门控生成；在没有任何 agent 门控的情况下运行 skill

存储解析后的模式供所有后续阶段使用。

## 团队组成
- **release-manager** — 发布分支、版本号、changelog、部署
- **qa-lead** — 测试签字、回归套件、发布质量门控
- **devops-engineer** — 构建流水线、产物、部署自动化
- **security-engineer** — 发布前安全审计（如果游戏有在线/多人功能或玩家数据时调用）
- **analytics-engineer** — 验证遥测事件正确触发且仪表板已上线
- **community-manager** — 补丁说明、发布公告、面向玩家的消息
- **producer** — 上线/搁置决策、利益相关者沟通、排期

## 如何委托

使用 Task 工具将每个团队成员生成为 subagent：
- `subagent_type: release-manager` — 发布分支、版本号、changelog、部署
- `subagent_type: qa-lead` — 测试签字、回归套件、发布质量门控
- `subagent_type: devops-engineer` — 构建流水线、产物、部署自动化
- `subagent_type: security-engineer` — 在线/多人/数据功能的安全审计
- `subagent_type: analytics-engineer` — 遥测事件验证和仪表板就绪
- `subagent_type: community-manager` — 补丁说明和发布沟通
- `subagent_type: producer` — 上线/搁置决策、利益相关者沟通
- `subagent_type: network-programmer` — 网络代码稳定性签字（如果游戏有多人模式时调用）

始终在每个 agent 的提示中提供完整上下文（版本号、里程碑状态、已知问题）。在流水线允许的地方并行启动独立的 agent（例如阶段 3 的 agent 可以同时运行）。

## 流水线

### 阶段 1：发布规划
委托给 **producer**：
- 确认所有里程碑验收标准已满足
- 识别本次发布中延期的任何范围项
- 设定目标发布日期并传达给团队
- 输出：带范围确认的发布授权

### 阶段 2：发布候选
委托给 **release-manager**：
- 从约定的提交切出发布分支
- 在所有相关文件中更新版本号
- 使用 `/release-checklist` 生成发布检查清单
- 冻结分支 — 不允许功能变更，只允许修复 bug
- 输出：发布分支名称和检查清单

### 阶段 3：质量门控（并行）
并行委托：
- **qa-lead**：执行完整回归测试套件。测试所有关键路径。验证无 S1/S2 bug。签字确认质量。
- **devops-engineer**：为所有目标平台构建发布产物。验证构建干净且可复现。在 CI 中运行自动化测试。
- **security-engineer** *（如果游戏有在线功能、多人模式或玩家数据）*：进行发布前安全审计。审查认证、反作弊、数据隐私合规。签字确认安全态势。
- **network-programmer** *（如果游戏有多人模式）*：签字确认网络代码稳定性。验证延迟补偿、重连处理、负载下的带宽使用。

### 阶段 4：本地化、性能和分析
委托（如果资源可用，可与阶段 3 并行运行）：
- 验证所有字符串已翻译（如果可用，委托给 **localization-lead**）
- 对照目标运行性能基准测试（如果可用，委托给 **performance-analyst**）
- **analytics-engineer**：验证所有遥测事件在发布构建上正确触发。确认仪表板正在接收数据。检查关键漏斗（新手引导、进度、货币化如适用）已埋点。
- 输出：本地化、性能和分析签字

### 阶段 5：上线/搁置决策
委托给 **producer**：
- 收集以下人员的签字：qa-lead、release-manager、devops-engineer、security-engineer（如果在阶段 3 生成）、network-programmer（如果在阶段 3 生成）和 technical-director
- 评估任何未解决的问题 — 它们是阻塞性的还是可以发布？
- 做出上线/搁置决策
- 输出：带理由的发布决策

**如果 producer 宣布 NO-GO：**
- 立即暴露决策："PRODUCER: NO-GO — [理由，例如 Phase 3 发现 S1 bug]。"
- 使用 `AskUserQuestion` 提供选项：
  - 修复阻塞项并重新运行受影响的阶段
  - 将发布延期到更晚日期
  - 用书面理由覆盖 NO-GO（用户必须提供书面理由）
- **完全跳过阶段 6** — 不要打标签、部署到 staging、部署到生产环境或生成 community-manager。
- 生成部分报告，总结阶段 1-5 以及跳过的（阶段 6）内容和原因。
- 结论：**BLOCKED** — 发布未部署。

在用户选择 "Override NO-GO with documented rationale" 后：
- 询问（纯文本，非组件）："Please describe the justification for overriding the NO-GO verdict. This will be embedded in the release record."
- 等待用户的书面理由。
- 在阶段 6 之前将理由文本嵌入部分批准记录：追加一个 "⚠️ Override Justification: [用户文本]" 字段。
- 然后才进入阶段 6。

### 阶段 6：部署（如果 GO）
委托给 **release-manager** + **devops-engineer**：
- 在版本控制中打发布标签
- 使用 `/changelog` 生成 changelog
- 部署到 staging 进行最终烟雾测试
- 部署到生产环境
- 人工团队操作：发布后 48 小时内监控仪表板和错误率。在 48 小时节点安排使用 `/retrospective` 进行复盘。

委托给 **community-manager**（与部署并行）：
- 使用 `/patch-notes [version]` 完成补丁说明
- 准备发布公告（商店页面更新、社交媒体、社区帖子）
- 如果有 S3+ 问题发布，起草已知问题帖子
- 输出：所有面向玩家的发布沟通，在部署确认后即可发布

### 阶段 7：发布后
- **release-manager**：生成发布报告（发布了什么、延期了什么、指标）
- **producer**：更新里程碑跟踪，与利益相关者沟通
- **qa-lead**：监控传入的 bug 报告以发现回归
- **community-manager**：发布所有面向玩家的沟通，监控社区情绪
- **analytics-engineer**：确认实时仪表板健康；如果任何关键事件缺失则告警
- 如果出现问题，安排发布后复盘

## 错误恢复协议

如果任何生成的 agent（通过 Task）返回 BLOCKED、错误或无法完成：

1. **立即暴露**：在继续到依赖阶段之前向用户报告 "[AgentName]: BLOCKED — [原因]"
2. **评估依赖**：检查被阻塞 agent 的输出是否是后续阶段所需的。如果是，未经用户输入不要超过该依赖点。
3. **提供选项** 通过 AskUserQuestion 选择：
   - 跳过此 agent 并在最终报告中注明缺口
   - 用更窄的范围重试
   - 停在这里先解决阻塞项
4. **始终生成部分报告** — 输出任何已完成的内容。永远不要因为一个 agent 阻塞而丢弃工作。

常见阻塞项：
- 输入文件缺失（未找到 story，GDD 不存在） → 重定向到创建它的 skill
- ADR 状态为 Proposed → 不要实现；先运行 `/architecture-decision`
- 范围太大 → 通过 `/create-stories` 拆分为两个 story
- ADR 和 story 之间的指令冲突 → 暴露冲突，不要猜测

## 文件写入协议

所有文件写入（发布检查清单、changelog、补丁说明、部署脚本）都委托给 sub-agent 和 sub-skill。每个都执行 "May I write to [path]?" 协议。此编排器不直接写入文件。

## 输出

一份总结报告，涵盖：发布版本、范围、质量门控结果、上线/搁置决策、部署状态和监控计划。

结论：**COMPLETE** — 发布已执行并部署。
结论：**BLOCKED** — 发布已停止；上线/搁置决策为 NO 或硬阻塞项未解决。

## 下一步

- 发布后 48 小时内监控仪表板。
- 如果发布期间出现重大问题，运行 `/retrospective`。
- 成功部署后将 `production/stage.txt` 更新为 `Live`。
