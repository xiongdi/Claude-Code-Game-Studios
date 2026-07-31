---
name: playtest-report
description: "Generates a structured playtest report template or analyzes existing playtest notes into a structured format. Use this to standardize playtest feedback collection and analysis."
argument-hint: "[new|analyze path-to-notes] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Task, AskUserQuestion
model: sonnet
---

## Phase 1: 解析参数

解析审查模式（一次，存储供本次运行的所有 gate spawn 使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

参见 `.claude/docs/director-gates.md` 了解完整的审查模式。

确定模式：

- `new` → 生成空白的 playtest 报告模板
- `analyze [路径]` → 读取原始笔记并用结构化发现填充模板

---

## Phase 2A: 新模板模式

生成此模板并输出给用户：

```markdown
# Playtest 报告

## 会话信息
- **日期**: [日期]
- **构建版本**: [版本/Commit]
- **时长**: [游戏时间]
- **测试者**: [姓名/ID]
- **平台**: [PC/主机/移动端]
- **输入方式**: [键鼠/手柄/触摸]
- **会话类型**: [首次/回访/定向测试]

## 测试重点
[正在测试的具体功能或流程]

## 第一印象（前 5 分钟）
- **理解目标了吗？** [是/否/部分]
- **理解操作了吗？** [是/否/部分]
- **情绪反应**: [投入/困惑/无聊/沮丧/兴奋]
- **笔记**: [观察]

## 玩法流程
### 运行良好的部分
- [观察 1]

### 痛点
- [问题 1 — 严重程度: 高/中/低]

### 困惑点
- [玩家感到困惑的地方及原因]

### 愉悦时刻
- [让玩家感到惊喜或开心的部分]

## 遇到的 Bug
| # | 描述 | 严重程度 | 可复现 |
|---|-------------|----------|-------------|

## 功能特定反馈
### [功能 1]
- **理解其用途了吗？** [是/否]
- **觉得有趣吗？** [是/否]
- **建议**: [测试者建议]

## 量化数据（如有）
- **死亡次数**: [计数和位置]
- **每区域耗时**: [明细]
- **使用物品**: [什么以及何时]
- **发现 vs 遗漏的功能**: [列表]

## 整体评估
- **会再玩一次吗？** [是/否/也许]
- **难度**: [太简单/刚好/太难]
- **节奏**: [太慢/好/太快]
- **会话长度偏好**: [更短/好/更长]

## 本次会话的 Top 3 优先事项
1. [最重要的发现]
2. [第二优先]
3. [第三优先]
```

---

## Phase 2B: 分析模式

读取所提供路径处的原始笔记。与现有设计文档交叉引用。用结构化发现填充上述模板。标记任何与设计意图冲突的 playtest 观察。

---

## Phase 3: 行动路由

将所有发现分类到四个桶中：

- **需要设计变更** — 乐趣问题、玩家困惑、机制损坏、与 GDD 预期体验冲突的观察
- **平衡性调整** — 数值感觉不对、难度波动过大或过平
- **Bug 报告** — 可复现的明确实现缺陷
- **Polish 项目** — 不阻塞进度，但属于后期需要处理的摩擦或手感问题

展示分类列表，然后路由：

- **设计变更：** "对受影响的设计文档运行 `/propagate-design-change [路径]`，在变更前找出下游影响。"
- **平衡性调整：** "运行 `/balance-check [系统]` 在调优数值前验证完整的平衡性状况。"
- **Bug：** "使用 `/bug-report` 正式跟踪这些问题。"
- **Polish 项目：** "当团队进入该阶段时，添加到 `production/` 中的 polish 待办列表。"

---

## Phase 3b: Creative Director 玩家体验审查

**审查模式检查** — 在 spawn CD-PLAYTEST 之前应用：
- `solo` → 跳过。注意："CD-PLAYTEST 已跳过 — Solo 模式。" 继续到 Phase 4（保存报告）。
- `lean` → 跳过（不是 PHASE-GATE）。注意："CD-PLAYTEST 已跳过 — Lean 模式。" 继续到 Phase 4（保存报告）。
- `full` → 正常 spawn。

在对发现进行分类后，使用 gate **CD-PLAYTEST**（`.claude/docs/director-gates.md`）通过 Task  spawn `creative-director`。

传递：结构化的报告内容、游戏支柱和核心幻想（来自 `design/gdd/game-concept.md`）、正在测试的具体假设。

在保存报告之前展示 creative director 的评估。如果结果是 CONCERNS 或 REJECT，在报告中添加 `## Creative Director Assessment` 部分，记录判定和反馈。如果 APPROVE，在报告中注明批准。

---

## Phase 4: 保存报告

询问："我可以将此 playtest 报告写入 `production/qa/playtests/playtest-[日期]-[测试者].md` 吗？"

如果同意，写入文件，必要时创建目录。

---

## Phase 5: 后续步骤

判定：**COMPLETE** — playtest 报告已生成。

- 首先处理最高优先级的发现类别。
- 处理完设计变更后：在更新后的 GDD 上重新运行 `/design-review`。
- 修复 bug 后：重新运行 `/bug-triage` 以更新优先级。
