---
name: vertical-slice
description: "Pre-Production validation — build a production-quality end-to-end build to confirm the full game loop is achievable before committing to Production. Run after GDDs, architecture, and UX specs are complete. Produces a PROCEED/PIVOT/KILL verdict that gates the Pre-Production → Production transition."
argument-hint: "[--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion
model: sonnet
agent: prototyper
isolation: worktree
---

## 目的

**垂直切片**回答了一个与概念原型不同的问题：
*"我们能否按计划以这个制作质量构建完整的游戏循环？"*

**默认使用** — 在 Pre-Production 后期运行，在 GDD、架构和 UX 规范完成后。它是一个接近制作质量的构建，展示一个完整的 [开始 → 挑战 → 解决] 周期。

**转向后？** 如果早期垂直切片的 PIVOT 结论让你回去修改 GDD 和架构，在修改后再次运行此 skill 以重新验证。可以运行任意次，直到达到 PROCEED 或 KILL 结论。

它验证：

1. 管线（团队能否实际制作这个质量的内容？）
2. 执行可行性（架构决策对这款游戏是否正确？）
3. 乐趣存活（概念原型的乐趣能否在完整设计中存活？）
4. **速度**（这花了多长时间？那是你真正的制作速率估算。）

**在项目早期？** 如果你还没有写 GDD，想验证核心想法是否值得设计，运行 `/prototype`（概念原型）代替。

---

## 阶段 1：解析审查模式并加载上下文

解析审查模式：
1. 如果传入了 `--review [full|lean|solo]` → 使用该模式
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

参见 `.claude/docs/director-gates.md` 了解完整的检查模式。

阅读以下文件以理解完整的设计意图：
- `CLAUDE.md` — 技术栈和引擎
- `design/gdd/game-concept.md` — 核心幻想和游戏支柱
- `design/gdd/systems-index.md` — MVP 系统及其优先级
- `docs/architecture/architecture.md` — 层级结构
- `docs/architecture/control-manifest.md` — 实现的技术规则
- 正在切片的系统的关键 GDD

---

## 阶段 2：定义切片范围和验证问题

在构建之前，定义 **可证伪的验证问题**：

> *"一个玩家，从零开始，能否在 [N] 分钟内体验到 [game-concept.md 中的核心幻想]，无需开发者指导 — 我们能否在 [X] 天内以代表性质量构建一个这样的循环？"*

两部分都重要：玩家体验和构建可行性。

**范围纪律：**
- 包含所有核心循环系统（最低要求）。如果一个系统需要完成一个 [开始 → 挑战 → 解决] 周期，它必须在切片中。
- **目标范围：3-5 分钟的打磨过的连续游戏玩法。** 这是行业标准的垂直切片长度 — 足够长以展示机制和基调，足够短以代表性质量构建。如果你的切片需要超过 5 分钟才能玩完，削减内容，而不是质量。
- **在削减质量之前削减范围。** 一个看起来与预期游戏完全不同的高质量切片无法验证制作可行性。
- 如果范围感觉太大无法在 1-3 周内构建，切片范围是错误的 — 不是太大无法构建，而是一次试图证明太多。

**范围蔓延警告：** 垂直切片是 pre-production 阶段范围蔓延的最高风险时刻。功能感觉"差不多完成了"，很诱人想添加"再多一个系统"。抵制这种诱惑。削减，不要扩展。

在构建之前向用户展示范围并获得确认。

---

## 阶段 3：规划构建

用要点定义：
- 实现的系统（正在使用哪些 GDD 部分）
- 完整的游戏循环周期（[开始] → [挑战] → [解决] 确切是什么）
- 美术和音频质量级别（占位符可接受，代表性优先）
- 验证问题的具体、可衡量的成功标准
- 硬性时间限制：[X] 天。如果超出，范围是错误的 — 停止并重新评估。

在构建之前要求用户确认范围。

确认后，将会话检查点写入 `production/session-state/active.md`（如果不存在则创建 `production/session-state/`）。包含：概念名称、验证问题、范围内的系统、美术质量级别和当前阶段（"阶段 4 — 实现"）。在每个构建日结束时更新此文件，记录完成的内容。如果会话在切片中间结束，这是主要的恢复机制 — 多周的 Engine 构建将跨越多个会话。

---

## 阶段 4：实现

询问："May I create the vertical slice directory at `prototypes/[concept-name]-vertical-slice/` and begin implementation?"

如果同意，创建目录。每个文件必须以以下内容开头：

```
// VERTICAL SLICE - NOT FOR PRODUCTION
// Validation Question: [What this build is proving]
// Date: [Current date]
```

**质量标准** — 高于概念原型，不是完整制作：
- 遵循 `docs/architecture/control-manifest.md` 中的架构层级
- 遵循 `.claude/docs/technical-preferences.md` 中的命名约定
- 没有硬编码的游戏玩法值 — 使用常量或配置文件
- 关键路径上的基本错误处理
- 占位符美术可接受；代表性美术优先

**多轮循环：** 在写入初始文件后，要求用户运行构建并报告他们观察到的内容。迭代直到完整的游戏循环周期可演示。每轮：
1. 用户运行 → 报告错误或观察
2. Agent 修复错误或调整系统
3. 重复直到完整的 [开始 → 挑战 → 解决] 周期可玩

**沉没成本检查点（计划时间线的第 3 天）：** 如果完整的游戏循环周期尚不可演示，停止并重新评估。要么范围太大，要么架构假设错误。明确呈现阻塞项，而不是继续迭代。

一旦循环可演示，进行至少 1 次试玩会话。

**试玩提示：** 如果你能让任何没见过游戏的人玩它 — 朋友、家人、在线社区 — 安静地观察他们，不要解释任何东西。不要引导他们。他们的困惑揭示了游戏自身没有传达的内容。这比自我测试给出更好的信号。

**没有外部测试者？** 在团队内轮换：开发者 A 构建了系统 X，所以开发者 A 是系统 Y 的朴素测试者。即使是两人团队也能有效轮换？独自一人？离开 2-3 天然后以新玩家的身份玩一遍 — 你不会拥有完美的第一印象信号，但你会抓住关键的阻塞项。还要尝试"无声演练"：一口气玩你自己的切片，不停下来修复任何东西，记录你犹豫的每一个时刻。

**想要更丰富的观察数据？** 要求测试者边玩边 **大声思考** — 实时叙述他们在做什么和为什么。"I'm trying to figure out how to attack... I pressed E... nothing... is it click?" 这会在困惑发生时立即呈现它，而不是回顾。最适合新手引导和 UI 清晰度验证。无声观察仍然更适合感觉测试；大声思考稍微改变了体验，但产生更细粒度的 UX 数据。

**异步远程选项：** 录制 Loom 或 OBS 会话 — 给某人构建，要求他们录制屏幕 + 音频，然后发给你视频。你获得真正的第一印象数据，无需同步安排。跨时区有效。

**在完全实现之前测试 AI、NPC 或复杂系统行为？** 使用 **Wizard of Oz** 技术：一个人正常玩，而第二个人秘密实时控制 NPC 或系统行为。玩家相信它是自动化的。这在实现完成之前验证了 AI 或经济系统的 *设计意图* — 并揭示了系统必须产生什么行为才能感觉正确。对于 AI 系统在范围内但尚未打磨到足以进行无引导测试的垂直切片特别有用。

---

## 阶段 5：试玩汇报

循环可演示。在写报告之前，从实际游玩中收集结构化观察。不要跳到报告生成 — 报告只和你在这里捕获的观察一样好。

准确说这句话：
> "Play through the complete [start → challenge → resolution] cycle from scratch, as if you're a new player with no knowledge of how it was built. Don't skip ahead or use developer shortcuts. Come back when you've completed the full loop — or when you've hit something that stopped you."

用户返回后，**逐个**询问这些问题：

1. **循环完成：**
   > "Did you complete the full [start → challenge → resolution] cycle on your own, without needing any guidance from me or prior knowledge of the build?"

2. **时间检查：**
   > "How long did it take to reach the first meaningful action — the first moment where you felt like you were actually playing the game?"

3. **核心幻想：**
   > "The game is supposed to make you feel [core fantasy from game-concept.md]. Did it? Be honest — not 'kind of' but specifically what you felt and when."

4. **阻塞项：**
   > "What stopped you, confused you, or pulled you out of the experience? Any moment where you weren't sure what to do, or where something broke?"

5. **管线检查：**
   > "As the developer — not the player — does this feel achievable at this quality for the full game? What surprised you about how long things took to build?"

6. **结论：**
   > "PROCEED, PIVOT, or KILL — and the specific reason."

如果任何回答模糊，询问："Can you give me the specific moment where that happened?" 精确的观察填充报告。模糊的观察产生无用的报告。

---

## 阶段 6：生成垂直切片报告

在整个构建过程中跟踪速度。记录：
- 第 1 天：构建了什么
- 第 2 天：构建了什么
- 等等。

这是你拥有的关于制作速率的最诚实的数据。不要跳过它。它直接输入 sprint 规划。

读取 `.claude/docs/templates/vertical-slice-report.md` 获取报告结构。
如果未找到模板文件，使用此回退结构：
- `## Vertical Slice Report — [Game Title] — [Date]`
- `### Executive Summary`（PROCEED / PIVOT / STOP 结论 + 2 句理由）
- `### Core Loop Validation`（测试了什么，什么通过，什么失败）
- `### Feel Assessment`（动画、控制、反馈 — 主观笔记）
- `### Technical Findings`（性能、引擎问题、架构风险）
- `### Velocity Log`（逐日实际进度 — 不要跳过）
- `### Recommended Next Steps`

根据此会话期间观察和构建的内容填写每个部分。速度日志必须反映实际逐日进度，而不是估算 — 这是你拥有的最诚实的制作速率数据。用真实观察替换所有占位符文本。

### 经验教训
- 哪些假设被实际构建到接近制作质量所打破？
- 管线或架构有什么让我们惊讶的？
- 如果我们再次运行这个，我们会改变切片范围的什么？
```

询问："May I write this report to `prototypes/[concept-name]-vertical-slice/REPORT.md`?"

如果同意，写入文件。然后更新 `prototypes/index.md`（如果不存在则创建）— 在垂直切片表格中追加一行：概念名称、日期、结论和 REPORT.md 的链接。注明这是首次运行切片还是 PIVOT 后的重新运行。此报告中的速度日志是项目中最有价值的数据之一 — 将其与 sprint 估算交叉引用。

---

## 阶段 7：创意总监审查

**审查模式检查：**
- `solo` → 跳过。注意："CD-PLAYTEST skipped — Solo mode."
- `lean` → 跳过（不是 PHASE-GATE）。注意："CD-PLAYTEST skipped — Lean mode."
- `full` → 通过 Task 使用门控 **CD-PLAYTEST** 生成 `creative-director`（`.claude/docs/director-gates.md`）。

传递：完整的 REPORT.md 内容、验证问题、`design/gdd/game-concept.md` 中的游戏支柱和核心幻想。

创意总监根据游戏的创意愿景和支柱评估垂直切片结果，然后确认、修改或覆盖建议。他们的结论是最终的。如果结论不同，更新 REPORT.md。

---

## 阶段 8：总结和下一步

输出总结：验证问题、速度数据和最终建议。
链接到 `prototypes/[concept-name]-vertical-slice/REPORT.md`。

**如果 PROCEED：**
你的垂直切片验证了完整的游戏循环。项目已准备好进入 Production。

推荐的下一步：
- `/create-epics layer:foundation` — 规划 Foundation 层 epic
- `/create-epics layer:core` — 规划 Core 层 epic
- `/create-stories [epic-slug]` — 将每个 epic 分解为可实现的 story
- `/sprint-plan` — 使用切片的速度数据规划第一个 sprint
- `/gate-check pre-production` — 正式将阶段推进到 Production

**试玩注意：** `/gate-check` 将寻找记录的试玩证据。
最低要求，1 个有 REPORT.md 显示 PROCEED 的记录会话是通过门控所必需的。更多会话给出更可靠的信号 — 在让整个团队投入 Production 之前推荐 3+，但不是硬性门控。

**如果 PIVOT：**

在路由回 GDD 修订之前，捕获继续前进的注意。询问这两个问题（纯文本，一次一个）：

1. "What systems or mechanics worked at this quality level and should be preserved in the revised design?"
2. "What specifically failed — the core loop, the architecture, the pipeline, or the fun?"

询问："May I write this to `prototypes/[concept-name]-vertical-slice/PIVOT-NOTE.md`?"

如果同意，写入文件，包含：什么有效、什么失败、需要修订的具体系统或架构决策，以及下一个切片应该证明什么不同。下次运行 `/vertical-slice` 后 PIVOT，检查 `prototypes/` 目录中的 `PIVOT-NOTE.md` — 用它来构建新的验证问题并告知范围决策。

- 使用 `/design-system [mechanic]` 修订受影响的 GDD
- 通过 `/architecture-decision` 解决架构问题
- 然后重新运行 `/vertical-slice` 验证修订后的方向

**如果 KILL：**

在放弃概念之前，确认结论是合理的：

- [ ] 完整的游戏循环即使对有经验的玩家也花费 >5 分钟？
- [ ] 在任何试玩会话中没有观察到情感高点（喜悦、惊讶、满足）？
- [ ] 50%+ 的测试者在 2+ 次切片尝试后在同一点困惑或卡住？
- [ ] 架构问题需要重建超过 50% 的已构建内容？
- [ ] 这是同一概念的第三次垂直切片尝试？

如果 2+ 个复选框适用 → KILL 结论是合理的。如果 0-1 个适用 → 一次有针对性的 PIVOT 可能挽救概念。

**在 `prototypes/GRAVEYARD.md` 中记录终止**（如果不存在则创建）。
询问："May I append this to `prototypes/GRAVEYARD.md`?" 如果同意，添加一个条目：

```
## [Concept Name] Vertical Slice — YYYY-MM-DD
- **Kill reason:** [what specifically prevented the player from experiencing the core fantasy]
- **What worked at slice quality:** [systems or mechanics that held up]
- **What failed:** [core loop issue, architecture decision, or pipeline blocker]
- **Next time:** [one specific change for the next time a similar concept is attempted]
```

- 带着你学到的东西返回 `/brainstorm`
- 或运行 `/prototype [new-concept]` 先便宜地测试新方向

---

### 重要约束

- 垂直切片代码绝不能重构为制作代码 — 仅供参考
- 制作代码绝不能从 `prototypes/` 导入
- 如果建议是 PROCEED，制作实现从头开始编写，仅将切片作为设计参考
- 范围削减是可接受的；质量削减不可接受 — 低质量的切片证明不了什么
- 总工作量：1-3 周。如果更长，范围太大 — 削减切片，而不是质量。
- 第 3 天沉没成本规则：如果到那时完整的游戏循环周期尚不可演示，停止并呈现阻塞项
- **网络化/多人游戏：** 本地垂直切片无法验证网络化机制的感觉。延迟从根本上改变了战斗、移动和预测的感觉 — 在 0ms 本地测试会与在 80ms 网络延迟下完全不同。切片可以验证游戏循环有趣且完整；它无法验证网络化机制在真实条件下感觉良好。网络感觉需要真实对等体或模拟延迟。
