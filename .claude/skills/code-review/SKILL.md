---
name: code-review
description: "Performs an architectural and quality code review on a specified file or set of files. Checks for coding standard compliance, architectural pattern adherence, SOLID principles, testability, and performance concerns."
argument-hint: "[path-to-file-or-directory]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Task, AskUserQuestion
model: sonnet
agent: lead-programmer
---

## 阶段 1：加载目标文件

完整读取目标文件。读取 CLAUDE.md 以获取项目编码标准。

---

## 阶段 2：识别引擎专家

读取 `.claude/docs/technical-preferences.md`，部分 `## Engine Specialists`。注意：

- **主要**专家（用于架构和广泛的引擎问题）
- **语言/代码专家**（在审查项目的主要语言文件时使用）
- **着色器专家**（在审查着色器文件时使用）
- **UI 专家**（在审查 UI 代码时使用）

如果该部分读取 `[TO BE CONFIGURED]`，则没有固定引擎 —— 跳过引擎专家步骤。

---

## 阶段 3：ADR 合规性检查

**参数：** `/code-review [file(s)]` 可以选择性地将 story 文件路径作为最后一个参数（例如 `/code-review src/combat/attack.gd production/epics/combat/story-001.md`）。如果提供了 story 路径，读取它以提取管辖 ADR 引用。

按优先级顺序搜索 ADR 引用：
1. Story 文件（如果作为参数提供）
2. 实现文件顶部的头部注释
3. 引用这些文件的提交消息（`git log --oneline -- [file]`）

查找 `ADR-NNN` 或 `docs/architecture/ADR-` 等模式。

如果未找到 ADR 引用，注意："未找到 ADR 引用 —— ADR 合规性检查已跳过。对于完整的 ADR 合规性审查，请提供 story 路径：`/code-review [files] [story-path]`。"

对于每个引用的 ADR：读取文件，提取 **Decision** 和 **Consequences** 部分，然后分类任何偏差：

- **ARCHITECTURAL VIOLATION**（BLOCKING）：使用 ADR 中明确拒绝的模式
- **ADR DRIFT**（WARNING）：有意义地偏离所选方法而不使用禁止模式
- **MINOR DEVIATION**（INFO）：不影响整体架构的 ADR 指导的小差异

---

## 阶段 4：标准合规性

识别系统类别（引擎、gameplay、AI、networking、UI、工具）并评估：

- [ ] 公共方法和类有文档注释
- [ ] 每个方法的圈复杂度低于 10
- [ ] 没有方法超过 40 行（不包括数据声明）
- [ ] 依赖项被注入（游戏状态没有静态单例）
- [ ] 配置值从数据文件加载
- [ ] 系统暴露接口（不是具体类依赖）

---

## 阶段 5：架构和 SOLID

**架构：**
- [ ] 正确的依赖方向（engine <- gameplay，而不是相反）
- [ ] 模块之间没有循环依赖
- [ ] 正确的分层（UI 不拥有游戏状态）
- [ ] 事件/信号用于跨系统通信
- [ ] 与代码库中已建立的模式一致

**SOLID：**
- [ ] 单一职责：每个类有一个改变的理由
- [ ] 开闭原则：可扩展而无需修改
- [ ] 里氏替换：子类型可替代基类型
- [ ] 接口隔离：没有胖接口
- [ ] 依赖倒置：依赖抽象，而不是具体

---

## 阶段 6：游戏特定问题

- [ ] 帧率独立（delta time 使用）
- [ ] 热路径中无分配（更新循环）
- [ ] 正确的 null/空状态处理
- [ ] 需要时的线程安全
- [ ] 资源清理（无泄漏）

---

## 阶段 7：专家审查（并行）

同时通过 Task 派生所有适用的专家 —— 不要等待一个完成再开始下一个。

### 引擎专家

如果配置了引擎，确定每个文件适用哪个专家并并行派生：

- 主要语言文件（`.gd`、`.cs`、`.cpp`）→ 语言/代码专家
- 着色器文件（`.gdshader`、`.hlsl`、shader graph）→ 着色器专家
- UI 屏幕/小部件代码 → UI 专家
- 跨领域或不明确 → 主要专家

还要为任何触及引擎架构（场景结构、节点层次、生命周期钩子）的文件派生**主要专家**。

### QA 可测试性审查

对于 Logic 和 Integration 故事，还要与引擎专家并行通过 Task 派生 `qa-tester`。传入：
- 正在审查的实现文件
- 故事的 `## QA Test Cases` 部分（来自 qa-lead 的预写测试规格）
- 故事的 `## Acceptance Criteria`

要求 qa-tester 评估：
- [ ] 所有测试钩子和接口是否暴露（不隐藏在 private/internal 访问后面）？
- [ ] 故事的 `## QA Test Cases` 部分中的 QA 测试用例是否映射到可测试代码路径？
- [ ] 是否有任何验收标准在实现后无法测试（例如，硬编码值，没有注入接缝）？
- [ ] 实现是否引入了现有 QA 测试用例未覆盖的任何新边缘情况？
- [ ] 是否有任何应该有测试但没有的可观察副作用？

对于 Visual/Feel 和 UI 故事：qa-tester 审查 `## QA Test Cases` 中的手动验证步骤是否可以通过实现实现 —— 例如，"手动检查器需要达到的状态是否真的可以达到？"

在生成输出之前收集所有专家发现。

---

## 阶段 8：输出审查

```
## Code Review: [文件/系统名称]

### 引擎专家发现: [N/A — 未配置引擎 / CLEAN / ISSUES FOUND]
[引擎专家的发现，或"未配置引擎。"如果已跳过]

### 可测试性: [N/A — Visual/Feel 或 Config story / TESTABLE / GAPS / BLOCKING]
[qa-tester 发现：测试钩子、覆盖缺口、不可测试路径、新边缘情况]
[如果 BLOCKING：实现必须在 ## QA Test Cases 中的测试运行之前暴露 [X]]

### ADR 合规性: [NO ADRS FOUND / COMPLIANT / DRIFT / VIOLATION]
[列出每个检查的 ADR、结果和任何带有严重性的偏差]

### 标准合规性: [X/6 passing]
[列出带有行引用的失败]

### 架构: [CLEAN / MINOR ISSUES / VIOLATIONS FOUND]
[列出具体的架构问题]

### SOLID: [COMPLIANT / ISSUES FOUND]
[列出具体的违规]

### 游戏特定问题
[列出游戏开发特定问题]

### 正面观察
[做得好的地方 —— 始终包含此部分]

### 必需更改
[批准前必须修复的项目 —— ARCHITECTURAL VIOLATION 始终出现在这里]

### 建议
[可有可无的改进]

### 裁决: [APPROVED / APPROVED WITH SUGGESTIONS / CHANGES REQUIRED]
```

此 skill 是只读的 —— 不写入文件。

---

## 阶段 9：下一步

使用 `AskUserQuestion`：
- 提示："代码审查完成 —— 裁决：[APPROVED / CHANGES REQUIRED / MAJOR REVISION]。你希望如何继续？"
- 选项（根据裁决调整）：
  - 如果 APPROVED：
    - `[A] Run /story-done 以标记故事完成`
    - `[B] Stop here`
  - 如果 CHANGES REQUIRED 或 MAJOR REVISION：
    - `[A] 修复问题并重新运行 /code-review`
    - `[B] Run /story-done anyway with noted exceptions`
    - `[C] Stop here`

如果发现 ARCHITECTURAL VIOLATION：
- 如果违规与**现有 ADR** 矛盾：修复实现以符合 `docs/architecture/[adr-file].md`。如果设计确实发生了变化，运行 `/architecture-decision` 正式*修订*现有 ADR —— 不要创建一个竞争的。
- 如果**没有 ADR** 存在针对被违规的模式：在修复代码之前运行 `/architecture-decision` 记录正确的方法。
