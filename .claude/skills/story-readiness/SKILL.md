---
name: story-readiness
description: "验证 story 文件是否已准备好实现。检查嵌入的 GDD 需求、ADR 引用、引擎说明、清晰的验收标准和无未解决的设计问题。生成 READY / NEEDS WORK / BLOCKED 裁决及具体缺口。当用户说"这个 story 准备好了吗"、"我可以开始这个 story 吗"、"story X 准备好实现了吗"时使用。"
argument-hint: "[story-file-path or 'all' or 'sprint']"
user-invocable: true
allowed-tools: Read, Glob, Grep, AskUserQuestion, Task
model: sonnet
---

# Story Readiness

此 skill 验证 story 文件是否包含开发者开始实现所需的一切 — 没有中期 sprint 的设计中断，没有猜测，没有模糊的验收标准。在分配 story 之前运行它。

**此 skill 是只读的。** 它绝不编辑 story 文件。它报告发现并询问用户是否希望帮助填补缺口。

**输出：** 每个 story 的裁决（READY / NEEDS WORK / BLOCKED），附带每个未就绪 story 的具体缺口列表。

---

## Phase 0: 解析 Review 模式

在启动时一次性解析 review 模式（存储供本运行的所有 gate 生成使用）：

1. 如果 skill 以 `--review [full|lean|solo]` 调用 → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

参见 `.claude/docs/director-gates.md` 了解完整的检查模式和模式定义。

---

## 1. 解析参数

**范围：** `$ARGUMENTS[0]`（空白 = 通过 AskUserQuestion 询问用户）

- **具体路径**（如 `/story-readiness production/epics/combat/story-001-basic-attack.md`）：
  验证该单个 story 文件。
- **`sprint`**：从 `production/sprints/` 读取当前 sprint 计划（最近
  文件），提取它引用的每个 story 路径，验证每一个。
- **`all`**：glob `production/epics/**/*.md`，排除 `EPIC.md` 索引文件，
  找到的每个 story 文件都验证。
- **无参数**：询问用户要验证哪个范围。

如果没有参数，使用 `AskUserQuestion`：
- "你想验证什么？"
  - 选项："特定的 story file"、"当前 sprint 中的所有 stories"、
    "production/epics/ 中的所有 stories"、"特定 epic 的 stories"

在继续前报告范围："正在验证 [N] 个 story 文件。"

---

## 2. 加载支持上下文

在检查任何 stories 之前，一次性加载参考文档（不是每个 story 都加载）：

- `design/gdd/systems-index.md` — 了解哪些系统有已批准的 GDDs
- `docs/architecture/control-manifest.md` — 了解存在哪些 manifest 规则
  （如果文件不存在，注明一次缺失；不要每个 story 都重新标记）
  如果文件存在，还要从头部块提取 `Manifest Version:` 日期。
- `docs/architecture/tr-registry.yaml` — 按 `id` 索引所有条目。用于
  验证 stories 中的 TR-IDs。如果文件不存在，注明一次；TR-ID
  检查将自动通过所有 stories（登记册早于 stories，所以缺失
  登记册意味着 stories 来自 TR 跟踪引入之前）。
- 所有 ADR 状态字段 — 对于被检查 stories 中引用的每个唯一 ADR，
  读取 ADR 文件并记录其 `Status:` 字段。缓存这些，这样你
  不必为每个 story 重新读取相同的 ADR。
- 当前 sprint 文件（如果范围是 `sprint`）— 识别 Must Have /
  Should Have 优先级以进行升级决策

---

## 3. Story 就绪检查清单

对于每个 story 文件，评估下面的每个项目。只有当所有项目通过或明确标记为 N/A 并说明原因时，story 才为 READY。

### 设计完整性

- [ ] **引用了 GDD 需求**：story 包含 `design/gdd/` 路径
  并引用或链接了该 GDD 中的特定需求、验收标准或规则 —
  不仅仅是 GDD 文件名。仅链接文档而不追溯到特定需求不通过。
- [ ] **需求是自包含的**：story 中的验收标准无需打开 GDD 即可理解。
  开发者不应需要阅读单独的文档来了解 DONE 的含义。
- [ ] **验收标准是可测试的**：每个标准都是具体的、
  可观察的条件 — 不是"实现 X"或"系统正常工作"。
  坏例子："实现跳跃机制。"好例子："跳跃在按住跳跃时
  0.3 秒内达到 5 单位的最大高度。"
- [ ] **验收标准不需要判断调用** *（`Type: Visual/Feel` 自动通过）*：像
  "感觉响应迅速"或"看起来好"这样的标准在没有定义基准的情况下不可测试。
  对于 Logic、Integration、UI 和 Config/Data stories，这些必须替换为
  具体的可观察条件。对于 Visual/Feel stories，主观标准是预期的，
  此检查自动通过 — 相反，验证每个主观标准是否有配对的 playtest 协议或
  证据要求（如"需要在 `production/qa/evidence/[slug]-evidence.md` 的证据文档"）。
  如果验收标准以显式引用文件路径如 `production/qa/evidence/[slug]-evidence.md` 结尾或伴随此类引用，则 PASS。如果标准是纯主观的且未指定证据文件路径，则 NEEDS WORK。

### 架构完整性

- [ ] **引用了 ADR 或说明了 N/A**：story 引用了至少一个 ADR，
  或明确说明"No ADR applies"并附有简短原因。
  没有 ADR 引用也没有明确 N/A 说明的 story 不通过此检查。
- [ ] **ADR 是 Accepted（不是 Proposed）**：对于每个引用的 ADR，使用第 2 节中缓存的 ADR 状态检查其
  `Status:` 字段。
  - 如果 `Status: Accepted` → 通过。
  - 如果 `Status: Proposed` → **BLOCKED**：ADR 在接受前可能更改，
    且 story 的实现指导可能是错误的。
    修复：`BLOCKED: ADR-NNNN 为 Proposed — 等待接受后再实现。`
  - 如果 ADR 文件不存在 → **BLOCKED**：引用的 ADR 缺失。
  - 如果 story 有明确的"No ADR applies" N/A 说明，自动通过。
- [ ] **TR-ID 有效且活跃**：如果 story 包含 `TR-[system]-NNN`
  引用，在第 2 节加载的 TR 登记册中查找。
  - 如果 ID 存在且 `status: active` → 通过。
  - 如果 ID 存在且 `status: deprecated` 或 `status: superseded-by: ...` →
    NEEDS WORK：需求已被移除或替换。
    修复：更新 story 以引用当前需求 ID 或如果不再适用则移除。
  - 如果 ID 在登记册中不存在 → NEEDS WORK：ID 未注册
    （story 可能早于登记册，或登记册需要 `/architecture-review` 运行）。
  - 如果 story 没有 TR-ID 引用或登记册不存在，自动通过。
- [ ] **Manifest 版本是最新的**：如果 story 头部有 `Manifest Version:` 日期
  且 `docs/architecture/control-manifest.md` 存在：
  - 如果 story 版本与当前 manifest `Manifest Version:` 匹配 → 通过。
  - 如果 story 版本比当前 manifest 更旧 → NEEDS WORK：新规则可能
    适用。修复：审查更改的 manifest 规则，如果任何禁止/必需
    条目更改则更新 story，然后将 story 的 `Manifest Version:` 更新为当前。
  - 如果 story 没有 `Manifest Version:` 字段或 manifest
    不存在，自动通过。
- [ ] **存在 Engine notes**：对于此 story 可能触及的任何引擎 API 知识截止后的
  API，包含了实现说明或验证要求。如果 story 明显不触及引擎 API（如
  它是纯数据/配置更改），"N/A — 不涉及引擎 API" 是可接受的。
- [ ] **记录了 Control manifest 规则**：引用了 control manifest 中的相关层规则，
  或说明了"N/A — manifest 尚未创建"。
  如果 `docs/architecture/control-manifest.md` 尚不存在，此项目自动通过
  （不要惩罚在 manifest 创建之前编写的 stories）。

### 范围清晰度

- [ ] **存在估算**：story 包含规模估算（小时、
  points 或 t-shirt size）。没有估算的 story 无法被计划。
- [ ] **说明了范围内/范围外边界**：story 说明了它不包含的内容，
  无论是在明确的 Out of Scope 章节中还是以使边界明确的语言。
  没有这个，实现期间范围蔓延的可能性很高。
- [ ] **列出了 Story 依赖关系**：如果此 story 依赖于其他 stories
  先完成，则列出那些 story ID。如果没有依赖关系，
  明确说明"None"（不仅仅是省略）。

### 开放问题

- [ ] **没有未解决的设计问题**：story 不包含标记为"UNRESOLVED"、
  "TBD"、"TODO"、"?"或等价标记的文本，在任何
  验收标准、实现说明或规则声明中。
- [ ] **依赖 stories 不在 DRAFT 中**：对于列为依赖关系的每个 story，
  检查文件是否存在且没有 DRAFT 状态。依赖于 DRAFT 或缺失 story 的 story
  是 BLOCKED，不仅仅是 NEEDS WORK。

### 资产引用检查

- [ ] **引用的资产存在**：扫描 story 文本中的资产路径模式
  （包含 `assets/` 的路径，或文件扩展名 `.png`、`.jpg`、`.svg`、
  `.wav`、`.ogg`、`.mp3`、`.glb`、`.gltf`、`.tres`、`.tscn`、`.res`）。
  - 对于找到的每个资产路径：使用 Glob 检查文件是否存在。
  - 如果任何引用的资产不存在：**NEEDS WORK** — 注明缺失的
    路径（s）。（story 引用了尚未创建的资产。
    要么移除引用，创建占位符，或将其标记为对资产创建 story 的
    明确依赖。）
  - 如果所有引用的资产都存在：注明"已验证引用的资产：
    找到 [count] 个。"
  - 如果 story 中未引用资产路径：注明"story 中未找到资产引用 — 跳过资产检查。" 此项目自动通过。
  - 这是仅存在性检查。不要验证文件格式或内容。

### 完成定义

- [ ] **按 story 类型的最小可测试验收标准**：
  - Logic / Integration stories：至少 3 个
  - Visual/Feel 和 UI stories：至少 2 个
  - Config/Data stories：至少 1 个
  应用与 story 的 `Type:` 字段匹配的阈值。如果 story 少于最小值，标记为 NEEDS WORK。
- [ ] **如适用，记录了性能预算**：如果此 story 触及游戏循环、
  渲染或物理的任何部分，存在性能预算或
  "预计无性能影响 — [原因]" 说明。
- [ ] **声明了 Story Type**：story 头部包含 `Type:` 字段，
  标识测试类别（Logic / Integration / Visual/Feel / UI / Config/Data）。
  没有这个，无法在 story 关闭时执行测试证据要求。
  修复：在 story 头部添加 `Type: [Logic|Integration|Visual/Feel|UI|Config/Data]`。
- [ ] **测试证据要求清晰**：如果设置了 Story Type，story
  包含 `## Test Evidence` 章节，说明证据将存储在哪里
  （Logic/Integration 的测试文件路径，或 Visual/Feel/UI 的证据文档路径）。
  修复：添加 `## Test Evidence`，包含 story 类型的预期证据位置。

---

## 4. 裁决分配

为每个 story 分配三个裁决之一：

**READY** — 所有检查清单项目通过或有明确的 N/A 理由。
story 可以立即分配。

**NEEDS WORK** — 一个或多个检查清单项目失败，但所有依赖 stories
存在且不是 DRAFT。story 可以在分配前修复。

**BLOCKED** — 一个或多个依赖 stories 缺失或在 DRAFT 状态，
或关键设计问题（在标准或规则中标记为 UNRESOLVED）没有负责人。
story 在阻塞解决之前无法分配。注意：
被 BLOCKED 的 story 也可能有 NEEDS WORK 项目 — 两者都列出。

---

## 5. 输出格式

### 单个 story 输出

```
## Story Readiness: [story title]
File: [path]
Verdict: [READY / NEEDS WORK / BLOCKED]

### Passing Checks (N/[total])
[list passing items briefly]

### Gaps
- [Checklist item]: [exact description of what is missing or wrong]
  Fix: [specific text needed to resolve this gap]

### Blockers (if BLOCKED)
- [What is blocking]: [story ID or design question that must resolve first]
```

### 多个 story 汇总输出

```
## Story Readiness Summary — [scope] — [date]

Ready:      [N] stories
Needs Work: [N] stories
Blocked:    [N] stories

### Ready Stories
- [story title] ([path])

### Needs Work
- [story title]: [primary gap — one line]
- [story title]: [primary gap — one line]

### Blocked Stories
- [story title]: Blocked by [story ID / design question]

---
[每个未就绪 story 的完整详情随后，使用单个 story 格式]
```

### Sprint 升级

如果范围是 `sprint` 且任何 Must Have stories 为 NEEDS WORK 或 BLOCKED，
在输出顶部添加显著警告：

```
警告：[N] 个 Must Have stories 未实现就绪。
[列出它们及其主要缺口或阻塞。]
在 sprint 开始前解决这些，或使用 `/sprint-plan update` 重新计划。
```

---

## 6. 协作协议

此 skill 是只读的。它绝不提议编辑或请求写入文件。

报告发现后，提供：

"你希望我帮助填补这些 stories 中的任何缺口吗？我可以为你起草缺失的章节供审批。"

如果用户对特定 story 说同意，仅在对话中起草缺失的章节。
不要使用 Write 或 Edit 工具 — 用户（或
`/create-stories`）负责写入。

**重定向规则：**
- 如果 story 文件完全不存在："此 story file 完全缺失。
  运行 `/create-epics [layer]` 然后 `/create-stories [epic-slug]` 从 GDD 和 ADR 生成 stories。"
- 如果 story 没有 GDD 引用且工作看起来很小："此 story 没有 GDD 引用。
  如果更改很小（约 4 小时以内），运行 `/quick-design [description]` 创建 Quick Design Spec，
  然后在 story 中引用该规范。"
- 如果 story 的范围已超出其原始规模："此 story 的范围似乎已扩大。
  考虑拆分或在实现开始前升级到 producer。"

---

## 7. 下一个 Story 交接

完成单个 story 就绪检查后（不是 `all` 或 `sprint` 范围）：

1. 从 `production/sprints/` 读取当前 sprint 文件（最近的）。
2. 找到符合以下条件的 stories：
   - Status: READY 或 NOT STARTED
   - 不是刚刚检查的 story
   - 未被不完整依赖关系阻塞
   - 在 Must Have 或 Should Have 层级

如果找到，浮现最多 3 个：

```
### 此 Sprint 中其他就绪的 Stories

1. [Story name] — [一句话描述] — 估算：[X 小时]
2. [Story name] — [一句话描述] — 估算：[X 小时]

运行 `/story-readiness [path]` 在开始前验证。
```

如果不存在 sprint 文件或未找到其他就绪 stories，静默跳过此章节。

---

## Phase 8: Director Gate — Story 就绪审查

在生成 QL-STORY-READY 之前应用 Phase 0 中解析的 review 模式：

- `solo` → 跳过。注明："QL-STORY-READY skipped — Solo mode." 继续到结束。
- `lean` → 跳过。注明："QL-STORY-READY skipped — Lean mode." 继续到结束。
- `full` → 正常生成。

使用 gate **QL-STORY-READY**（`.claude/docs/director-gates.md`）通过 Task 生成 `qa-lead`。

传递以下上下文：
- Story 标题
- 验收标准列表（story 验收标准章节中的所有项目）
- 依赖状态（列出的所有依赖关系及其当前状态：存在 / DRAFT / 缺失）
- Phase 4 的整体裁决（READY / NEEDS WORK / BLOCKED）

按照 `director-gates.md` 中的标准规则处理裁决：
- **ADEQUATE** → story 已清除。继续到结束。
- **GAPS [list]** → 通过 `AskUserQuestion` 向用户展示具体差距：
  选项：`用建议的差距更新 story` / `接受并继续进行` / `进一步讨论`。
- **INADEQUATE** → 展示具体差距；询问用户是更新 story 还是继续进行。

---

## 推荐的后续步骤

- 运行 `/dev-story [story-path]` 在 story 就绪后开始实现
- 运行 `/story-readiness sprint` 一次检查当前 sprint 中的所有 stories
- 如果 story 文件完全缺失，运行 `/create-stories [epic-slug]`
