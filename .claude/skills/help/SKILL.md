---
name: help
description: "Analyzes what is done and the users query and offers advice on what to do next. Use if user says what should I do next or what do I do now or I'm stuck or I don't know what to do"
argument-hint: "[optional: what you just finished, e.g. 'finished design-review' or 'stuck on ADRs']"
user-invocable: true
allowed-tools: Read, Glob, Grep
context: |
  !echo "=== Live Project State ===" && echo "Stage: $(cat production/stage.txt 2>/dev/null | tr -d '[:space:]' || echo 'not set')" && echo "Latest sprint: $(ls -t production/sprints/*.md 2>/dev/null | head -1 || echo 'none')" && echo "Session state: $(head -5 production/session-state/active.md 2>/dev/null || echo 'none')"
model: haiku
---

# Studio Help — 我下一步做什么？

此技能是只读的——它报告发现但不写入文件。

此技能精确找出你在游戏开发管道中的位置，并告诉你接下来是什么。它是**轻量级的**——不是完整审计。对于完整的空白分析，使用 `/project-stage-detect`。

---

## 步骤 1：读取目录

读取 `.claude/docs/workflow-catalog.yaml`。这是所有阶段的权威列表，它们的步骤（按顺序），每个步骤是必需还是可选，以及指示完成的工件 glob。

---

## 步骤 1b：查找目录中未列出的技能

读取目录后，Glob `.claude/skills/*/SKILL.md` 以获取已安装技能的完整列表。对于每个文件，从其 frontmatter 中提取 `name:` 字段。

与目录中的 `command:` 值进行比较。任何名称未作为目录命令出现的技能是**未目录化的技能**——仍可使用但不属于 phase-gated 工作流。

收集这些用于第 7 步的输出——将它们展示为页脚块：

```
### 也已安装（不在工作流中）
- `/skill-name`——[来自 SKILL.md frontmatter 的描述]
- `/skill-name`——[描述]
```

仅当至少存在一个未目录化技能时才显示此块。根据用户当前阶段限制为最相关的 10 个（生产中的 QA 技能、生产/打磨中的团队技能等）。

---

## 步骤 2：确定当前阶段

按以下顺序检查：

1. **读取 `production/stage.txt`**——如果它存在且有内容，这是权威阶段名称。将其映射到目录阶段键：
   - "Concept" → `concept`
   - "Systems Design" → `systems-design`
   - "Technical Setup" → `technical-setup`
   - "Pre-Production" → `pre-production`
   - "Production" → `production`
   - "Polish" → `polish`
   - "Release" → `release`

2. **如果 stage.txt 缺失**，从工件推断阶段（最先进的匹配获胜）：
   - `src/` 有 10+ 个源文件 → `production`
   - `production/stories/*.md` 存在 → `pre-production`
   - `docs/architecture/adr-*.md` 存在 → `technical-setup`
   - `design/gdd/systems-index.md` 存在 → `systems-design`
   - `design/gdd/game-concept.md` 存在 → `concept`
   - 无 → `concept`（新项目）

---

## 步骤 3：读取会话上下文

如果存在，读取 `production/session-state/active.md`。提取：
- 最近处理的内容
- 任何进行中的任务或未解决问题
- STATUS 块中的当前 epic/feature/task（如果存在）

这告诉你用户刚刚完成或卡在哪里——用于个性化输出。

---

## 步骤 4：检查当前阶段的步骤完成

对于当前阶段中的每个步骤（来自目录）：

### 基于工件的检查

如果步骤有 `artifact.glob`：
- 使用 Glob 检查是否存在匹配模式的文件
- 如果指定了 `min_count`，验证至少匹配该数量的文件
- 如果指定了 `artifact.pattern`，使用 Grep 验证匹配文件中是否存在该模式
- **完成** = 工件条件满足
- **不完整** = 工件缺失或未找到模式

如果步骤有 `artifact.note`（无 glob）：
- 标记为 **MANUAL** —— 无法自动检测，将询问用户

如果步骤没有 `artifact` 字段：
- 标记为 **UNKNOWN** —— 完成不可跟踪（例如可重复的实现工作）

### 特殊情况：生产阶段——读取 `sprint-status.yaml`

当当前阶段为 `production` 时，在进行任何基于 glob 的故事检查之前，检查 `production/sprint-status.yaml`。如果存在，直接读取它：

- 状态为 `in-progress` 的 story → 展示为"当前活跃"
- 状态为 `ready-for-dev` 的 story → 展示为"下一个"
- 状态为 `done` 的 story → 计为完成
- 状态为 `blocked` 的 story → 展示为阻塞项及 `blocker` 字段

这提供精确的每个 story 状态，无需 markdown 扫描。跳过 `implement` 和 `story-done` 步骤的 glob 工件检查——YAML 是权威的。

### 特殊情况：`repeatable: true`（非生产）

对于生产以外的可重复步骤（例如"系统 GDD"），工件检查告诉你是否完成了*任何*工作，而不是是否完成。
以不同方式标记这些——展示检测到的内容，然后注明它可能正在进行中。

---

## 步骤 5：找出位置并识别后续步骤

从完成数据中，确定：

1. **最后确认完成的步骤**——最远的已完成必需步骤
2. **当前阻塞项**——第一个不完整的*必需*步骤（这是用户接下来必须做的）
3. **可选机会**——不完整的*可选*步骤，可以在阻塞项之前或同时进行
4. **即将到来的必需步骤**——当前阻塞项之后的必需步骤
   （展示为"即将到来"以便用户可以提前计划）

如果用户提供了参数（例如"刚刚完成 design-review"），即使工件检查不明确，也用它来推进他们命名的步骤。

---

## 步骤 6：检查进行中的工作

如果 `active.md` 显示活动任务或 epic：
- 在顶部突出展示："看起来你在处理 [X]"
- 建议继续或确认是否完成

---

## 步骤 7：展示输出

保持**简短直接**。这是一个快速定位，不是报告。

```
## 你的位置：[阶段标签]

**进行中：** [来自 active.md，如果有]

### ✓ 已完成
- [已完成的步骤名称]
- [已完成的步骤名称]

### → 接下来（必需）
**[步骤名称]** —— [描述]
命令：`[/command]`

### ~ 也可用（可选）
- **[步骤名称]** —— [描述] → `/command`
- **[步骤名称]** —— [描述] → `/command`

### 接下来的
- [下一个必需步骤名称]（`/command`）
- [下一个必需步骤名称]（`/command`）

---
接近 **[下一个阶段]** 门 → 准备好时运行 `/gate-check`。
```

**格式规则：**
- `✓` 表示确认完成
- `→` 表示当前必需的下一步（仅一个——第一个阻塞项）
- `~` 表示现在可用的可选步骤
- 将命令内联展示为反引码代码
- 如果步骤没有命令（例如"实现 Story"），解释要做什么而不是显示斜杠命令
- 对于 MANUAL 步骤，询问用户："我无法判断 [step] 是否完成——它完成了吗？"

裁决：**COMPLETE**——已识别后续步骤。

---

## 步骤 8：门警告（如果接近）

当前阶段的步骤之后，检查用户是否可能接近门：
- 如果当前阶段的所有必需步骤都完成（或接近完成），
  添加："你接近 **[当前] → [下一个]** 门。准备好时运行 `/gate-check`。"
- 如果仍有多个必需步骤，跳过门警告——它还不相关。

---

## 步骤 9：升级路径

建议之后，如果用户似乎卡住或困惑，添加：

```
---
需要更多细节？
- `/project-stage-detect` —— 完整空白分析，列出所有缺失的工件
- `/gate-check` —— 你下一阶段的正式准备情况检查
- `/start` —— 从头重新定位
```

仅当用户的输入暗示困惑时才显示此内容（例如"我不知道"、"卡住"、
"迷失"、"不确定"）。不要为简单的"下一步是什么？"查询显示它。

---

## 协作协议

- **永远不要自动运行下一个技能。** 推荐它，让用户调用它。
- **询问 MANUAL 步骤**而不是假设完成或不完整。
- **匹配用户的语气**——如果他们听起来有压力（"我完全迷失了"），
  要令人放心并给一个行动，而不是六个的列表。
- **一个主要建议**——用户离开时应该确切知道一件事
  接下来做。可选步骤和"即将到来"是次要上下文。
