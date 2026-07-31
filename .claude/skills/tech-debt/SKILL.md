---
name: tech-debt
description: "Track, categorize, and prioritize technical debt across the codebase. Scans for debt indicators, maintains a debt register, and recommends repayment scheduling."
argument-hint: "[scan|add|prioritize|report]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion
model: sonnet
---

## 阶段 1：解析子命令

从参数确定模式：

- `scan` — 扫描代码库中的技术债务指标
- `add` — 手动添加新的技术债务条目
- `prioritize` — 重新排序现有债务登记册
- `report` — 生成当前债务状态的总结报告

如果未提供子命令，输出用法并停止。结论：**FAIL** — 缺少必需的子命令。

---

## 阶段 2A：扫描模式

搜索代码库中的债务指标：

- `TODO` 注释（计数和分类）
- `FIXME` 注释（这些是伪装的 bug）
- `HACK` 注释（需要正确解决方案的变通方法）
- `@deprecated` 标记
- 重复代码块（多个文件中的相似模式）
- 超过 500 行的文件（潜在的 god objects）
- 超过 50 行的函数（潜在的复杂性）

对每个发现进行分类：

- **Architecture Debt**：错误的抽象、缺失的模式、耦合问题
- **Code Quality Debt**：重复、复杂性、命名、缺失类型
- **Test Debt**：缺失测试、不稳定测试、未测试的边缘情况
- **Documentation Debt**：缺失文档、过时文档、未记录的 API
- **Dependency Debt**：过时的包、废弃的 API、版本冲突
- **Performance Debt**：已知的慢路径、未优化的查询、内存问题

向用户展示发现。

询问："May I write these findings to `docs/tech-debt-register.md`?"

如果同意，更新登记册（追加新条目，不覆盖现有条目）。结论：**COMPLETE** — 扫描发现已写入登记册。

如果不同意，停在这里。结论：**BLOCKED** — 用户拒绝写入。

---

## 阶段 2B：添加模式

向用户询问描述、受影响的文件以及如果不修复的影响（纯文本提示）。

然后使用 `AskUserQuestion` 收集 **分类**：
- 提示："What category does this tech debt belong to?"
- 选项：
  - `[A] Architecture Debt — wrong abstractions, missing patterns, coupling issues`
  - `[B] Code Quality Debt — duplication, complexity, naming, missing types`
  - `[C] Test Debt — missing tests, flaky tests, untested edge cases`
  - `[D] Documentation Debt — missing/outdated docs, undocumented APIs`
  - `[E] Dependency Debt — outdated packages, deprecated APIs, version conflicts`
  - `[F] Performance Debt — known slow paths, memory issues, unoptimized queries`

然后使用 `AskUserQuestion` 收集 **估计修复工作量**：
- 提示："What is the estimated effort to fix this item?"
- 选项：
  - `[A] S — Small (under 1 day)`
  - `[B] M — Medium (1–3 days)`
  - `[C] L — Large (3–7 days)`
  - `[D] XL — Extra Large (over 1 week)`

向用户展示完整的新条目。

询问："May I append this entry to `docs/tech-debt-register.md`?"

如果同意，追加条目。结论：**COMPLETE** — 条目已添加到登记册。

如果不同意，停在这里。结论：**BLOCKED** — 用户拒绝写入。

---

## 阶段 2C：排序模式

读取 `docs/tech-debt-register.md` 中的债务登记册。

按以下公式对每个项目评分：`(impact_if_unfixed × frequency_of_encounter) / fix_effort`

按优先级分数重新排序登记册，并推荐下一个 sprint 应包含的项目。

向用户展示重新排序的登记册。

询问："May I write the re-prioritized register back to `docs/tech-debt-register.md`?"

如果同意，写入更新后的文件。结论：**COMPLETE** — 登记册已重新排序并保存。

如果不同意，停在这里。结论：**BLOCKED** — 用户拒绝写入。

---

## 阶段 2D：报告模式

读取债务登记册。生成总结统计：

- 按分类的项目总数
- 总估计修复工作量
- 自上次报告以来添加与解决的项目
- 趋势方向（增长 / 稳定 / 缩减）

标记登记册中超过 3 个 sprint 的任何项目。

向用户输出报告。此模式是只读的 — 不写入文件。结论：**COMPLETE** — 债务报告已生成。

---

## 阶段 3：下一步

- 运行 `/sprint-plan` 将高优先级债务项目排入下一个 sprint。
- 在每个 sprint 开始时运行 `/tech-debt report` 跟踪债务随时间的趋势。

### 债务登记册格式

```markdown
## Technical Debt Register
Last updated: [Date]
Total items: [N] | Estimated total effort: [T-shirt sizes summed]

| ID | Category | Description | Files | Effort | Impact | Priority | Added | Sprint |
|----|----------|-------------|-------|--------|--------|----------|-------|--------|
| TD-001 | [Cat] | [Description] | [files] | [S/M/L/XL] | [Low/Med/High/Critical] | [Score] | [Date] | [Sprint to fix or "Backlog"] |
```

### 规则
- 技术债务本质上不是坏事 — 它是一个工具。登记册跟踪有意识的决策。
- 每个债务条目必须解释为什么被接受（截止日期、原型、缺失信息）
- "Scan" 应每个 sprint 至少运行一次以捕获新债务
- 超过 3 个 sprint 未处理的项目应被修复或有意识地接受并记录原因
