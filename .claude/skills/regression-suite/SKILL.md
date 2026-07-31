---
name: regression-suite
description: "Map test coverage to GDD critical paths, identify fixed bugs without regression tests, flag coverage drift from new features, and maintain tests/regression-suite.md. Run after implementing a bug fix or before a release gate."
argument-hint: "[update | audit | report]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
model: sonnet
---

# 回归测试套件

此 skill 确保每个 bug 修复都有本可以捕获原始 bug 的测试作为后盾 — 并且随着游戏的演进回归测试套件保持最新。它还检测何时添加了新功能但没有相应的回归覆盖。

回归测试套件不是新的测试类别 — 它是 `tests/` 中**精选的测试列表**，共同覆盖游戏的关键路径和已知故障点。此 skill 维护该列表。

**输出：** `tests/regression-suite.md`

**何时运行：**
- 修复 bug 后（确认已编写回归测试或识别缺口）
- 在发布门槛之前（`/gate-check polish` 要求回归测试套件存在）
- 作为 sprint 结束的一部分以检测覆盖漂移

---

## 1. 解析参数

**模式：**
- `/regression-suite update` — 扫描本 sprint 的新 bug 修复并检查回归测试是否存在；将新测试添加到套件清单
- `/regression-suite audit` — 对所有 GDD 关键路径与现有测试覆盖进行全面审计；标记没有回归测试的路径
- `/regression-suite report` — 只读状态报告（无写入）；适合 sprint 审查
- 无参数 — 如果 sprint 明显活跃（存在包含进行中故事的 sprint 计划），运行 `update`。如果模糊或未检测到活跃 sprint，使用 `AskUserQuestion`：
  - 提示："未指定子命令。您想运行哪个模式？"
  - 选项：
    - `[A] update — 扫描本 sprint 的新 bug 修复并添加缺失的回归测试`
    - `[B] audit — 对所有 GDD 关键路径与现有测试覆盖进行全面审计`
    - `[C] report — 只读状态报告（无写入）`

---

## 2. 加载上下文

### 步骤 2a — 加载现有回归测试套件

如果存在，读取 `tests/regression-suite.md`。提取：
- 已注册的回归测试总数
- 最后更新日期
- 任何标记为 `STALE` 或 `QUARANTINED` 的测试

如果不存在：记录"未找到回归测试套件 — 将创建一个。"

### 步骤 2b — 加载测试清单

Glob 所有测试文件：
```
tests/unit/**/*_test.*
tests/integration/**/*_test.*
tests/regression/**/*
```

对于每个文件，记录系统（来自目录路径）和文件名。
除非需要名称到测试的映射，否则不要读取测试文件内容。

### 步骤 2c — 加载 GDD 关键路径

对于 `audit` 模式：读取 `design/gdd/systems-index.md` 获取所有系统。
对于每个 MVP 层级的系统，读取其 GDD 并提取：
- 验收标准（这些定义了关键路径）
- Formulas 部分（公式必须有回归测试）
- Edge Cases 部分（已知边缘情况应该有回归测试）

对于 `update` 模式：跳过完整 GDD 扫描。而是读取当前 sprint
计划和故事文件以找到本 sprint 中状态为 Status: Complete 的故事。

### 步骤 2d — 加载已关闭的 bug

Glob `production/qa/bugs/*.md` 并过滤具有 `Status: Closed`
或 `Status: Fixed` 字段的 bug。记录：
- bug 在哪个故事或系统中
- 修复描述中是否提到了回归测试

---

## 3. 映射覆盖 — 关键路径

仅对于 `audit` 模式：

对于每个 GDD 验收标准，确定是否存在测试：

1. Grep `tests/unit/[系统]/` 和 `tests/integration/[系统]/` 获取与标准的关键名词/动词相关的文件名和函数名
2. 分配覆盖：

| 状态 | 含义 |
|--------|---------|
| **COVERED** | 存在针对此标准逻辑的测试文件 |
| **PARTIAL** | 存在测试但未覆盖所有情况（例如仅 happy path） |
| **MISSING** | 未找到此关键路径的测试 |
| **EXEMPT** | Visual/Feel 或 UI 标准 — 按设计不可自动化 |

3. 将对应于公式或状态机的 MISSING 项目提升为
   **高优先级**缺口 — 这些是最可能的回归源。

---

## 4. 映射覆盖 — 已修复的 Bug

对于每个已关闭的 bug：

1. 从 bug 的元数据中提取系统 slug
2. Grep `tests/unit/[系统]/` 和 `tests/integration/[系统]/` 获取引用 bug ID 或特定故障场景的测试
3. 分配：
   - **HAS REGRESSION TEST** — 找到了会捕获此 bug 的测试
   - **MISSING REGRESSION TEST** — bug 已修复但没有测试防范再次发生

对于 MISSING REGRESSION TEST 项目：
- 将其标记为回归缺口
- 建议测试文件路径：`tests/unit/[系统]/[bug-slug]_regression_test.[ext]`
- 记录："没有此测试，此 bug 可以在未来的 sprint 中悄悄回来。"

---

## 5. 检测覆盖漂移

当游戏增长但回归测试套件没有时，就会发生覆盖漂移。

检查漂移指标：
- 本 sprint 完成的故事在 `tests/` 中没有对应的测试文件
- 自上次回归测试套件更新以来在 `systems-index.md` 中添加了新系统
- 自上次回归测试套件更新以来 GDD 部分被添加或修订
  （如果可用，在 GDD 文件修改提示上使用 Grep，或询问用户）
- `tests/regression-suite.md` 的最后更新日期与当前日期 — 如果差距 >
  2 个 sprint，标记为可能过时

---

## 6. 生成报告和套件清单

### 报告格式（在对话中）

```
## 回归测试套件状态

**模式**: [update | audit | report]
**现有已注册测试**: [N]
**已扫描测试文件**: [N]

### 关键路径覆盖（仅 audit 模式）
| 系统 | 总 AC 数 | 已覆盖 | 部分 | 缺失 | 豁免 |
|--------|-----------|---------|---------|---------|--------|
| [名称] | [N] | [N] | [N] | [N] | [N] |

**覆盖率（非豁免）**: [N]%

### Bug 回归覆盖
| Bug ID | 系统 | 严重程度 | 有回归测试？ |
|--------|--------|----------|----------------------|
| BUG-NNN | [系统] | S[N] | 是 / 否 ⚠ |

**无回归测试的 bug**: [N]

### 覆盖漂移指标
[列出没有测试覆盖的新系统或故事，或"未检测到。"]

### 推荐的新回归测试
| 优先级 | 系统 | 建议的测试文件 | 覆盖 |
|----------|--------|---------------------|--------|
| 高 | [系统] | `tests/unit/[系统]/[slug]_regression_test.[ext]` | BUG-NNN / AC-[N] |
| 中 | [系统] | `tests/unit/[系统]/[slug]_test.[ext]` | [标准] |
```

### 套件清单格式（`tests/regression-suite.md`）

清单是精选索引 — 不是测试本身，而是记录哪些测试在发布前应始终通过的注册表：

```markdown
# 回归测试套件清单

> 最后更新: [日期]
> 已注册测试总数: [N]
> 覆盖: [N]% 的 GDD 关键路径

## 如何运行

[运行所有回归测试的引擎特定命令]

## 已注册的回归测试

### [系统名称]

| 测试文件 | 测试函数（如已知） | 覆盖 | 添加日期 |
|-----------|--------------------------|--------|-------|
| `tests/unit/[系统]/[文件]_test.[ext]` | `test_[场景]` | AC-N / BUG-NNN | [日期] |

## 已知缺口

应该存在但尚未存在的测试：

| 优先级 | 系统 | 建议路径 | 覆盖 | 尚未编写的原因 |
|----------|--------|----------------|--------|------------------------|
| 高 | [系统] | `tests/unit/[系统]/[路径]` | BUG-NNN | Bug 已修复但无测试 |

## 隔离的测试

不稳定或禁用的测试（不在 CI 中运行）：

| 测试文件 | 函数 | 原因 | 隔离日期 |
|-----------|----------|--------|-------------------|
| (无) | | | |
```

---

## 7. 写入输出

询问："我可以将 `tests/regression-suite.md` 写入/更新为当前的
回归测试套件清单吗？"

对于 `update` 模式：追加新条目；永远不要删除现有条目
（使用 `Edit` 进行有针对性的插入）。
对于 `audit` 模式：用更新的覆盖数据重写完整清单。
对于 `report` 模式：不写入任何内容。

写入后（如果已批准）：

- 对于每个高优先级缺口："考虑在下一个 sprint 之前创建缺失的回归测试。运行 `/test-helpers` 以搭建测试文件。"
- 如果 bug 回归缺口 > 0："这些 bug 在没有回归测试的情况下可以悄悄回来。下一个 sprint 应该包含一个故事来编写缺失的测试。"
- 如果检测到覆盖漂移："回归测试套件可能在漂移。考虑在下一个 sprint 边界运行 `/regression-suite audit`。"

判定：**COMPLETE** — 回归测试套件已更新。（如果用户拒绝写入：判定：**BLOCKED**。）

---

## 协作协议

- **永远不要从清单中删除现有回归测试**，未经
  用户明确批准 — 删除故意编写的测试本身就是回归风险
- **缺口是建议性的，不是阻塞的** — 清楚地展示它们但不阻止
  其他工作继续进行（除非在发布门槛处要求回归测试套件）
- **隔离不是删除** — 有间歇性失败的测试应该被隔离
  （在清单中记录）但不删除；它们应该由
  `/test-flakiness` 修复
- **写入前询问** — 在创建或更新清单之前始终确认
