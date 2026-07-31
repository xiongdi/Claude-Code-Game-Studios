---
name: consistency-check
description: "Scan all GDDs against the entity registry to detect cross-document inconsistencies: same entity with different stats, same item with different values, same formula with different variables. Grep-first approach — reads registry then targets only conflicting GDD sections rather than full document reads."
argument-hint: "[full | since-last-review | entity:<name> | item:<name>]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, AskUserQuestion
model: sonnet
---

# Consistency Check

通过将所有 GDD 与实体注册表（`design/registry/entities.yaml`）进行比较来检测跨文档不一致。使用 grep 优先方法：读取一次注册表，然后仅针对提及注册名称的 GDD 部分 —— 除非需要调查冲突，否则不进行完整文档读取。

**此 skill 是写入时的安全网。** 它捕获 `/design-system` 的逐节检查可能遗漏的内容，以及 `/review-all-gdds` 的整体审查捕获太晚的内容。

**何时运行：**
- 写入每个新 GDD 后（在移动到下一个系统之前）
- `/review-all-gdds` 之前（以便该 skill 以干净的基线开始）
- `/create-architecture` 之前（不一致会毒害下游 ADR）
- 按需：`/consistency-check entity:[name]` 专门检查一个实体

**输出：** 冲突报告 + 可选的注册表更正

---

## 阶段 1：解析参数和加载注册表

**模式：**
- 无参数 / `full` —— 检查所有注册条目与所有 GDD
- `since-last-review` —— 仅检查自上次审查报告以来修改的 GDD
- `entity:<name>` —— 跨所有 GDD 检查一个特定实体
- `item:<name>` —— 跨所有 GDD 检查一个特定物品

**加载注册表：**

```
Read path="design/registry/entities.yaml"
```

如果文件不存在或没有条目：
> "实体注册表为空。运行 `/design-system` 写入 GDD —— 注册表
> 在每个 GDD 完成后自动填充。还没有要检查的内容。"

停止并退出。

从注册表构建四个查找表：
- **entity_map**：`{ name → { source, attributes, referenced_by } }`
- **item_map**：`{ name → { source, value_gold, weight, ... } }`
- **formula_map**：`{ name → { source, variables, output_range } }`
- **constant_map**：`{ name → { source, value, unit } }`

统计注册条目总数。报告：
```
Registry loaded: [N] entities, [N] items, [N] formulas, [N] constants
Scope: [full | since-last-review | entity:name]
```

---

## 阶段 2：定位范围内的 GDD

```
Glob pattern="design/gdd/*.md"
```

排除：`game-concept.md`、`systems-index.md`、`game-pillars.md` —— 这些不是系统 GDD。

对于 `since-last-review` 模式：
```bash
git log --name-only --pretty=format: -- design/gdd/ | grep "\.md$" | sort -u
```
限制为自最近的 `design/gdd/gdd-cross-review-*.md` 文件创建日期以来修改的 GDD。

在扫描前报告范围内的 GDD 列表。

---

## 阶段 3：Grep 优先冲突扫描

对于每个注册条目，grep 每个范围内 GDD 以查找条目的名称。
不要完整读取 —— 仅提取匹配行及其直接上下文（-C 3 行）。

这是核心优化：不是读取 10 个 GDD × 每个 400 行
（4,000 行），而是 grep 50 个实体名称 × 10 个 GDD（50 个有针对性的搜索，
每个命中返回约 10 行）。

### 3a：实体扫描

对于 entity_map 中的每个实体：

```
Grep pattern="[entity_name]" glob="design/gdd/*.md" output_mode="content" -C 3
```

对于每个 GDD 命中，提取实体名称附近提到的值：
- 任何数字属性（计数、成本、持续时间、范围、速率）
- 任何分类属性（类型、层级、类别）
- 任何派生值（总计、输出、结果）
- entity_map 中注册的任何其他属性

将提取的值与注册表条目进行比较。

**冲突检测：**
- 注册表说 `[entity_name].[attribute] = [value_A]`。GDD 说 `[entity_name] has [value_B]`。→ **CONFLICT**
- 注册表说 `[item_name].[attribute] = [value_A]`。GDD 说 `[item_name] is [value_B]`。→ **CONFLICT**
- GDD 提到 `[entity_name]` 但未指定属性。→ **NOTE**（无冲突，仅不可验证）

### 3b：物品扫描

对于 item_map 中的每个物品，grep 所有 GDD 以查找物品名称。提取：
- 出售价格 / 价值 / 金币价值
- 重量
- 堆叠规则（可堆叠 / 不可堆叠）
- 类别

与注册表条目值进行比较。

### 3c：公式扫描

对于 formula_map 中的每个公式，grep 所有 GDD 以查找公式名称。提取：
- 公式附近提到的变量名称
- 提到的输出范围或上限值

与注册表条目进行比较：
- 不同的变量名称 → **CONFLICT**
- 输出范围不同 → **CONFLICT**

### 3d：常数扫描

对于 constant_map 中的每个常数，grep 所有 GDD 以查找常数名称。提取：
- 常数名称附近提到的任何数字值

与注册表值进行比较：
- 不同的数字 → **CONFLICT**

---

## 阶段 4：深入调查（仅冲突）

对于阶段 3 中发现的每个冲突，对冲突 GDD 进行有针对性的完整部分读取以获取精确上下文：

```
Read path="design/gdd/[conflicting_gdd].md"
```
（或者如果文件较大，使用 Grep 配合更宽的上下文）

用完整上下文确认冲突。确定：
1. **哪个 GDD 是正确的？** 检查注册表中的 `source:` 字段 —— 源 GDD 是权威所有者。任何其他与之矛盾的 GDD 是需要更新的。
2. **注册表本身是否过时？** 如果源 GDD 在注册表条目写入后更新（检查 git log），注册表可能已过时。
3. **这是真正的设计变更吗？** 如果冲突代表有意的设计决策，解决方案是：更新源 GDD，更新注册表，然后修复所有其他 GDD。

对于每个冲突，分类：
- **🔴 CONFLICT** —— 同名实体/物品/公式/常数在不同 GDD 中有不同值。在架构开始之前必须解决。
- **⚠️ STALE REGISTRY** —— 源 GDD 值已更改但注册表未更新。注册表需要更新；其他 GDD 可能已经正确。
- **ℹ️ UNVERIFIABLE** —— 实体被提及但未说明可比较的属性。不是冲突；仅记录引用。

---

## 阶段 5：输出报告

```
## Consistency Check Report
Date: [date]
Registry entries checked: [N entities, N items, N formulas, N constants]
GDDs scanned: [N] ([list names])

---

### Conflicts Found (must resolve before architecture)

🔴 [Entity/Item/Formula/Constant Name]
   Registry (source: [gdd]): [attribute] = [value]
   Conflict in [other_gdd].md: [attribute] = [different_value]
   → Resolution needed: [which doc to change and to what]

---

### Stale Registry Entries (registry behind the GDD)

⚠️ [Entry Name]
   Registry says: [value] (written [date])
   Source GDD now says: [new value]
   → Update registry entry to match source GDD, then check referenced_by docs.

---

### Unverifiable References (no conflict, informational)

ℹ️ [gdd].md mentions [entity_name] but states no comparable attributes.
   No conflict detected. No action required.

---

### Clean Entries (no issues found)

✅ [N] registry entries verified across all GDDs with no conflicts.

---

Verdict: PASS | CONFLICTS FOUND
```

**裁决：**
- **PASS** —— 无冲突。注册表和 GDD 在所有检查的值上达成一致。
- **CONFLICTS FOUND** —— 检测到一个或多个冲突。列出解决步骤。

---

## 阶段 6：注册表更正

如果发现过时的注册表条目，询问：
> "我可以更新 `design/registry/entities.yaml` 以修复 [N] 个过时条目吗？"

对于每个过时条目：
- 更新 `value` / 属性字段
- 将 `revised:` 设置为今天的日期
- 添加带有旧值的 YAML 注释：`# was: [old_value] before [date]`

如果在 GDD 中发现注册表中不存在的新条目，询问：
> "在 GDD 中发现 [N] 个实体/物品被提及但尚未在注册表中。
> 我可以将它们添加到 `design/registry/entities.yaml` 吗？"

仅添加出现在多个 GDD 中的条目（真正的跨系统事实）。

**永远不要删除注册表条目。** 如果条目从所有 GDD 中删除，设置 `status: deprecated`。

写入后：裁决：**COMPLETE** —— 一致性检查完成。
如果冲突仍未解决：裁决：**BLOCKED** —— [N] 个冲突需要在架构开始之前手动解决。

### 6b：追加到 Reflexion 日志

如果发现任何 🔴 CONFLICT 条目（无论是否解决），
为每个冲突追加一个条目到 `docs/consistency-failures.md`：

```markdown
### [YYYY-MM-DD] — /consistency-check — 🔴 CONFLICT
**Domain**: [system domain(s) involved]
**Documents involved**: [source GDD] vs [conflicting GDD]
**What happened**: [specific conflict — entity name, attribute, differing values]
**Resolution**: [how it was fixed, or "Unresolved — manual action needed"]
**Pattern**: [generalised lesson, e.g. "Item values defined in combat GDD were not
referenced in economy GDD before authoring — always check entities.yaml first"]
```

如果 `docs/consistency-failures.md` 不存在，在追加之前用此头部创建它：

```markdown
# Consistency Failure Log

<!-- Auto-maintained by /consistency-check. Do not edit manually. -->
<!-- One entry per detected conflict, in chronological order. -->

| Date | GDD A | GDD B | Conflict Type | Status |
|------|-------|-------|---------------|--------|
```

然后追加新的冲突条目。永远不要跳过日志记录 —— 缺少文件不是丢失冲突历史的理由。

---

## 阶段 7：会话状态和结束

静默追加到 `production/session-state/active.md`（如果文件不存在则创建）：

```
<!-- CONSISTENCY-CHECK: [date] | GDDs checked: [N] | Conflicts found: [N] | Report: docs/consistency-report-[date].md -->
```

然后使用 `AskUserQuestion` 小部件结束：

- **提示**："一致性检查完成 —— 发现 [N] 个冲突。下一步？"
- **选项**：
  - `[A] 现在修复最高优先级冲突`
  - `[B] 保存完整报告并停止`
  - `[C] 对冲突最多的 GDD 运行 /design-review`
  - `[D] 在此停止`

永远不要用纯文本结束 skill。始终用此小部件结束。

---

## 恢复 / 参考

- **如果 PASS**：运行 `/review-all-gdds` 进行整体设计理论审查，或
  如果所有 MVP GDD 都已完成，运行 `/create-architecture`。
- **如果 CONFLICTS FOUND**：修复标记的 GDD，然后重新运行
  `/consistency-check` 确认解决。
- **如果 STALE REGISTRY**：更新注册表（阶段 6），然后重新运行验证。
- 写入每个新 GDD 后运行 `/consistency-check` 以尽早捕获问题，
  而不是在架构时。
