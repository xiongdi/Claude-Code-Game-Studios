# 导演门控 — 共享评审模式

本文档为所有工作流程阶段的所有导演和主管评审定义标准门控提示。
技能从本文档引用门控 ID，而不是将完整提示内联 — 消除了提示需要更新时的漂移。

**范围**：全部 7 个生产阶段（Concept → Release）、全部 3 个第一层导演、
所有关键第二层主管。任何技能、团队编排器或工作流程都可以调用这些门控。

---

## 如何使用本文档

在任何技能中，用引用替换内联导演提示：

```
通过 Task 派生 `creative-director`，使用来自
`.claude/docs/director-gates.md` 的门控 **CD-PILLARS**。
```

传递该门控 **Context to pass** 字段下列出的上下文，然后使用下面的
**Verdict handling** 规则处理裁决。

---

## 评审模式

评审强度控制是否运行导演门控。可以全局设置（跨会话保持）
或按技能运行覆盖。

**全局配置**：`production/review-mode.txt` — 一个词：`full`、`lean` 或 `solo`。
在 `/start` 期间设置一次。随时直接编辑文件以更改。

**每次运行覆盖**：任何使用门控的技能接受 `--review [full|lean|solo]` 作为
参数。这仅对该次运行覆盖全局配置。

示例：
```
/brainstorm space horror           → 使用全局模式
/brainstorm space horror --review full   → 强制本次 full 模式
/architecture-decision --review solo     → 跳过所有门控本次运行
```

| 模式 | 运行内容 | 适用于 |
|------|-----------|----------|
| `full` | 所有门控活跃 — 每个工作流程步骤都被审查 | 团队、学习用户、或当你希望在每一步都获得彻底的导演反馈 |
| `lean` | 仅 PHASE-GATE（`/gate-check`）— 跳过每个技能的门控 | **默认** — solo 开发者和小队；导演仅在里程碑审查 |
| `solo` | 任何地方都没有导演门控 | Game jam、原型、最大速度 |

**检查模式 — 在每个门控派生前应用：**

```
在派生门控 [GATE-ID] 之前：
1. 如果技能用 --review [mode] 调用，使用那个
2. 否则读取 production/review-mode.txt
3. 否则默认为 full

应用解析的模式：
- solo → 跳过所有门控。输出注明："[GATE-ID] skipped — Solo mode"
- lean → 跳过除非这是 PHASE-GATE（CD-PHASE-GATE、TD-PHASE-GATE、PR-PHASE-GATE）
         输出注明："[GATE-ID] skipped — Lean mode"
- full → 正常派生
```

---

## 调用模式（复制到任何技能）

**强制：每个门控派生前解析评审模式。** 永远不要在没有检查的情况下派生门控。
解析的模式在每次技能运行中确定一次：
1. 如果技能用 `--review [mode]` 调用，使用那个
2. 否则读取 `production/review-mode.txt`
3. 否则默认为 `lean`

应用解析的模式：
- `solo` → **跳过所有门控**。输出注明：`[GATE-ID] skipped — Solo mode`
- `lean` → **跳过除非这是 PHASE-GATE**（CD-PHASE-GATE、TD-PHASE-GATE、PR-PHASE-GATE、AD-PHASE-GATE）。输出注明：`[GATE-ID] skipped — Lean mode`
- `full` → 正常派生

```
# 应用模式检查，然后：
通过 Task 派生 `[agent-name]`：
- Gate: [GATE-ID]（见 .claude/docs/director-gates.md）
- Context: [该门控下列出的字段]
- 在继续前等待裁决。
```

对于并行派生（多个导演在同一门控点）：

```
# 首先对每个门控应用模式检查，然后派生所有幸存者：
通过 Task 同时派生所有 [N] 个 Agent — 在等待任何结果前发出所有 Task 调用。
在继续前收集所有裁决。
```

---

## 标准裁决格式

所有门控返回三种裁决之一。技能必须处理全部三种：

| 裁决 | 含义 | 默认操作 |
|---------|---------|----------------|
| **APPROVE / READY** | 没有问题。继续。 | 继续工作流程 |
| **CONCERNS [列表]** | 存在问题但不阻塞。 | 通过 `AskUserQuestion` 显示给用户 — 选项：`修订标记项目` / `接受并继续` / `进一步讨论` |
| **REJECT / NOT READY [阻塞项]** | 存在阻塞问题。不要继续。 | 将阻塞项显示给用户。在解决前不写入文件或进入下一阶段。 |

**升级规则**：当多个导演并行派生时，应用最严格的裁决 — 一个 NOT READY 覆盖所有 READY 裁决。

---

## 记录门控结果

门控解决后，在相关文档的状态头中记录裁决：

```markdown
> **[Director] Review ([GATE-ID])**: APPROVED [date] / CONCERNS (accepted) [date] / REVISED [date]
```

对于阶段门控，记录在 `docs/architecture/architecture.md` 或
`production/session-state/active.md` 中（视情况而定）。

---

## 第一层 — 创意总监门控

Agent: `creative-director` | 模型层级: Opus | 领域: 愿景、支柱、玩家体验

---

### CD-PILLARS — 支柱压力测试

**触发器**：游戏支柱和反支柱定义后（头脑风暴第 4 阶段，
或任何时候支柱被修订）

**传递的上下文**：
- 完整支柱集（名称、定义、设计测试）
- 反支柱列表
- 核心幻想声明
- 独特钩子（"像 X，但还有 Y"）

**提示**：
> "审查这些游戏支柱。它们是否可以证伪 — 一个真实的设计决策是否可能实际上无法满足这个支柱？它们彼此之间是否创造了有意义的张力？它们是否能将这个游戏与其最接近的比较对象区分开来？它们在实践中是否有助于解决设计分歧，还是太模糊而无法使用？为每个支柱返回具体反馈和总体裁决：APPROVE（强）、CONCERNS [列表]（需要打磨）或 REJECT（弱 — 支柱没有分量）。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### CD-GDD-ALIGN — GDD 支柱一致性检查

**触发器**：系统 GDD 创作后（design-system、quick-design 或任何产生 GDD 的工作流程）

**传递的上下文**：
- GDD 文件路径
- 游戏支柱（来自 `design/gdd/game-concept.md` 或 `design/gdd/game-pillars.md`）
- 此游戏的 MDA 美学目标
- 系统的陈述玩家幻想章节

**提示**：
> "审查此系统 GDD 的支柱一致性。每个章节是否为陈述的支柱服务？是否存在违反或削弱支柱的机制或规则？玩家幻想章节是否与游戏的核心幻想匹配？返回 APPROVE、CONCERNS [有问题的具体章节] 或 REJECT [必须在系统可实现前重新设计的支柱违规]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### CD-SYSTEMS — 系统分解愿景检查

**触发器**：`/map-systems` 写出系统索引后 — 在 GDD 创作开始前验证完整系统集

**传递的上下文**：
- 系统索引路径（`design/gdd/systems-index.md`）
- 游戏支柱和核心幻想（来自 `design/gdd/game-concept.md`）
- 优先级层级分配（MVP / Vertical Slice / Alpha / Full Vision）
- 在依赖映射中识别的高风险或瓶颈系统

**提示**：
> "根据游戏的设计支柱审查此系统分解。完整的 MVP 层系统集合是否共同提供了核心幻想？是否存在其机制不服务于任何陈述支柱的系统 — 表明它们可能是范围蔓延？是否存在没有系统分配来交付的核心幻想玩家体验？核心循环是否缺少必需的系统？返回 APPROVE（系统服务于愿景）、CONCERNS [具体的差距或其支柱含义与对齐问题] 或 REJECT [根本性差距 — 分解错过了关键设计意图，必须在 GDD 创作开始前修订]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### CD-NARRATIVE — 叙事一致性检查

**触发器**：叙事 GDD、传说文档、对话规格或世界构建文档创作后
（team-narrative、故事系统 design-system、writer 交付物）

**传递的上下文**：
- 文档文件路径
- 游戏支柱
- 叙事方向简报或基调指南（如果存在于 `design/narrative/`）
- 新文档引用的任何现有传说

**提示**：
> "审查此叙事内容与游戏支柱和已建立的世界规则的一致性。基调是否与游戏已建立的 voice 匹配？是否与现有传说或世界构建存在矛盾？内容是否为玩家体验支柱服务？返回 APPROVE、CONCERNS [具体不一致] 或 REJECT [破坏世界凝聚力的矛盾]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### CD-PLAYTEST — 玩家体验验证

**触发器**：生成试玩报告后（`/playtest-report`），或任何产生玩家反馈的会话后

**传递的上下文**：
- 试玩报告文件路径
- 游戏支柱和核心幻想声明
- 被测试的具体假设

**提示**：
> "根据游戏的设计支柱和核心幻想审查此试玩报告。玩家体验是否符合预期幻想？是否存在系统性问题代表了支柱漂移 — 单独感觉良好但破坏预期体验的机制？返回 APPROVE（核心幻想正在落地）、CONCERNS [预期和实际体验之间的差距] 或 REJECT [核心幻想不存在 — 在进一步试玩前需要重新设计]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### CD-PHASE-GATE — 阶段转换时的创意就绪

**触发器**：始终在 `/gate-check` — 与 TD-PHASE-GATE 和 PR-PHASE-GATE 并行派生

**传递的上下文**：
- 目标阶段名称
- 存在的所有产物列表（文件路径）
- 游戏支柱和核心幻想

**提示**：
> "从创意方向角度审查当前项目状态是否准备好进入 [目标阶段] 门控。游戏支柱是否在所有设计产物中忠实地代表？当前状态是否保留了核心幻想？是否存在跨 GDD 或架构的任何设计决策损害了预期的玩家体验？返回 READY、CONCERNS [列表] 或 NOT READY [阻塞项]。"

**裁决**：READY / CONCERNS / NOT READY

---

## 第一层 — 技术总监门控

Agent: `technical-director` | 模型层级: Opus | 领域: 架构、引擎风险、性能

---

### TD-SYSTEM-BOUNDARY — 系统边界架构审查

**触发器**：`/map-systems` 第 3 阶段依赖映射达成一致后但 GDD 创作开始前 —
在团队投入编写 GDD 之前验证系统结构在架构上是否健全

**传递的上下文**：
- 系统索引路径（或依赖映射摘要如果索引尚未写入）
- 层级分配（Foundation / Core / Feature / Presentation / Polish）
- 完整依赖图（每个系统依赖什么）
- 任何标记的瓶颈系统（许多依赖者）
- 发现的任何循环依赖及其 proposed resolutions

**提示**：
> "在 GDD 创作开始前从架构角度审查此系统分解。系统边界是否清晰 — 每个系统是否拥有 distinct concern 且最小重叠？是否存在 God Object 风险（系统做太多）？依赖排序是否会造成实现顺序问题？提议边界中是否存在会导致紧密耦合的隐含共享状态问题？是否存在实际上依赖于 Feature 层系统的 Foundation 层系统（反向依赖）？返回 APPROVE（边界在架构上健全 — 继续 GDD 创作）、CONCERNS [具体边界问题在 GDD 本身中解决] 或 REJECT [根本性边界问题 — 系统结构将导致架构问题，必须在任何 GDD 编写前重组]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### TD-FEASIBILITY — 技术可行性评估

**触发器**：在范围/可行性期间识别最大技术风险后（头脑风暴第 6 阶段、
quick-design 或任何有技术未知数的早期阶段概念）

**传递的上下文**：
- 概念的核心循环描述
- 目标平台
- 引擎选择（或"未决定"）
- 识别的技术风险列表

**提示**：
> "审查针对使用 [引擎或'未决定引擎'] 的 [类型] 游戏 targeting [平台] 的技术风险。标记任何可能使所描述概念无效的 HIGH 风险项目、任何引擎特定且应影响引擎选择的风险，以及任何 solo 开发者常被低估的风险。返回 VIABLE（风险可控）、CONCERNS [带缓解建议的列表] 或 HIGH RISK [需要概念或范围修订的阻塞项]。"

**裁决**：VIABLE / CONCERNS / HIGH RISK

---

### TD-ARCHITECTURE — 架构签字

**触发器**：主架构文档起草后（`/create-architecture` 第 7 阶段），
以及任何主要架构修订后

**传递的上下文**：
- 架构文档路径（`docs/architecture/architecture.md`）
- 技术需求基线（TR-ID 和数量）
- ADR 列表及状态
- 引擎知识差距清单

**提示**：
> "审查此主架构文档的技术合理性。检查：(1) 基线中的每个技术需求是否被架构决策覆盖？(2) 所有 HIGH 风险引擎域是否明确解决或标记为开放问题？(3) API 边界是否清晰、最小且可实现？(4) Foundation 层 ADR 差距是否在实现开始前解决？返回 APPROVE、CONCERNS [列表] 或 REJECT [编码开始前必须解决的阻塞项]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### TD-ADR — 架构决策审查

**触发器**：单个 ADR 创作后（`/architecture-decision`），在标记为 Accepted 前

**传递的上下文**：
- ADR 文件路径
- 引擎版本和领域知识差距风险级别
- 相关 ADR（如有）

**提示**：
> "审查此架构决策记录。它是否有清晰的问题陈述和理由？被拒绝的替代方案是否被认真考虑？后果部分是否诚实承认权衡？是否盖了引擎版本戳？是否标记了后期截止 API 风险？它是否链接到它覆盖的 GDD 需求？返回 APPROVE、CONCERNS [具体差距] 或 REJECT [决策描述不足或做出不合理技术假设]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### TD-ENGINE-RISK — 引擎版本风险审查

**触发器**：在做触及后期截止引擎 API 的架构决策时，
或在确定任何引擎特定实现方法前

**传递的上下文**：
- 正在使用的特定 API 或功能
- 引擎版本和 LLM 知识截止日期（来自 `docs/engine-reference/[engine]/VERSION.md`）
- 来自 breaking-changes 或 deprecated-apis 文档的相关摘要

**提示**：
> "根据版本参考审查此引擎 API 使用。该 API 是否存在于 [引擎版本]？自 LLM 知识截止日期以来其签名、行为或命名空间是否已更改？是否存在已知的弃用或后期截止替代方案？返回 APPROVE（按描述使用安全）、CONCERNS [实现前验证] 或 REJECT [API 已更改 — 提供更正方法]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### TD-PHASE-GATE — 阶段转换时的技术就绪

**触发器**：始终在 `/gate-check` — 与 CD-PHASE-GATE 和 PR-PHASE-GATE 并行派生

**传递的上下文**：
- 目标阶段名称
- 架构文档路径（如存在）
- 引擎参考路径
- ADR 列表

**提示**：
> "从技术方向角度审查当前项目状态是否准备好进入 [目标阶段] 门控。架构对于此阶段是否健全？所有高风险引擎域是否已解决？性能预算是否现实且有文档记录？Foundation 层决策是否足够完整以开始实现？返回 READY、CONCERNS [列表] 或 NOT READY [阻塞项]。"

**裁决**：READY / CONCERNS / NOT READY

---

## 第一层 — 制作人门控

Agent: `producer` | 模型层级: Opus | 领域: 范围、时间线、依赖、生产风险

---

### PR-SCOPE — 范围和时间线验证

**触发器**：范围层级定义后（头脑风暴第 6 阶段、quick-design 或任何产生 MVP 定义和时间线估计的工作流程）

**传递的上下文**：
- 完整愿景范围描述
- MVP 定义
- 时间线估计
- 团队规模（solo / 小队 / 等）
- 范围层级（如果时间用完则发货什么）

**提示**：
> "审查此范围估计。MVP 在所述团队规模的所述时间线内是否可实现？范围层级是否按风险正确排序 — 每个层级如果工作在那里停止是否提供可发布产品？在时间压力下最可能的 cut 点是什么，它是一个 graceful fallback 还是 broken product？返回 REALISTIC（范围匹配容量）、OPTIMISTIC [具体调整建议] 或 UNREALISTIC [阻塞项 — 时间线或 MVP 必须修订]。"

**裁决**：REALISTIC / OPTIMISTIC / UNREALISTIC

---

### PR-SPRINT — Sprint 可行性审查

**触发器**：在最终确定 Sprint 计划前（`/sprint-plan`），以及任何中期 Sprint 范围更改后

**传递的上下文**：
- 拟议的 Sprint story 列表（标题、估计、依赖）
- 团队容量（可用小时数）
- 当前 Sprint 积压债务（如有）
- 里程碑约束

**提示**：
> "审查此 Sprint 计划的可行性。对于可用容量，story 负载是否现实？story 是否按依赖正确排序？是否存在可能中途阻塞 Sprint 的 story 之间的隐藏依赖？考虑到其技术复杂性，是否有任何 story 被低估？返回 REALISTIC（计划可实现）、CONCERNS [具体风险] 或 UNREALISTIC [Sprint 必须降级 — 确定哪些 story 应推迟]。"

**裁决**：REALISTIC / CONCERNS / UNREALISTIC

---

### PR-MILESTONE — 里程碑风险评估

**触发器**：在里程碑审查（`/milestone-review`）、中期 Sprint 回顾，
或当提议影响里程碑的范围更改时

**传递的上下文**：
- 里程碑定义和目标日期
- 当前完成百分比
- 阻塞 story 计数
- Sprint 速度数据（如有）

**提示**：
> "审查此里程碑状态。基于当前速度和阻塞 story 计数，此里程碑是否会达到目标日期？从现在到里程碑之间 top 3 生产风险是什么？是否有应该削减的范围项目以保护里程碑日期 vs 哪些是不可协商的？返回 ON TRACK、AT RISK [具体缓解] 或 OFF TRACK [日期必须推迟或范围必须削减 — 提供两个选项]。"

**裁决**：ON TRACK / AT RISK / OFF TRACK

---

### PR-EPIC — Epic 结构可行性审查

**触发器**：`/create-epics` 定义 epics 后，story 分解前 —
在 `/create-stories` 调用前验证 epic 结构可生产

**传递的上下文**：
- Epic 定义文件路径（刚创建的所有 epic）
- Epic 索引路径（`production/epics/index.md`）
- 里程碑时间线和目标日期
- 团队容量（solo / 小队 / 规模）
- 正在 epic 的层级（Foundation / Core / Feature / 等）

**提示**：
> "在 story 分解开始前审查此 epic 结构的生产可行性。Epic 边界是否适当 scoped — 每个 epic 真的能在里程碑截止日期前完成？Epic 是否按系统依赖正确排序 — 是否有任何 epic 需要另一个 epic 的输出才能开始？是否有任何 epic  scoping 不足（太小，应合并）或过度（太大，应拆分为 2-3 个 focused epic）？Foundation 层 epic 是否 scoped 以允许 Core 层 epic 在 Foundation 完成后下一个 Sprint 开始？返回 REALISTIC（epic 结构可生产）、CONCERNS [story 编写前的具体结构调整] 或 UNREALISTIC [epic 必须拆分、合并或重新排序 — 在解决前 story 分解无法开始]。"

**裁决**：REALISTIC / CONCERNS / UNREALISTIC

---

### PR-PHASE-GATE — 阶段转换时的生产就绪

**触发器**：始终在 `/gate-check` — 与 CD-PHASE-GATE 和 TD-PHASE-GATE 并行派生

**传递的上下文**：
- 目标阶段名称
- Sprint 和里程碑产物存在
- 团队规模和容量
- 当前阻塞 story 计数

**提示**：
> "从生产角度审查当前项目状态是否准备好进入 [目标阶段] 门控。范围对于所述时间线和团队规模是否现实？依赖是否正确排序以便团队实际可以顺序执行？是否存在可能在前两个 Sprint 内导致阶段脱轨的里程碑或 Sprint 风险？返回 READY、CONCERNS [列表] 或 NOT READY [阻塞项]。"

**裁决**：READY / CONCERNS / NOT READY

---

## 第一层 — 美术总监门控

Agent: `art-director` | 模型层级: Sonnet | 领域: 视觉身份、美术圣经、视觉生产就绪

---

### AD-CONCEPT-VISUAL — 视觉身份锚点

**触发器**：游戏支柱锁定后（头脑风暴第 4 阶段），与 CD-PILLARS 并行

**传递的上下文**：
- 游戏概念（电梯演讲、核心幻想、独特钩子）
- 完整支柱集（名称、定义、设计测试）
- 目标平台（如已知）
- 用户提到的任何参考游戏或视觉标杆

**提示**：
> "基于这些游戏支柱和核心概念，提出 2-3 个 distinct 视觉身份方向。对于每个方向提供：(1) 一行视觉规则可以指导所有视觉决策（例如'一切必须移动'、'美在衰败中'），(2) 氛围和氛围目标，(3) 形状语言（锐利/圆润/有机/几何强调），(4) 色彩哲学（调色板方向、在这个世界上颜色的含义）。要具体 — 避免通用描述。一个方向应直接为主要设计支柱服务。为每个方向命名。推荐哪个最能服务于陈述的支柱并解释原因。"

**裁决**：CONCEPTS（多个有效选项 — 用户选择）/ STRONG（一个方向明显 dominant）/ CONCERNS（支柱尚未提供足够方向来区分视觉身份）

---

### AD-ART-BIBLE — 美术圣经签字

**触发器**：美术圣经起草后（`/art-bible`），资源生产开始前

**传递的上下文**：
- 美术圣经路径（`design/art/art-bible.md`）
- 游戏支柱和核心幻想
- 平台和性能约束（如已配置，来自 `.claude/docs/technical-preferences.md`）
- 头脑风暴期间选择的视觉身份锚点（来自 `design/gdd/game-concept.md`）

**提示**：
> "审查此美术圣经的完整性和内部一致性。色彩系统是否匹配氛围目标？形状语言是否遵循视觉身份声明？资源标准是否在平台约束内可实现？角色设计方向是否为艺术家提供了足够的工作基础而没有 over-specifying？章节之间是否存在矛盾？外包团队能否无需额外 briefing 就能从此文档生产资源？返回 APPROVE（美术圣经已生产就绪）、CONCERNS [需要澄清的具体章节] 或 REJECT [必须在资源生产开始前解决的根本性不一致]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### AD-PHASE-GATE — 阶段转换时的视觉就绪

**触发器**：始终在 `/gate-check` — 与 CD-PHASE-GATE、TD-PHASE-GATE 和 PR-PHASE-GATE 并行派生

**传递的上下文**：
- 目标阶段名称
- 存在的所有美术/视觉产物列表（文件路径）
- 来自 `design/gdd/game-concept.md` 的视觉身份锚点（如存在）
- 美术圣经路径（如存在 `design/art/art-bible.md`）

**提示**：
> "从视觉方向角度审查当前项目状态是否准备好进入 [目标阶段] 门控。视觉身份是否在此阶段要求的水平上建立并有文档记录？正确的视觉产物是否就位？视觉团队能否开始工作而不会有导致以后昂贵返工的视觉方向差距？是否存在被 defer 超过其最新 responsible moment 的视觉决策？返回 READY、CONCERNS [可能导致生产返工的具体视觉方向差距] 或 NOT READY [此阶段成功前必须存在的视觉阻塞项 — 指定缺少什么产物以及为什么在这个阶段重要]。"

**裁决**：READY / CONCERNS / NOT READY

---

## 第二层 — 主管门控

这些门控由编排技能和高级技能调用，当需要领域专家可行性签字时。
第二层主管使用 Sonnet（默认）。

---

### LP-FEASIBILITY — 主管程序员实现可行性

**触发器**：主架构文档写完后（`/create-architecture` 第 7b 阶段），
或当提出新的架构模式时

**传递的上下文**：
- 架构文档路径
- 技术需求基线摘要
- ADR 列表及状态

**提示**：
> "审查此架构的实现可行性。标记：(a) 任何使用陈述的引擎和语言难以或不可能实现的决策，(b) 程序员需要自己发明的任何缺失接口定义，(c) 任何创建可避免技术债务或与标准 [引擎] 习语矛盾的模式。返回 FEASIBLE、CONCERNS [列表] 或 INFEASIBLE [使此架构按 Written 无法实现的阻塞项]。"

**裁决**：FEASIBLE / CONCERNS / INFEASIBLE

---

### LP-CODE-REVIEW — 主管程序员代码审查

**触发器**：dev story 实现后（`/dev-story`、`/story-done`），
或作为 `/code-review` 的一部分

**传递的上下文**：
- 实现文件路径
- Story 文件路径（用于验收标准）
- 相关 GDD 章节
- 治理此系统的 ADR

**提示**：
> "根据 story 验收标准和治理 ADR 审查此实现。代码是否匹配架构边界定义？是否存在编码标准或禁止模式的违规？公共 API 是否可测试且有文档记录？相对于 GDD 规则是否存在任何正确性问题？返回 APPROVE、CONCERNS [具体问题] 或 REJECT [合并前必须修订]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### QL-STORY-READY — QA 主管 Story 就绪检查

**触发器**：在 story 被接受进入 Sprint 前 — 由 `/create-stories`、
`/story-readiness` 和 `/sprint-plan` 在 story 选择期间调用

**传递的上下文**：
- Story 文件路径
- Story 类型（Logic / Integration / Visual/Feel / UI / Config/Data）
- 验收标准列表（逐字从 story 复制）
- Story 覆盖的 GDD 需求（TR-ID 和文本）

**提示**：
> "在 story 进入 Sprint 前审查此 story 的验收标准可测试性。所有标准是否足够具体以使开发者明确知道何时完成？对于 Logic 类型 story：每个标准是否可以用自动化测试验证？对于 Integration story：每个标准是否可在受控测试环境中观察？标记对于实现来说太模糊的标准，并标记需要完整游戏构建才能测试的标准（标记为 DEFERRED，不是 BLOCKED）。返回 ADEQUATE（标准按 Written 可实现）、GAPS [需要完善的具体标准] 或 INADEQUATE [标准太模糊 — story 必须修订才能包含在 Sprint 中]。"

**裁决**：ADEQUATE / GAPS / INADEQUATE

---

### QL-TEST-COVERAGE — QA 主管测试覆盖率审查

**触发器**：实现 story 完成后，在标记 epic done 前，
或在 `/gate-check` Production → Polish

**传递的上下文**：
- 已实现 story 列表及 story 类型（Logic / Integration / Visual / UI / Config）
- `tests/` 中的测试文件路径
- 系统 GDD 验收标准

**提示**：
> "审查这些实现 story 的测试覆盖率。所有 Logic story 是否被通过的单元测试覆盖？Integration story 是否被集成测试或记录的试玩覆盖？GDD 验收标准是否每个都映射到至少一个测试？GDD 边缘情况章节是否有未测试的？返回 ADEQUATE（覆盖率符合标准）、GAPS [具体缺失测试] 或 INADEQUATE [关键逻辑未测试 — 不要推进]。"

**裁决**：ADEQUATE / GAPS / INADEQUATE

---

### ND-CONSISTENCY — 叙事总监一致性检查

**触发器**：writer 交付物（对话、传说、物品描述）创作后，
或当设计决策有叙事影响时

**传递的上下文**：
- 文档或内容文件路径
- 叙事圣经或基调指南路径（如存在）
- 相关的世界构建规则
- 受影响的角色或派系档案

**提示**：
> "审查此叙事内容的内部一致性和对已建立世界规则的遵循。角色 voice 是否与其已建立档案一致？传说是否与任何已建立的事实矛盾？基调是否与游戏的叙事方向一致？返回 APPROVE、CONCERNS [具体要修复的不一致] 或 REJECT [破坏叙事基础的根本矛盾]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

### AD-VISUAL — 美术总监视觉一致性审查

**触发器**：美术方向决策做出后，当引入新资源类型时，
或当影响视觉风格的技术美术决策时

**传递的上下文**：
- 美术圣经路径（如存在于 `design/art-bible.md`）
- 正在审查的特定资源类型、风格决策或视觉方向
- 参考图像或风格描述
- 平台和性能约束

**提示**：
> "审查此视觉方向决策与已建立艺术风格和生产约束的一致性。它是否匹配美术圣经？在平台的性能预算内是否可实现？是否存在造成技术风险的资源管线影响？返回 APPROVE、CONCERNS [具体调整] 或 REJECT [必须先解决的风格违规或生产风险]。"

**裁决**：APPROVE / CONCERNS / REJECT

---

## 并行门控协议

当工作流程需要在同一检查点派生多个导演时（最常见于 `/gate-check`），
同时派生所有代理：

```
并行派生（等待任何结果前发出所有 Task 调用）：
1. creative-director  → gate CD-PHASE-GATE
2. technical-director → gate TD-PHASE-GATE
3. producer           → gate PR-PHASE-GATE
4. art-director       → gate AD-PHASE-GATE

收集所有四个裁决，然后应用升级规则：
- 任何 NOT READY / REJECT → 总体裁决最低 FAIL
- 任何 CONCERNS → 总体裁决最低 CONCERNS
- 所有 READY / APPROVE → 有资格 PASS（仍取决于产物检查）
```

---

## 添加新门控

当新技能或工作流程需要新门控时：

1. 分配门控 ID：`[DIRECTOR-PREFIX]-[DESCRIPTIVE-SLUG]`
   - 前缀：`CD-` `TD-` `PR-` `LP-` `QL-` `ND-` `AD-`
   - 为新 Agent 添加新前缀：`AudioDirector` → `AU-`、`UX` → `UX-`
2. 在适当的导演部分下添加门控，包含所有五个字段：
   Trigger、Context to pass、Prompt、Verdicts 和任何特殊处理说明
3. 仅通过 ID 在技能中引用 — 永远不要将提示文本复制到技能中

---

## 按阶段划分的门控覆盖

| 阶段 | 必需门控 | 可选门控 |
|-------|---------------|----------------|
| **Concept** | CD-PILLARS、AD-CONCEPT-VISUAL | TD-FEASIBILITY、PR-SCOPE |
| **Systems Design** | TD-SYSTEM-BOUNDARY、CD-SYSTEMS、PR-SCOPE、CD-GDD-ALIGN（每 GDD） | ND-CONSISTENCY、AD-VISUAL |
| **Technical Setup** | TD-ARCHITECTURE、TD-ADR（每 ADR）、LP-FEASIBILITY、AD-ART-BIBLE | TD-ENGINE-RISK |
| **Pre-Production** | PR-EPIC、QL-STORY-READY（每 story）、PR-SPRINT、所有四个 PHASE-GATE（通过 gate-check） | CD-PLAYTEST |
| **Production** | LP-CODE-REVIEW（每 story）、QL-STORY-READY、PR-SPRINT（每 sprint） | PR-MILESTONE、QL-TEST-COVERAGE、AD-VISUAL |
| **Polish** | QL-TEST-COVERAGE、CD-PLAYTEST、PR-MILESTONE | AD-VISUAL |
| **Release** | 所有四个 PHASE-GATE（通过 gate-check） | QL-TEST-COVERAGE |