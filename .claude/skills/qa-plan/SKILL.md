---
name: qa-plan
description: "Generate a QA test plan for a sprint or feature. Reads GDDs and story files, classifies stories by test type (Logic/Integration/Visual/UI), and produces a structured test plan covering automated tests required, manual test cases, smoke test scope, and playtest sign-off requirements. Run before sprint begins or when starting a major feature."
argument-hint: "[sprint | feature: system-name | story: path]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion
model: sonnet
agent: qa-lead
---

# QA 计划

此 skill 为 sprint、功能或单个故事生成结构化的 QA 计划。
它读取所有范围内的故事文件及其引用的 GDD，按测试类型对每个故事进行分类，并生成一个计划，告诉开发者确切应该自动化什么、手动验证什么、smoke test 的范围是什么，以及何时引入 playtester。

在 sprint 开始之前运行此，以便团队提前了解所需的测试工作。实现后编写的测试计划是事后分析，不是计划。

**输出：** `production/qa/qa-plan-[sprint-slug]-[日期].md`

---

## Phase 1: 解析范围

**参数：** `$ARGUMENTS`（空白 = 通过 AskUserQuestion 询问用户）

从参数确定范围：

- **`sprint`** — 读取 `production/sprints/` 中最新的文件，提取引用的每个故事文件路径。如果 `production/sprint-status.yaml` 存在，将其用作主要故事列表，并回退到 sprint 计划获取故事元数据。
- **`feature: [系统名称]`** — glob `production/epics/*/story-*.md`，过滤到文件路径或标题包含系统名称的故事。同时检查该系统目录中的 epic 索引文件（`EPIC.md`）。
- **`story: [路径]`** — 验证路径是否存在并加载该单个文件。
- **无参数** — 使用 `AskUserQuestion`：
  - "此 QA 计划的范围是什么？"
  - 选项："当前 sprint"、"特定功能（输入系统名称）"、
    "特定故事（输入路径）"、"完整 epic"

解析范围后，报告："正在为 [范围] 中的 [N] 个故事构建 QA 计划。"

如果引用了故事文件路径但文件不存在，将其记为
MISSING 并继续处理剩余的故事。不要因为一个文件缺失而让整个计划失败。

---

## Phase 2: 加载输入

对于每个范围内的故事文件，读取完整文件并提取：

- **故事标题** 和故事 ID（来自文件名或标题）
- **Story Type** 字段（如果文件标题中存在 — 例如 `Type: Logic`）
- **验收标准** — 完整的编号/要点列表
- **实现文件** — 列在"Files to Create / Modify"或类似标题下
- **引擎注释** — 任何引擎 API 警告或版本特定注释
- **GDD 引用** — 引用的 GDD 路径
- **ADR 引用** — 引用的 ADR
- **估算** — 小时或故事点（如有）
- **依赖** — 此故事依赖的其他故事

阅读故事后，一次性加载支持上下文（不是每个故事都加载）：

- `design/gdd/systems-index.md` — 了解系统优先级以及哪些
  GDD 已批准
- 对于所有故事中引用的每个唯一 GDD：读取
  **Acceptance Criteria**、**Formulas** 和 **Edge Cases** 部分。不要加载
  完整的 GDD 文本。这三个部分包含可测试的需求、要验证的数学
  以及测试必须覆盖的边界条件。如果 GDD 中缺少 Edge Cases
  部分，按 GDD 记录："未找到 Edge Cases 部分 — 边缘情况覆盖将仅从验收标准推断。"
- `docs/architecture/control-manifest.md` — 扫描自动化测试应该防范的禁止模式（如果文件存在）

如果故事中未引用 GDD，将其记为缺口但不阻塞计划。
该故事将仅使用验收标准进行分类。

---

## Phase 3: 分类每个故事

对于每个故事，分配一个 Story Type：

- **如果故事标题中已有 `Type:` 字段**：按原样接受。不要根据以下标准重新分类或验证 — 该 Type 由 lead-programmer 在故事创建时设置，具有权威性。按原样记录。
- **如果 `Type:` 字段缺失**：使用下表从验收标准推断类型，并在报告中注明类型是推断的（不是声明的）。将此标记为缺口 — 故事应在实现开始之前明确声明其 Type。

| Story Type | 分类指标 |
|---|---|
| **Logic** | 验收标准引用计算、公式、数值阈值、状态转换、AI 决策、数据验证、buff/debuff 叠加、经济交易或任何可测试的计算 |
| **Integration** | 标准涉及两个或多个系统交互、信号或事件跨系统边界传播、保存/加载往返、网络同步或持久化 |
| **Visual/Feel** | 标准引用动画行为、VFX、shader 输出、"感觉响应"、感知时机、屏幕震动、粒子效果、音频同步或视觉反馈质量 |
| **UI** | 标准引用菜单、HUD 元素、按钮、屏幕、对话框、库存面板、工具提示或任何面向玩家的界面元素 |
| **Config/Data** | 变更仅限于平衡调优值、数据文件或配置 — 不涉及新代码逻辑 |

**混合故事**（例如，同时添加公式和 UI 显示的故事）：
根据哪个验收标准具有最高的实现风险分配主要类型，并注明次要类型。Logic+Integration 或
Visual+UI 组合最常见。

对所有故事进行分类后，在继续到 Phase 4 之前在对话中生成分类摘要表格。这为用户提供了测试如何分配的可见性。

---

## Phase 4: 生成测试计划

组装完整的 QA 计划文档。使用此结构：

````markdown
# QA 计划: [Sprint/功能名称]
**日期**: [日期]
**生成者**: /qa-plan
**范围**: [N 个故事跨 [N 个系统]]
**引擎**: [.claude/docs/technical-preferences.md 中的引擎名称，或"未配置"]
**Sprint 文件**: [sprint 计划路径（如适用）]

---

## 测试摘要

| 故事 | 类型 | 需要自动化测试 | 需要手动验证 |
|-------|------|------------------------|------------------------------|
| [故事标题] | Logic | 单元测试 — `tests/unit/[系统]/` | 无 |
| [故事标题] | Integration | 集成测试 — `tests/integration/[系统]/` | Smoke check |
| [故事标题] | Visual/Feel | 无（不可自动化） | 截图 + 主管签字 |
| [故事标题] | UI | 交互演练 | 手动逐步执行 |
| [故事标题] | Config/Data | 数据验证测试 | 在游戏中抽查数值 |

---

## 需要的自动化测试

### [故事标题] — [类型]
**测试文件路径**: `tests/[unit|integration]/[系统]/[故事-slug]_test.[ext]`
**测试内容**:
- [来自 GDD Formulas 部分的具体公式或规则]
- [每个命名的状态转换或决策分支]
- [每个应该或不应该发生的副作用]

**要覆盖的边缘情况**:
- 零/最小输入值（例如，0 伤害、空库存）
- 最大/边界输入值（例如，最大等级、属性上限）
- 无效或空输入（例如，缺失目标、死亡实体）
- [GDD Edge Cases 部分明确指出的任何边缘情况]

**估算测试数量**: ~[N] 个单元测试

[如果此故事在引用的 GDD 中未找到公式，注明:]
*引用的 GDD 中未找到公式 — 测试用例必须直接从验收标准推导。在编写测试之前审查 GDD Formulas 部分。*

---

## 手动 QA 检查清单

### [故事标题] — [类型]
**验证方法**: [截图 + 设计师签字 | Playtest 会话 |
手动逐步执行 | 与参考画面比较]
**必须签字者**: [设计师 / lead-programmer / qa-lead / art-lead]
**要捕获的证据**: [X 的截图 | Y 的视频片段 | 书面 playtest
笔记 | 并排比较]

检查清单:
- [ ] [具体可观察条件 — 具体且可证伪]
- [ ] [另一个条件]
- [ ] [每个验收标准转化为手动检查项]

*如果任何标准使用主观语言（"感觉"、"看起来"、"似乎"），必须
用具体基准或 playtest 协议注释补充。*

---

## Smoke Test 范围

在此 sprint 的任何 QA 交接之前验证的关键路径：

1. 游戏启动到主菜单无崩溃
2. 可以开始新游戏/新会话
3. [本 sprint 引入或更改的主要机制]
4. [本 sprint 变更带来回归风险的任何系统]
5. 保存/加载循环完成无数据损失（如果存在保存系统）
6. 性能在目标硬件上在预算内（无新帧峰值）

*Smoke test 由开发者通过 `/smoke-check` 验证。运行该 skill 时引用此列表。*

---

## Playtest 需求

| 故事 | Playtest 目标 | 最少会话数 | 目标玩家类型 |
|-------|--------------|--------------|-------------------|
| [故事] | [会话必须回答什么问题？] | [N] | [新玩家 / 有经验玩家] |

**签字要求**: Playtest 笔记必须写入
`production/session-logs/playtest-[sprint]-[故事-slug].md` 并由
[设计师 / qa-lead] 审查，然后故事才能标记为 COMPLETE。

如果不需要故事进行 playtest 验证：*本 sprint 不需要 playtest 会话。*

---

## 完成定义 — 本 Sprint

当以下所有条件都为真时，故事才算完成：

- [ ] 所有验收标准已验证 — 通过自动化测试结果 OR 记录的
      手动证据（截图、视频或含签字的 playtest 笔记）
- [ ] Logic 和 Integration 故事在指定路径存在测试文件
- [ ] Visual/Feel 和 UI 故事存在手动证据文档
- [ ] Smoke check 通过（在 QA 交接前运行 `/smoke-check sprint`）
- [ ] 未引入回归
- [ ] 代码已审查（通过 `/code-review` 或记录的同事审查）
- [ ] 故事文件已更新为 `Status: Complete`（通过 `/story-done`）
````

生成内容时，使用 Phase 2 中提取的实际故事标题、GDD 公式文本和
验收标准。不要使用占位文本 — 每个测试条目都应反映这些特定故事的真实需求。

---

## Phase 5: 写入输出

在对话中展示完整计划（如果计划很长则展示摘要），
然后使用 `AskUserQuestion` 一起问两个问题：

```
question: "准备写入 QA 计划。选择输出选项:"
multiSelect: true
options:
  - "将 QA 计划写入 production/qa/qa-plan-[sprint-slug]-[日期].md"
  - "还将测试用例规范回填到每个故事文件的 ## QA Test Cases 部分（推荐 — 启用 /dev-story 和 /code-review 可追溯性）"
```

如果选择了"写入 QA 计划"：按生成的原样写入计划文件 — 不要截断。

如果选择了"还回填故事文件"：对于范围内的每个 Logic 和 Integration 故事，编辑其路径处的故事文件。找到 `## QA Test Cases` 部分并将其内容替换为 Phase 4 为该故事生成的测试用例规范。如果故事没有 `## QA Test Cases` 部分，在 `## Test Evidence` 之前追加。对于 Visual/Feel 和 UI 故事，写入手动验证步骤而不是测试规范。

写入后：

"QA 计划已写入 `production/qa/qa-plan-[sprint-slug]-[日期].md`。

后续步骤：
- 在 sprint 实现开始之前与团队分享此计划
- 一旦所有 sprint 故事实现完成，运行 `/smoke-check sprint` 以 gate QA 交接 — 还不是现在，仅在实现完成后
- 对于 Logic/Integration 故事，在标记故事完成之前
  在列出的路径创建测试文件 — `/story-done` 会检查它们"

静默追加到 `production/session-state/active.md`（如果文件不存在则创建）：

```
<!-- QA-PLAN: [日期] | 系统: [系统/sprint 标识符] | 计划已写入: production/qa/qa-plan-[标识符]-[日期].md -->
```

---

## 协作协议

- **永远不要未经询问就写入计划** — Phase 5 需要明确批准。
- **保守分类**：当故事在 Logic 和 Integration 之间模糊时，
  将其分类为 Integration — 它需要单元测试和
  集成测试。
- **不要编造测试用例**，超出验收标准和 GDD 公式
  支持的范围。如果 GDD 中缺少公式，标记它而不是猜测。
- **Playtest 需求是建议性的**：用户决定是否对边缘的 Visual/Feel 故事
  进行 playtest。标记情况；不要强制。
- 当未提供参数时，使用 `AskUserQuestion` 进行范围选择。
  保持所有其他阶段非交互式 — 展示结果，然后询问一次以
  批准写入。
