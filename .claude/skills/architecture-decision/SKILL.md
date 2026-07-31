---
name: architecture-decision
description: "Creates an Architecture Decision Record (ADR) documenting a significant technical decision, its context, alternatives considered, and consequences. Every major technical choice should have an ADR."
argument-hint: "[title] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Task, AskUserQuestion
model: sonnet
---

当此 skill 被调用时：

## 0. 解析参数 —— 检测 Retrofit 模式

解析审查模式（一次，存储供本次运行的所有门生成使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

完整检查模式见 `.claude/docs/director-gates.md`。

**如果参数以 `retrofit` 开头后跟文件路径**
（例如 `/architecture-decision retrofit docs/architecture/adr-0001-event-system.md`）：

进入 **retrofit 模式**：

1. 完整读取现有 ADR 文件。
2. 通过扫描标题识别存在哪些模板章节：
   - `## Status` —— **如果缺失则为 BLOCKING**：`/story-readiness` 无法检查 ADR 接受状态
   - `## ADR Dependencies` —— 如果缺失则为 HIGH：依赖排序中断
   - `## Engine Compatibility` —— 如果缺失则为 HIGH：post-cutoff 风险未知
   - `## GDD Requirements Addressed` —— 如果缺失则为 MEDIUM：可追溯性丢失
3. 向用户展示：
   ```
   ## Retrofit: [ADR title]
   File: [path]

   Sections already present (will not be touched):
   ✓ Status: [current value, or "MISSING — will add"]
   ✓ [section]

   Missing sections to add:
   ✗ Status —— BLOCKING (stories cannot validate ADR acceptance without this)
   ✗ ADR Dependencies —— HIGH
   ✗ Engine Compatibility —— HIGH
   ```
4. 询问："我可以添加 [N] 个缺失的章节吗？我不会修改任何现有内容。"
5. 如果同意：
   - 对于 **Status**：询问用户 —— "此决策的当前状态是什么？"
     选项："Proposed"、"Accepted"、"Deprecated"、"Superseded by ADR-XXXX"
   - 对于 **ADR Dependencies**：询问 —— "此决策是否依赖于任何其他 ADR？
     它是否启用或阻止任何其他 ADR 或 epic？" 每个字段接受 "None"。
   - 对于 **Engine Compatibility**：读取引擎参考文档（与下面的步骤 1 相同）
     并要求用户确认域。然后使用验证过的数据生成表格。
   - 对于 **GDD Requirements Addressed**：询问 —— "哪些 GDD 系统促成了此决策？
     此 ADR 解决了每个 GDD 中的什么具体需求？"
   - 使用 Edit 工具将每个缺失的章节追加到 ADR 文件。
   - **永远不要修改任何现有章节。** 仅追加或填充缺失的章节。
6. 添加所有缺失章节后，如果 ADR 的 `## Date` 字段缺失，则更新它。
7. 建议："运行 `/architecture-review` 重新验证覆盖范围，现在此 ADR 有其 Status 和 Dependencies 字段。"

如果不是 retrofit 模式，继续下面的步骤 1（正常 ADR 编写）。

**无参数保护**：如果未提供参数（标题为空），在运行阶段 0 之前询问：

> "你正在记录什么技术决策？请提供一个简短的标题
> （例如 `event-system-architecture`、`physics-engine-choice`）。"

使用用户的回答作为标题，然后继续步骤 1。

---

## 1. 加载引擎上下文（始终首先）

在做任何其他事情之前，建立引擎环境：

1. 读取 `docs/engine-reference/[engine]/VERSION.md` 获取：
   - 引擎名称和版本
   - LLM 知识截止日期
   - Post-cutoff 版本风险级别（LOW / MEDIUM / HIGH）

2. 从标题或用户描述中识别此架构决策的**域**。
   常见域：Physics、Rendering、UI、Audio、Navigation、
   Animation、Networking、Core、Input、Scripting。

3. 如果存在，读取相应的模块参考：
   `docs/engine-reference/[engine]/modules/[domain].md`

4. 读取 `docs/engine-reference/[engine]/breaking-changes.md` —— 标记域中
   在 LLM 训练截止日期之后的任何更改。

5. 读取 `docs/engine-reference/[engine]/deprecated-apis.md` —— 标记域中
   不应使用的任何 API。

6. 如果域具有 MEDIUM 或 HIGH 风险，在继续之前**显示知识缺口警告**：

   ```
   ⚠️  ENGINE KNOWLEDGE GAP WARNING
   Engine: [name + version]
   Domain: [domain]
   Risk Level: HIGH — This version is post-LLM-cutoff.

   Key changes verified from engine-reference docs:
   - [Change 1 relevant to this domain]
   - [Change 2]

   This ADR will be cross-referenced against the engine reference library.
   Proceed with verified information only — do NOT rely solely on training data.
   ```

   如果尚未配置引擎，提示："未配置引擎。
   先运行 `/setup-engine`，或告诉我你正在使用哪个引擎。"

---

## 2. 确定下一个 ADR 编号

扫描 `docs/architecture/` 中的现有 ADR 以找到下一个编号。

---

## 3. 收集上下文

从 `design/gdd/` 读取相关代码、现有 ADR 和相关 GDD。

### 3a：架构注册表检查（BLOCKING 门）

读取 `docs/registry/architecture.yaml`。提取与此 ADR 的域和决策相关的条目（按系统名称、域关键字或正在触及的状态进行 grep）。

在协作设计开始之前，将任何相关立场展示给用户，作为锁定约束：

```
## Existing Architectural Stances (must not contradict)

State Ownership:
  player_health → owned by health-system (ADR-0001)
  Interface: HealthComponent.current_health (read-only float)
  → If this ADR reads or writes player health, it must use this interface.

Interface Contracts:
  damage_delivery → signal pattern (ADR-0003)
  Signal: damage_dealt(amount, target, is_crit)
  → If this ADR delivers or receives damage events, it must use this signal.

Forbidden Patterns:
  ✗ autoload_singleton_coupling (ADR-0001)
  ✗ direct_cross_system_state_write (ADR-0000)
  → The proposed approach must not use these patterns.
```

如果用户提出的决策会与任何已注册的立场冲突，立即暴露冲突：

> "⚠️ 冲突：此 ADR 提议 [X]，但 ADR-[NNNN] 已确立 [Y] 是此目的的接受模式。
> 在未解决此冲突的情况下继续将产生矛盾的 ADR 和不一致的 story。
> 选项：(1) 与现有立场对齐，(2) 用显式替换取代 ADR-[NNNN]，(3) 解释为什么此情况是例外。"

在冲突解决或明确接受为有意例外之前，不要继续步骤 4（协作设计）。

---

## 4. 协作引导决策

在询问任何内容之前，从已收集的上下文（已读取的 GDD、已加载的引擎参考、已扫描的现有 ADR）中推导 skill 的最佳猜测。然后使用 `AskUserQuestion` 展示一个**确认/调整**提示 —— 不是开放式问题。

**首先推导假设：**
- **问题**：从标题 + GDD 上下文推断需要做出什么决策
- **替代方案**：从引擎参考 + GDD 需求提出 2-3 个具体选项
- **依赖关系**：扫描现有 ADR 以获取上游依赖；如果不清楚则假设为 None
- **GDD 关联**：提取标题直接关联的 GDD 系统
- **状态**：对于新 ADR 始终为 `Proposed` —— 永远不要询问用户状态是什么

**假设范围选项卡**：假设仅涵盖：问题框架、替代方法、上游依赖、GDD 关联和状态。模式设计问题（例如，"生成计时应该如何工作？"、"数据应该是内联还是外部的？"）不是假设 —— 它们是属于假设确认后的单独步骤的设计决策。不要在假设 AskUserQuestion 小部件中包含模式设计问题。

**假设确认后**，如果 ADR 涉及模式或数据设计选择，在起草之前使用单独的多选项卡 `AskUserQuestion` 独立询问每个设计问题。

**使用 `AskUserQuestion` 展示假设：**

```
这是我起草前的假设：

问题：[从上下文推导的一句话问题陈述]
我将考虑的替代方案：
  A) [从引擎参考推导的选项]
  B) [从 GDD 需求推导的选项]
  C) [来自常见模式的选项]
驱动此决策的 GDD 系统：[从上下文推导的列表]
依赖关系：[上游 ADR（如果有），否则为 "None"]
状态：Proposed

[A] 继续 —— 用这些假设起草
[B] 更改替代方案列表
[C] 调整 GDD 关联
[D] 添加性能预算约束
[E] 首先需要更改其他内容
```

在用户确认假设或提供修正之前，不要生成 ADR。

**引擎专家和 TD 审查返回后**（步骤 5.5/5.6），如果仍存在未解决的决策，将每个决策作为单独的 `AskUserQuestion` 展示，将提议的选项作为选择加上一个自由文本转义：

```
决策：[具体的未解决点]
[A] [来自专家审查的选项]
[B] [替代选项]
[C] 不同方法 —— 我会描述它
```

**ADR 依赖关系** —— 从现有 ADR 推导，然后确认：
- 此决策是否依赖于任何尚未 Accepted 的其他 ADR？
- 它是否解锁或解除了任何其他 ADR 或 epic 的阻塞？
- 它是否阻止任何特定的 epic 开始？

将答案记录在 **ADR 依赖关系** 部分。如果没有约束适用，每个字段写 "None"。

---

## 5. 生成 ADR

遵循此格式：

```markdown
# ADR-[NNNN]: [Title]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-XXXX]

## Date
[Date of decision]

## Engine Compatibility

| Field | Value |
|-------|-------|
| **Engine** | [e.g. Godot 4.6] |
| **Domain** | [Physics / Rendering / UI / Audio / Navigation / Animation / Networking / Core / Input] |
| **Knowledge Risk** | [LOW / MEDIUM / HIGH — from VERSION.md] |
| **References Consulted** | [List engine-reference docs read, e.g. `docs/engine-reference/godot/modules/physics.md`] |
| **Post-Cutoff APIs Used** | [Any APIs from post-LLM-cutoff versions this decision depends on, or "None"] |
| **Verification Required** | [Specific behaviours to test before shipping, or "None"] |

## ADR Dependencies

| Field | Value |
|-------|-------|
| **Depends On** | [ADR-NNNN (must be Accepted before this can be implemented), or "None"] |
| **Enables** | [ADR-NNNN (this ADR unlocks that decision), or "None"] |
| **Blocks** | [Epic/Story name — cannot start until this ADR is Accepted, or "None"] |
| **Ordering Note** | [Any sequencing constraint that isn't captured above] |

## Context

### Problem Statement
[What problem are we solving? Why does this decision need to be made now?]

### Constraints
- [Technical constraints]
- [Timeline constraints]
- [Resource constraints]
- [Compatibility requirements]

### Requirements
- [Must support X]
- [Must perform within Y budget]
- [Must integrate with Z]

## Decision

[The specific technical decision made, described in enough detail for someone
to implement it.]

### Architecture Diagram
[ASCII diagram or description of the system architecture this creates]

### Key Interfaces
[API contracts or interface definitions this decision creates]

## Alternatives Considered

### Alternative 1: [Name]
- **Description**: [How this would work]
- **Pros**: [Advantages]
- **Cons**: [Disadvantages]
- **Rejection Reason**: [Why this was not chosen]

### Alternative 2: [Name]
- **Description**: [How this would work]
- **Pros**: [Advantages]
- **Cons**: [Disadvantages]
- **Rejection Reason**: [Why this was not chosen]

## Consequences

### Positive
- [Good outcomes of this decision]

### Negative
- [Trade-offs and costs accepted]

### Risks
- [Things that could go wrong]
- [Mitigation for each risk]

## GDD Requirements Addressed

| GDD System | Requirement | How This ADR Addresses It |
|------------|-------------|--------------------------|
| [system-name].md | [specific rule, formula, or performance constraint from that GDD] | [how this decision satisfies it] |

## Performance Implications
- **CPU**: [Expected impact]
- **Memory**: [Expected impact]
- **Load Time**: [Expected impact]
- **Network**: [Expected impact, if applicable]

## Migration Plan
[If this changes existing code, how do we get from here to there?]

## Validation Criteria
[How will we know this decision was correct? What metrics or tests?]

## Related Decisions
- [Links to related ADRs]
- [Links to related design documents]
```

5.5. **引擎专家验证** —— 在保存之前，通过 Task 派生**主要引擎专家**来验证起草的 ADR：
   - 读取 `.claude/docs/technical-preferences.md` 的 `Engine Specialists` 部分以获取主要专家
   - 如果未配置引擎（`[TO BE CONFIGURED]`），跳过此步骤
   - 派生 `subagent_type: [primary specialist]`，传入：ADR 的 Engine Compatibility 部分、Decision 部分、Key Interfaces 和引擎参考文档路径。要求他们：
     1. 确认提议的方法对于固定的引擎版本是惯用的
     2. 标记任何在训练截止日期后已弃用或更改的 API 或模式
     3. 识别当前 ADR 草稿中未捕获的引擎特定风险或陷阱
   - 如果专家识别出**阻塞问题**（错误的 API、已弃用的方法、引擎版本不兼容性）：相应地修改 Decision 和 Engine Compatibility 部分，然后在继续之前与用户确认更改
   - 如果专家仅发现**次要说明**：将它们纳入 ADR 的 Risks 子部分

**审查模式检查** —— 在派生 TD-ADR 之前应用：
- `solo` → 跳过。注意："TD-ADR skipped — Solo mode." 继续步骤 5.7（GDD 同步检查）。
- `lean` → 跳过（不是 PHASE-GATE）。注意："TD-ADR skipped — Lean mode." 继续步骤 5.7（GDD 同步检查）。
- `full` → 正常派生。

5.6. **技术总监战略审查** —— 引擎专家验证后，使用门 **TD-ADR** 通过 Task 派生 `technical-director`（`.claude/docs/director-gates.md`）：
   - 传入：ADR 文件路径（或草稿内容）、引擎版本、域、同域中的任何现有 ADR
   - TD 验证架构一致性（此决策是否与整个系统一致？）—— 不同于引擎专家的 API 级别检查
   - 如果 CONCERNS 或 REJECT：在继续之前相应地修改 Decision 或 Alternatives 部分

5.7. **GDD 同步检查** —— 在展示写入批准之前，扫描"GDD Requirements Addressed"部分中引用的所有 GDD，检查与 ADR 的 Key Interfaces 和 Decision 部分的命名不一致（重命名的信号、API 方法或数据类型）。如果发现任何问题，在写入批准之前立即将其作为**突出警告块**展示 —— 而不是作为脚注：

```
⚠️ GDD SYNC REQUIRED
[gdd-filename].md uses names this ADR has renamed:
  [old_name] → [new_name_from_adr]
  [old_name_2] → [new_name_2_from_adr]
The GDD must be updated before or alongside writing this ADR to prevent
developers reading the GDD from implementing the wrong interface.
```

如果没有不一致：静默跳过此块。

5. **写入批准** —— 使用 `AskUserQuestion`：

如果发现 GDD 同步问题：
- "ADR 草稿已完成。你希望如何继续？"
  - [A] 写入 ADR + 在同一遍中更新 GDD
  - [B] 仅写入 ADR —— 我会手动更新 GDD
  - [C] 还不行 —— 我需要进一步审查

如果没有 GDD 同步问题：
- "ADR 草稿已完成。我可以写入吗？"
  - [A] 将 ADR 写入 `docs/architecture/adr-[NNNN]-[slug].md`
  - [B] 还不行 —— 我需要进一步审查

如果同意任何写入选项，写入文件，需要时创建目录。
对于带有 GDD 更新的选项 [A]：还要更新 GDD 文件以使用新名称。

6. **更新架构注册表**

扫描已写入的 ADR 以获取应注册的新架构立场：
- 它声称拥有所有权的状态
- 它定义的接口契约（信号签名、方法 API）
- 它声明的性能预算
- 它明确做出的 API 选择
- 它禁止的模式（Consequences → Negative 或显式的"不要使用 X"）

展示候选：
```
Registry candidates from this ADR:
  NEW state ownership:      player_stamina → stamina-system
  NEW interface contract:   stamina_depleted signal
  NEW performance budget:   stamina-system: 0.5ms/frame
  NEW forbidden pattern:    polling stamina each frame (use signal instead)
  EXISTING (referenced_by update only): player_health → already registered ✅
```

**注册表追加逻辑**：写入 `docs/registry/architecture.yaml` 时，不要假设部分为空。文件可能已有本次会话中先前写入的 ADR 的条目。在每次 Edit 调用之前：
1. 读取 `docs/registry/architecture.yaml` 的当前状态
2. 找到正确的部分（state_ownership、interfaces、forbidden_patterns、api_decisions）
3. 将新条目追加到该部分最后一个现有条目之后 —— 不要尝试替换可能不再存在的 `[]` 占位符
4. 如果部分已有条目，使用最后一个条目的结束内容作为 `old_string` 锚点，并在其后追加新条目

**BLOCKING —— 未经明确用户批准，不要写入 `docs/registry/architecture.yaml`。**

使用 `AskUserQuestion` 询问：
- "我可以用这些 [N] 个新立场更新 `docs/registry/architecture.yaml` 吗？"
  - 选项："是 —— 更新注册表"、"还不行 —— 我想审查候选"、"跳过注册表更新"

仅在用户选择是时继续。如果同意：追加新条目。永远不要修改现有条目 —— 如果立场正在更改，将旧条目设置为 `status: superseded_by: ADR-[NNNN]` 并添加新条目。

---

## 6. 结束下一步

写入 ADR 后（以及可选地更新注册表后），使用 `AskUserQuestion` 结束。

在生成小部件之前：
1. 读取 `docs/registry/architecture.yaml` —— 检查是否仍有未写入的高优先级 ADR（查找在 technical-preferences.md 或 systems-index.md 中标记为前置条件的 ADR）
2. 检查所有前置 ADR 是否现在已写入。如果是，包含"开始编写 GDD"选项。
3. 将所有剩余的高优先级 ADR 列为单独选项 —— 不仅仅是下一个或两个。

小部件格式：
```
ADR-[NNNN] written and registry updated. What would you like to do next?
[1] Write [next-priority-adr-name] — [brief description from prerequisites list]
[2] Write [another-priority-adr] — [brief description]  (include ALL remaining ones)
[N] Start writing GDDs — run `/design-system [first-undesigned-system]` (only show if all prerequisite ADRs are written)
[N+1] Stop here for this session
```

如果没有剩余的高优先级 ADR 和未设计的 GDD 系统，仅提供"Stop here"并建议在新会话中运行 `/architecture-review`。

**始终在结束输出中包含此固定通知（不要省略）：**

> 要验证 ADR 覆盖范围是否符合你的 GDD，请打开一个**全新的 Claude Code 会话**
> 并运行 `/architecture-review`。
>
> **永远不要在与 `/architecture-decision` 相同的会话中运行 `/architecture-review`。**
> 审查 agent 必须独立于编写上下文才能给出无偏见的评估。
> 在此处运行将使审查无效。

将任何状态为 `Status: Blocked` 等待此 ADR 的 story 更新为 `Status: Ready`。
