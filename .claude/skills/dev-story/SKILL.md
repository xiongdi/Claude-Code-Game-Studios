---
name: dev-story
description: "Read a story file and implement it. Loads the full context (story, GDD requirement, ADR guidelines, control manifest), routes to the right programmer agent for the system and engine, implements the code and test, and confirms each acceptance criterion. The core implementation skill — run after /story-readiness, before /code-review and /story-done."
argument-hint: "[story-path]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Bash, Task, AskUserQuestion
model: sonnet
---

# Dev Story

此技能桥接规划和代码。它完整读取 story 文件，组装程序员需要的所有上下文，路由到正确的专家代理，并驱动实现完成——包括编写测试。

**每个 story 的循环：**
```
/qa-plan sprint           ← 在 sprint 开始前定义测试需求
/story-readiness [path]   ← 开始前验证
/dev-story [path]         ← 实现它（此技能）
/code-review [files]      ← 审查它
/story-done [path]        ← 验证并关闭它
```

**所有 sprint story 完成后：** 运行 `/team-qa sprint` 执行完整的 QA 循环，并在推进项目阶段之前获得签字确认裁决。

**输出：** 项目 `src/` 和 `tests/` 目录中的源代码 + 测试文件。

---

## 第 1 阶段：找到 Story

**如果提供了路径**：直接读取该文件。

**如果无参数**：检查 `production/session-state/active.md` 中的活动 story。如果找到，确认："继续 [story title] 的工作——正确吗？"
如果未找到，询问："我们在实现哪个 story？" Glob `production/epics/**/*.md` 并列出状态为 Ready 的 story。

---

## 第 2 阶段：加载完整上下文

**在加载任何上下文之前，验证必需文件存在。** 从 story 的 `ADR Governing Implementation` 字段提取 ADR 路径，然后检查：

| 文件 | 路径 | 如果缺失 |
|------|------|------------|
| TR 注册表 | `docs/architecture/tr-registry.yaml` | **停止** — "在 `docs/architecture/tr-registry.yaml` 未找到 TR 注册表。运行 `/architecture-review` 从你的 GDD 和 ADR 引导注册表。" |
| 管辖 ADR | 来自 story 的 ADR 字段的路径 | **停止** — "未找到 ADR 文件 [path]。运行 `/architecture-decision` 创建它，或更正 story 的 ADR 字段中的文件名。" |
| Control manifest | `docs/architecture/control-manifest.md` | **警告并继续** — "未找到 control manifest——无法检查层级规则。运行 `/create-control-manifest`。" |

如果 TR 注册表或管辖 ADR 缺失，在会话状态中将 story 状态设置为 **BLOCKED**，不要生成任何程序员代理。

同时读取以下所有内容——这些是独立的读取。在所有上下文加载之前不要开始实现：

### story 文件
提取并保留：
- **Story 标题、ID、层级、类型**（Logic / Integration / Visual/Feel / UI / Config/Data）
- **TR-ID**——GDD 需求标识符
- **管辖 ADR** 引用
- **Manifest 版本**——嵌入在 story 头部
- **Acceptance Criteria**——每个复选框项目，逐字
- **Implementation Notes**——story 中的 ADR 指导部分
- **Out of Scope** 边界
- **Test Evidence**——必需的测试文件路径
- **Dependencies**——此 story 之前必须为 DONE 的内容

### TR 注册表
读取 `docs/architecture/tr-registry.yaml`。查找 story 的 TR-ID。
读取当前的 `requirement` 文本——这是 GDD 现在需要什么的事实来源。不要依赖 story 文件中的任何内联文本（可能已过时）。

### 管辖 ADR
读取 `docs/architecture/[adr-file].md`。提取：
- 完整的 Decision 部分
- Implementation Guidelines 部分（这是程序员遵循的内容）
- Engine Compatibility 部分（cutoff 后 API、已知风险）
- ADR Dependencies 部分

### control manifest
读取 `docs/architecture/control-manifest.md`。提取此 story 层级的规则：
- 必需模式
- 禁止模式
- 性能护栏

检查：story 的嵌入 Manifest 版本是否与当前 manifest 头部日期匹配？
如果它们不同，在继续之前使用 `AskUserQuestion`：
- 提示："Story 是针对 manifest v[story-date] 编写的。当前 manifest 是 v[current-date]。可能适用新规则。你想如何继续？"
- 选项：
  - `[A] 更新 story manifest 版本并使用当前规则实现（推荐）`
  - `[B] 使用旧规则实现——我接受不合规的风险`
  - `[C] 停在这里——我想先审查 manifest 差异`

如果 [A]：在生成程序员之前，将 story 文件的 `Manifest Version:` 字段编辑为当前 manifest 日期。然后仔细阅读 manifest 中的新规则。
如果 [B]：将 story 文件的 `Manifest Version:` 字段编辑为当前 manifest 日期，并在 story 头部添加一行 `Manifest-Note: Proceeded with old manifest rules on [date] — non-compliance risk accepted.` 无论如何都要阅读 manifest 中的新规则。在第 6 阶段摘要中的 "Deviations" 下注明该决定。`/story-done` 将在其偏差部分包含 Manifest-Note，而不重新检查过时性。
如果 [C]：停止。不要生成任何代理。让用户审查并重新运行 `/dev-story`。

### 依赖关系验证

从 story 文件提取 **Dependencies** 列表后，验证每个：

1. Glob `production/epics/**/*.md` 以找到每个依赖 story 文件。
2. 读取其 `Status:` 字段。
3. 如果任何依赖的状态不是 `Complete` 或 `Done`：
   - 使用 `AskUserQuestion`：
     - 提示："Story '[current story]' 依赖于 '[dependency title]'，其当前状态为 [status]，不是 Complete。你想如何继续？"
     - 选项：
       - `[A] 无论如何继续——我接受依赖风险`
       - `[B] 停止——我先完成依赖`
       - `[C] 依赖已完成但状态未更新——将其标记为 Complete 并继续`
   - 如果 [B]：在会话状态中将 story 状态设置为 **BLOCKED** 并停止。不要生成任何程序员代理。
   - 如果 [C]：在继续之前询问"我可以将 [dependency path] 状态更新为 Complete 吗？"
   - 如果 [A]：在第 6 阶段摘要中的 "Deviations" 下注明："在不完整的依赖下实现：[dependency title] — [status]。"

如果找不到依赖 story 文件：警告"未找到依赖 story：[path]。验证路径或创建 story 文件。"

---

### 引擎参考
读取 `.claude/docs/technical-preferences.md`：
- `Engine:` 值——确定使用哪个程序员代理
- 命名约定（类名、文件名、信号/事件名）
- 性能预算（帧预算、内存上限）
- 禁止模式

### 标记 Story 进行中

在生成任何代理之前，静默更新两件事：

1. **`production/sprint-status.yaml`**（如果存在）：找到与此 story 文件路径匹配的条目并设置 `status: in_progress`。将顶层 `updated` 字段更新到今天日期。如果文件不存在，静默跳过。

2. **story 文件本身**：将 story 头部的 `Last Updated:` 字段编辑为今天日期（格式：`YYYY-MM-DD`）。如果 story 头部中不存在该字段，请在 `Status:` 行后添加。这为此 story 启用 sprint-status 过时检测。

---

## 第 3 阶段：路由到正确的程序员

根据 story 的 **层级**、**类型** 和 **系统名称**，确定通过 Task 生成哪个专家。

**Config/Data story——完全跳过代理生成：**
如果 story 的类型是 `Config/Data`，不需要程序员代理或引擎专家。直接跳到第 4 阶段（Config/Data 备注）。实现是数据文件编辑——没有路由表评估，没有引擎专家。

### 主要代理路由表

| Story 上下文 | 主要代理 |
|---|---|
| Foundation 层级——任何类型 | `engine-programmer` |
| 任何层级——类型：UI | `ui-programmer` |
| 任何层级——类型：Visual/Feel | `gameplay-programmer`（实现） |
| Core 或 Feature——游戏机制 | `gameplay-programmer` |
| Core 或 Feature——AI 行为、寻路 | `ai-programmer` |
| Core 或 Feature——网络、复制 | `network-programmer` |
| Config/Data——无代码 | 不需要代理（参见第 4 阶段 Config 备注） |

### 引擎专家——始终作为代码故事的次要代理生成

读取 `.claude/docs/technical-preferences.md` 的 `Engine Specialists` 部分以获取配置的主要专家。当 story 涉及引擎特定 API、模式或 ADR 具有 HIGH 引擎风险时，与主要代理一起生成它们。

| 引擎 | 可用的专家代理 |
|--------|----------------------------|
| Godot 4 | `godot-specialist`、`godot-gdscript-specialist`、`godot-shader-specialist` |
| Unity | `unity-specialist`、`unity-ui-specialist`、`unity-shader-specialist` |
| Unreal Engine | `unreal-specialist`、`ue-gas-specialist`、`ue-blueprint-specialist`、`ue-umg-specialist`、`ue-replication-specialist` |

**当引擎风险为 HIGH** 时（来自 ADR 或 VERSION.md）：始终生成引擎专家，即使对于非引擎面向的故事。高风险意味着 ADR 记录了对 cutoff 后引擎 API 的假设，需要专家验证。

---

## 第 4 阶段：实现

通过 Task 生成选定的程序员代理，提供完整的上下文包：

用文件和针对性阅读指示向代理简报——不要将文档内容序列化到 Task 提示中。代理直接读取它需要的内容：

1. **Story 文件**：`[story-path]`——完整读取
2. **GDD 需求**：在 `docs/architecture/tr-registry.yaml` 中查找 TR-ID `[TR-XXX-NNN]`——使用 `requirement` 字段作为事实来源
3. **ADR**：`docs/architecture/[adr-file].md`——仅读取 **Decision** 和 **Implementation Guidelines** 部分
4. **Control manifest**：`docs/architecture/control-manifest.md`——仅读取 **[layer]** 层级的规则
5. **引擎偏好**：`.claude/docs/technical-preferences.md`——读取命名约定和性能预算
6. **测试文件路径**：`[来自 story 的 Test Evidence 部分的路径]`——此文件必须作为实现的一部分创建
7. **测试需求**（仅 Logic 和 Integration story）：测试文件 MUST 在 `[来自 story 的 Test Evidence 部分的路径]` 创建。与实现一起编写测试——不要推迟。`/story-done` 无法关闭没有此文件的 story。每个验收标准必须至少有一个测试函数覆盖它。测试文件命名：`[system]_[feature]_test.[ext]`。函数命名：`test_[scenario]_[expected_outcome]`。无随机种子、无时间相关断言、无外部 I/O。
8. **明确指示**：按照 ADR 指南实现此 story，尊重 manifest 规则，保持在 story 的 Out of Scope 边界内。编写干净、有文档注释的公共 API。

代理应该：
- 按照 ADR 指南在 `src/` 中创建或修改文件
- 尊重 control manifest 中所有必需和禁止的模式
- 保持在 story 的 Out of Scope 边界内（不要触及无关文件）
- 编写干净、有文档注释的公共 API

### Config/Data story（不需要代理）

对于类型：Config/Data 的故事，不需要程序员代理。实现是编辑数据文件。读取 story 的验收标准并直接对数据文件进行指定的更改。注明哪些值已更改以及从/到什么更改。

### Visual/Feel story

生成 `gameplay-programmer` 实现代码/动画调用。注意，Visual/Feel 验收标准无法自动验证——"感觉对吗？"的检查通过手动确认在 `/story-done` 中进行。

---

## 第 5 阶段：测试证据需求

测试需求已包含在第 4 阶段程序员代理简报中（项目 7）。此阶段总结每种故事类型需要什么证据——在收集第 6 阶段摘要时使用。

| Story 类型 | 所需证据 | 备注 |
|---|---|---|
| **Logic** | 在 story 的 Test Evidence 部分的路径上的自动化单元测试 | BLOCKING——包含在第 4 阶段代理简报中 |
| **Integration** | 集成测试或记录的试玩记录 | BLOCKING——包含在第 4 阶段代理简报中 |
| **Visual/Feel** | `production/qa/evidence/[slug]-evidence.md` 的证据文档 | ADVISORY——在第 6 阶段摘要中注明 |
| **UI** | 手动演练文档或交互测试 | ADVISORY——在第 6 阶段摘要中注明 |
| **Config/Data** | 无——冒烟检查作为证据 | N/A |

对于 Visual/Feel 和 UI story，在第 6 阶段摘要中包含："在 `production/qa/evidence/[slug]-evidence.md` 需要手动证据，然后此 story 才能完全关闭。"

---

## 第 6 阶段：收集和总结

程序员代理完成后，收集：

- 创建或修改的文件（含路径）
- 创建的测试文件（路径和编写的测试函数数量）
- 任何偏离 story 的 Out of Scope 边界的情况（标记这些）
- 代理提出的任何问题或阻塞
- 专家标记的任何引擎特定风险

展示简洁的实现摘要：

```
## 实现完成：[Story Title]

**更改的文件**：
- `src/[path]`——创建/修改（[简要描述]）
- `tests/[path]`——测试文件（[N] 个测试函数）

**覆盖的验收标准**：
- [x] [标准]——在 [file:function] 中实现
- [x] [标准]——由测试 [test_name] 覆盖
- [ ] [标准]——推迟：需要试玩（Visual/Feel）

**偏离范围**：[无] 或 [列出 story 边界外触及的文件]
**标记的引擎风险**：[无] 或 [专家发现]
**阻塞**：[无] 或 [描述]

**运行 `/story-done` 之前：** 在本地运行测试套件并确认你编写的测试通过。
`/story-done` 将自动重新运行它们，但那里发现的失败测试意味着返回实现上下文。

准备运行：`/code-review [file1] [file2]` 然后 `/story-done [story-path]`
```

---

## 第 7 阶段：更新会话状态

静默追加到 `production/session-state/active.md`：

```
## 会话摘要 — /dev-story [date]
- Story：[story-path]——[story title]
- 更改的文件：[逗号分隔列表]
- 编写的测试：[path，或 "无——Visual/Feel/Config story"]
- 阻塞：[无，或描述]
- 下一步：/code-review [files] 然后 /story-done [story-path]
```

如果 `active.md` 不存在，创建它。确认："会话状态已更新。"

---

## 错误恢复协议

如果任何生成的代理（通过 Task）返回 BLOCKED、错误或无法完成：

1. **立即展示**：在继续依赖阶段之前向用户报告 "[AgentName]：BLOCKED——[reason]"
2. **评估依赖关系**：检查被阻塞代理的输出是否是后续阶段所必需的。如果是，在没有用户输入的情况下不要超过该依赖点。
3. **通过 AskUserQuestion 提供选项**，选择：
   - 跳过此代理并在最终报告中注明空白
   - 以更窄的范围重试
   - 停在这里并首先解决阻塞
4. **始终生成部分报告**——输出任何完成的内容。永远不要因为一个代理阻塞而丢弃工作。

常见阻塞：
- 输入文件缺失（未找到 story、GDD 缺失）→ 重定向到创建它的技能
- ADR 状态是 Proposed → 不要实现；先运行 `/architecture-decision`
- 范围太大 → 通过 `/create-stories` 拆分为两个 story
- ADR 和 story 之间的冲突指示 → 展示冲突，不要猜测
- Manifest 版本不匹配 → 向用户展示差异，询问是使用旧规则继续还是先更新 story

## 协作协议

- **文件写入是委托的**——所有源代码、测试文件和证据文档都由通过 Task 生成的子代理编写。每个子代理单独执行"我可以写入 [path] 吗？"协议。此编排器不直接写入文件。
- **实现前先加载**——在所有上下文加载之前不要开始编码
  （story、TR-ID、ADR、manifest、引擎偏好）。不完整的上下文会产生偏离设计的代码。
- **ADR 是法律**——实现必须遵循 ADR 的 Implementation Guidelines。如果指南与看起来"更好"的内容冲突，在摘要中标记它而不是默默偏离。
- **保持在范围内**——Out of Scope 部分是一个合同。如果实现 story 需要触及范围外的文件，停止并展示它：
  "实现 [criterion] 需要修改 [file]，这超出了范围。我继续还是创建一个单独的 story？"
- **Logic/Integration 的测试不是可选的**——在测试文件存在之前不要标记实现完成
- **Visual/Feel 标准是推迟的，不是跳过的**——在摘要中将它们标记为 DEFERRED；它们将在 `/story-done` 中手动验证
- **在大结构决策前询问**——如果 story 需要 ADR 未涵盖的架构模式，在实现之前展示它：
  "ADR 没有指定如何处理 [case]。我的计划是 [X]。继续？"

---

## 推荐的后续步骤

- 运行 `/code-review [file1] [file2]` 在关闭 story 之前审查实现
- 运行 `/story-done [story-path]` 验证验收标准并标记 story 完成
- 所有 sprint story 完成后：在推进项目阶段之前运行 `/team-qa sprint` 进行完整的 QA 循环
