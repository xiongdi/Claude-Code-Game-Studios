# 示例会话：范围危机 — 战略决策

**背景：** 独立开发者，项目第8周（月第2），接近 Alpha 里程碑。

**危机：**
- Alpha 里程碑还有2周
- 制作系统（支柱2）尚未实现
- 首席程序员说完整制作需要3周
- Alpha 的投资者演示 — 必须展示所有支柱

**会话时长：** 约25分钟
**轮次：** 8
**结果：** 带文档化 ADR 的战略范围决策

---

## 第1轮：用户向创意总监升级问题

**用户：**
> 我们遇到问题了。Alpha 还有2周。制作系统（支柱2"涌现发现"）还没有实现。首席程序员说完整实现需要3周。但投资者演示在 Alpha，他们需要看到两个支柱（战斗+制作）。我该怎么办？

## 第2轮：创意总监收集完整上下文

**Agent（creative-director）：**
> 这是经典的支柱 vs. 进度冲突 - 这正是我来帮助解决的。让我在推荐选项之前收集完整上下文。
>
> 首先让我审查相关文档：
> - `production/milestones/alpha.md`（里程碑定义）
> - `design/pillars.md`（什么是不可妥协的）
> - `design/gdd/crafting-system.md`（已设计的内容）
> - `production/sprints/sprint-current.md`（当前工作）
>
> [读取文档]
>
> **已理解的上下文：**
> - 支柱2："通过实验的涌现发现"（制作是主要表达）
> - Alpha 成功标准："所有支柱以可玩形式演示"
> - 投资者演示目的：验证愿景并确保下一轮融资
> - 当前制作设计：基于标签的推理系统，12个标签，约50个配方，渐进提示解锁
>
> **澄清问题：**
> 1. Alpha 日期是**硬性**（投资者不会改期）还是**弹性**（我们可以请求多1周）？
> 2. **最低制作**是什么，仍然能演示支柱2？（一个配方？十个配方？完整标签系统？）
> 3. 如果完全砍掉制作会**出什么状况**？（游戏还能有意义吗？）
> 4. 这个投资者关系有多**关键**？（可有可无的融资 vs. 项目生存的成败在此一举？）
> 5. **团队状态**如何？（赶工烧坏了？还是fresh可以冲刺？）

## 第3轮：用户提供决策上下文

**用户：**
> 1. 硬性日期 - 投资者专程飞来演示，不能改期
> 2. 我认为最低是：发现机制工作（标签系统）+ 10个配方。不需要打磨。
> 3. 没有制作，Alpha 只能展示战斗。这是50%的愿景 - 投资者可能认为这只是另一个战斗游戏
> 4. 成败在此一举。这个投资者是我们的唯一线索。没有融资 = 项目死亡。
> 5. 就我一个人，我不累。

## 第4轮：创意总监框架决策

**Agent：**
> 谢谢。现在我理解真正危险的是什么了。让我proper地框架这个决策：
>
> **核心问题：**
> 我们是交付完整制作并推迟 Alpha，还是交付简化制作并达成 Alpha，还是砍掉制作并冒着投资者看不到我们完整愿景的风险？
>
> **真正危险的是什么：**
> - **愿景完整性**：投资者能看到支柱2吗？
> - **进度信任**：我们遵守 Alpha 日期的承诺吗？
> - **项目生存**：没有融资 = 没有游戏
> - **质量标准**："粗糙但能用"对这个演示可接受吗？
>
> **决策标准（按此场景的优先级顺序）：**
> 1. **投资者信心**（最高 - 项目生存依赖于此）
> 2. **支柱代表性**（高 - 他们需要看到完整愿景）
> 3. **进度完整性**（中等 - 重要，但不如融资）
> 4. **打磨质量**（最低 - 垂直切片可以粗糙）
>
> 现在让我展示三个战略选项...

## 第5轮：创意总监展示选项及分析

**Agent：**
> **选项A：实现完整制作系统（3周，Alpha 推迟1周）**
>
> **执行：**
> - 实现完整的基于标签的系统，包含所有50个配方
> - 完整的 UI 打磨，所有 GDD 功能
> - 错过 Alpha 日期，重新安排投资者演示（+1周）
>
> **优点：**
> - ✅ 支柱2完全代表，达到production质量
> - ✅ 无设计妥协
> - ✅ 第一印象是打磨过的
>
> **缺点：**
> - ❌ 打破投资者承诺（他们按原定日期飞来）
> - ❌ 发出进度管理问题信号（"如果他们推迟 Alpha，他们会推迟发布吗？"）
> - ❌ 可能比粗糙演示更损害投资者信任
>
> **风险：严重**
> - 投资者可能不会改期（忙碌的日程，其他机会）
> - 即使他们改期，推迟也损害信誉
> - 鉴于硬性日期限制，不太可能成功
>
> **判定：不推荐** - 日期是硬性的，不可协商
>
> ---
>
> **选项B：简化制作到核心支柱（1.5周，达到 Alpha）**
>
> **执行：**
> - 仅实现核心标签发现机制
> - 10个配方而不是50个（足以演示系统）
> - 基础 UI（功能性的，不打磨）
> - 无渐进提示解锁（为演示清晰度，所有标签在技能0可见）
> - 无失败尝试反馈系统（只有成功/失败）
>
> **范围："支柱2的垂直切片"**
> - 玩家可以通过组合带标签的成分发现配方
> - 玩家可以学习标签模式并推断新配方
> - 支柱2的**灵魂**存在，只是未打磨
>
> **优点：**
> - ✅ 达到 Alpha 截止日期（保持进度完整性）
> - ✅ 演示了支柱2（投资者看到完整愿景）
> - ✅ 核心机制可玩且可验证
> - ✅ 可以用"Alpha → Beta 改进"作为叙事（"这是粗糙的，这是打磨的"）
>
> **缺点：**
> - ⚠️ 第一印象是"粗糙"（但垂直切片预期是粗糙的）
> - ⚠️ 投资者可能觉得"这感觉很小"（缓解：用展示路线图说明 Alpha 后扩展）
>
> **风险：中等**
> - 投资者可能从10个配方中无法"理解"支柱（缓解：好的演示脚本解释愿景）
> - 可能感觉不完整（缓解：框架为"垂直切片证明概念"）
>
> **判定：推荐** - 平衡所有约束
>
> ---
>
> **选项C：从 Alpha 砍掉制作，专注于战斗打磨（0周，达到 Alpha）**
>
> **执行：**
> - 只用支柱1（战斗）完全打磨后交付 Alpha
> - 用额外的2周超级打磨战斗，添加内容，让它闪耀
> - 将制作呈现为"设计文档+概念艺术"（不可玩）
>
> **优点：**
> - ✅ 零压力达到截止日期
> - ✅ 有的东西是 AAA 打磨
> - ✅ 无"半生不熟"感知的风险
>
> **缺点：**
> - ❌ 支柱2未演示（50%愿景缺失）
> - ❌ 投资者看到"另一个战斗游戏" - 无差异化
> - ❌ "潜力" vs. "已验证概念"更难获得融资
> - ❌ 如果战斗本身不能打动他们，我们没有退路
>
> **风险：高**
> - 投资者看不到让这个游戏特别的地方
> - 他们可能认为"战斗不错，但亮点在哪？"
> - 竞争游戏有战斗 - 我们需要展示我们的差异化
>
> **判定：不推荐** - 对于成败在此一举的演示风险太大
>
> ---
>
> ## 我的推荐：选项B（简化制作核心）
>
> **推理：**
> 1. **投资者信心**：按我们承诺的日期交付展示可靠性
> 2. **支柱可见性**：粗糙但能工作 > 打磨但缺失。投资者投资**愿景**，不是打磨。
> 3. **演示叙事**："这是 Alpha vs. Beta 进展"引人注目 - 展示轨迹
> 4. **风险缓解**：如果制作不成功，战斗仍然扎实。如果砍掉制作（选项C）而战斗本身不能打动他们，我们什么都没有。
> 5. **历史先例**：Hades、Dead Cells、Slay the Spire 都展示了粗糙的 Early Access 垂直切片，投资者喜欢**系统**而不是打磨
>
> **选项B牺牲了什么：**
> - 打磨（对垂直切片可接受）
> - 功能完整性（50个配方 → 10个）
> - 渐进解锁（推迟到 Alpha 后）
>
> **选项B保护什么：**
> - 进度信任
> - 支柱代表性
> - 愿景清晰度
> - 项目生存（融资确保）
>
> **如果你选择选项B的实施计划：**
> 1. 我将创建 ADR 记录此范围决策
> 2. 我将用"Alpha 范围"标记更新 `design/gdd/crafting-system.md`
> 3. 我将向 gameplay-programmer 简要介绍简化实施
> 4. 我将为你创建演示脚本：如何呈现10配方系统来展示50配方的愿景
>
> **但这是你的决定。** 你比我更了解这个投资者。如果你相信他们会重视打磨而不是支柱代表性，选项C可能是对的。如果你认为他们会原谅1周的推迟，选项A创意上最安全。
>
> 你的决定是什么？

## 第6轮：用户做出决定

**用户：**
> 选项B。投资者是技术人员，即使粗糙也会欣赏系统思维。让我们简化制作范围以达到 Alpha。你能创建 ADR 并更新 GDD 吗？

## 第7轮：创意总监记录决策

**Agent：**
> 好的决定。我将彻底记录这个，以便未来的你能理解我们为什么做出这个选择。
>
> **创建：**
> 1. ADR-007：Alpha 制作范围缩减
> 2. 更新的 design/gdd/crafting-system.md，带 Alpha/Beta 范围标记
> 3. 更新的 production/milestones/alpha.md 成功标准
> 4. 投资者演示指导的演示脚本
>
> [创建文档]
>
> **ADR 摘要：**
> ```markdown
> ## Decision
> Simplify crafting system to core tag discovery mechanic (10 recipes)
> for Alpha milestone. Defer full implementation (50 recipes + progressive
> unlocks) to post-Alpha.
>
> ## Context
> - Alpha in 2 weeks (hard date, investor demo)
> - Full crafting requires 3 weeks
> - Investor demo is make-or-break for funding
>
> ## Consequences
> Positive:
> - Makes Alpha deadline, preserves investor trust
> - Both pillars demonstrated (rough but playable)
> - Can show Alpha→Beta improvement narrative
>
> Negative:
> - First impression is "rough"
> - Limited recipe variety
> - Missing progressive unlock system
>
> ## Validation Criteria
> We'll know this was the right choice if:
> - Investor demo successfully shows both pillars
> - Funding secured for next development phase
> - Post-Alpha sprint can implement full system without rework
> ```
>
> **更新的 GDD 标记：**
> ```markdown
> ## Detailed Rules
>
> ### Core Loop (**ALPHA SCOPE**)
> [Core mechanic description]
>
> ### Progressive Unlocks (**POST-ALPHA**)
> [Skill-based hint system - deferred]
>
> ### Recipe Catalog
> - Alpha: 10 recipes (demonstrates tag variety)
> - Beta: 50 recipes (full content)
> ```
>
> **演示脚本指导：**
> ```markdown
> ## Presenting Crafting to Investor (10-recipe Alpha)
>
> **Frame it as proof-of-concept:**
> "This is a vertical slice of the crafting pillar. You'll see 10 recipes
> here, but the SYSTEM supports unlimited recipes through tag combinations.
> Watch how players deduce patterns..."
>
> **Show, don't tell:**
> - Live demo: Discover 2 recipes on the fly
> - Explain: "Notice Fire+Water made Steam Potion. Now the player knows
>   Fire+Water is valid. They can deduce that Fire+Earth might work..."
> - Connect to pillar: "This is Emergent Discovery - the game teaches rules,
>   players become scientists."
>
> **Roadmap slide:**
> "Alpha: 10 recipes (proof of concept)
>  Beta: 50 recipes (content expansion)
>  Release: 100+ recipes (includes community-designed recipes)"
>
> **Emphasize systems:**
> "We're not making a content game, we're making a systems game. The tag
> system can generate endless combinations. We're building a platform for
> discovery, not a recipe database."
> ```
>
> 可以继续写这些文档吗？

## 第8轮：用户批准，Agent 完成

**用户：**
> 是的，创建所有这些文档，并通知 gameplay-programmer 范围变更。

**Agent：**
> ✅ 完成！创建/更新：
> - `docs/architecture/adr-007-alpha-crafting-scope.md`
> - `design/gdd/crafting-system.md`（添加了 Alpha/Beta 范围标记）
> - `production/milestones/alpha.md`（更新了成功标准）
> - `production/demo-script-investor-alpha.md`（新文件，含演示指南）
>
> **通知已发送：**
>
> 给 `gameplay-programmer`：
> > "制作范围为 Alpha 简化：核心标签机制 + 仅10个配方。
> > 渐进解锁和完整配方集推迟到 Alpha 后。更新的 GDD 有实施指导。估计1.5周实施 Alpha 范围。"
>
> 给 `producer`：
> > "战略决策已做出：简化制作以达到 Alpha 截止日期。
> > ADR-007 记录了理由。Sprint 计划需要更新以反映新范围。"
>
> **下一步：**
> 1. Gameplay-programmer 可以开始实施简化制作
> 2. 你可以在投资者会议前练习演示脚本
> 3. Producer 将用新任务分解更新 sprint 计划
>
> **验证：**
> 当以下发生时，我们将知道这个决定是正确的：
> - Alpha 按时交付，两个支柱可玩
> - 投资者演示成功传达愿景
> - 融资确保用于下一开发阶段
>
> 演示顺利！这个决定是对的 - 对这个听众，愿景优于打磨。

---

## 会话摘要

**什么效果很好：**
- 创意总监在提出解决方案前阅读上下文文档
- 提出5个澄清问题以理解约束
- proper地框架决策（危险是什么，评估标准）
- 展示3个不同选项带诚实的优缺点
- 提出明确推荐带推理，但 defer 给用户
- 用 ADR彻底记录决策并更新所有受影响的文档
- 提供演示脚本帮助用户积极呈现决策

**关键协作时刻：**
- 第2轮：Agent 在跳到解决方案前收集完整上下文
- 第3轮：用户提供决策标准优先级（融资 > 进度 > 打磨）
- 第5轮：Agent 展示3个选项，提出推荐，但明确"这是你的决定"
- 第6轮：用户做出战略决策
- 第7轮：Agent 提供演示脚本支持用户的呈现
- 第8轮：Agent 将决策级联到受影响的各方

**战略决策质量：**
- 用户做出最终决定
- Agent 使用游戏开发理论和先例提供专家分析
- 权衡是明确和诚实的
- 决策被记录供将来参考
- 定义了验证成功标准

**创建/更新的文件：**
- ADR-007（决策记录）
- 更新的制作 GDD 带范围标记
- 更新的 Alpha 里程碑定义
- 投资者演示的演示脚本

**上下文使用：** ~30%
**用户满意度：** 高 - 用完整信息做出自信决策
