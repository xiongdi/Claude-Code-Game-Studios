---
name: content-audit
description: "Audit GDD-specified content counts against implemented content. Identifies what's planned vs built."
argument-hint: "[system-name | --summary | (no arg = full audit)]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write
model: sonnet
agent: producer
---

当此 skill 被调用时：

解析参数：
- 无参数 → 跨所有系统的完整审计
- `[system-name]` → 仅审计那个单一系统
- `--summary` → 仅摘要表，不写入文件

---

## 阶段 1 —— 上下文收集

1. **读取 `design/gdd/systems-index.md`** 以获取系统的完整列表、它们的
   类别和 MVP/优先级层级。

2. **L0 预扫描**：在完整读取任何 GDD 之前，Grep 所有 GDD 文件以获取
   `## Summary` 部分以及常见的内容计数关键词：
   ```
   Grep pattern="(## Summary|N enemies|N levels|N items|N abilities|enemy types|item types)" glob="design/gdd/*.md" output_mode="files_with_matches"
   ```
   对于单一系统审计：跳过此步骤并直接进行完整读取。
   对于完整审计：仅完整读取匹配内容计数关键词的 GDD。
   没有内容计数语言的 GDD（纯机制 GDD）被记录为
   "No auditable content counts"而无需完整读取。

3. **完整读取范围内的 GDD 文件**（或如果给定系统名称则为单一系统 GDD）。

4. **对于每个 GDD，提取明确的内容计数或列表。** 查找模式
   如：
   - "N enemies" / "enemy types:" / 命名敌人的列表
   - "N levels" / "N areas" / "N maps" / "N stages"
   - "N items" / "N weapons" / "N equipment pieces"
   - "N abilities" / "N skills" / "N spells"
   - "N dialogue scenes" / "N conversations" / "N cutscenes"
   - "N quests" / "N missions" / "N objectives"
   - 任何明确的枚举列表（命名内容项的子弹列表）

4. **从提取的数据构建内容清单表**：

   | System | Content Type | Specified Count/List | Source GDD |
   |--------|-------------|---------------------|------------|

   注意：如果 GDD 定性地描述内容但没有给出计数，记录
   "Unspecified" 并标记它 —— 未指定的计数是值得注意的设计缺口。

---

## 阶段 2 —— 实现扫描

对于阶段 1 中找到的每种内容类型，扫描相关目录以计数
已实现的内容。使用 Glob 和 Grep 来定位文件。

**关卡/区域/地图：**
- Glob `assets/**/*.tscn`、`assets/**/*.unity`、`assets/**/*.umap`
- Glob `src/**/*.tscn`、`src/**/*.unity`
- 在名为 `levels/`、`areas/`、`maps/`、
  `worlds/`、`stages/` 的子目录中查找场景文件
- 计数似乎是关卡/场景定义的独特文件（不是 UI 场景）

**敌人/角色/NPC：**
- Glob `assets/data/**/enemies/**`、`assets/data/**/characters/**`
- Glob `src/**/enemies/**`、`src/**/characters/**`
- 查找定义实体属性的 `.json`、`.tres`、`.asset`、`.yaml` 数据文件
- 查找角色子目录中的场景/预制体文件

**物品/装备/战利品：**
- Glob `assets/data/**/items/**`、`assets/data/**/equipment/**`、
  `assets/data/**/loot/**`
- 查找 `.json`、`.tres`、`.asset` 数据文件

**能力/技能/法术：**
- Glob `assets/data/**/abilities/**`、`assets/data/**/skills/**`、
  `assets/data/**/spells/**`
- 查找 `.json`、`.tres`、`.asset` 数据文件

**对话/对话/过场动画：**
- Glob `assets/**/*.dialogue`、`assets/**/*.csv`、`assets/**/*.ink`
- Grep `assets/data/` 中的对话数据文件

**任务/使命：**
- Glob `assets/data/**/quests/**`、`assets/data/**/missions/**`
- 查找 `.json`、`.yaml` 定义文件

**引擎特定说明（在报告中确认）：**
- 计数是近似值 —— skill 无法完美解析每种引擎
  格式或区分编辑器专用文件与发布内容
- 场景文件可能同时包含游戏内容和系统/UI 场景；扫描
  计数所有匹配项并注意此警告

---

## 阶段 3 —— 缺口报告

生成缺口表：

```
| System | Content Type | Specified | Found | Gap | Status |
|--------|-------------|-----------|-------|-----|--------|
```

**状态类别：**
- `COMPLETE` — Found ≥ Specified（100%+）
- `IN PROGRESS` — Found 是 Specified 的 50–99%
- `EARLY` — Found 是 Specified 的 1–49%
- `NOT STARTED` — Found 为 0

**优先级标志：**
如果以下情况，在报告中将系统标记为 `HIGH PRIORITY`：
- 状态为 `NOT STARTED` 或 `EARLY`，且
- 系统在系统索引中标记为 MVP 或 Vertical Slice，或
- 系统索引显示系统正在阻塞下游系统

**摘要行：**
- 指定的内容项总数（所有 Specified 列值的总和）
- 找到的内容项总数（所有 Found 列值的总和）
- 整体缺口百分比：`(Specified - Found) / Specified * 100`

---

## 阶段 4 —— 输出

### 完整审计和单一系统模式

向用户展示缺口表和摘要。询问："我可以将完整报告写入 `docs/content-audit-[YYYY-MM-DD].md` 吗？"

如果是，写入文件：

```markdown
# Content Audit — [Date]

## Summary
- **Total specified**: [N] content items across [M] systems
- **Total found**: [N]
- **Gap**: [N] items ([X%] unimplemented)
- **Scope**: [Full audit | System: name]

> Note: Counts are approximations based on file scanning.
> The audit cannot distinguish shipped content from editor/test assets.
> Manual verification is recommended for any HIGH PRIORITY gaps.

## Gap Table

| System | Content Type | Specified | Found | Gap | Status |
|--------|-------------|-----------|-------|-----|--------|

## HIGH PRIORITY Gaps

[List systems flagged HIGH PRIORITY with rationale]

## Per-System Breakdown

### [System Name]
- **GDD**: `design/gdd/[file].md`
- **Content types audited**: [list]
- **Notes**: [any caveats about scan accuracy for this system]

## Recommendation

Focus implementation effort on:
1. [Highest-gap HIGH PRIORITY system]
2. [Second system]
3. [Third system]

## Unspecified Content Counts

The following GDDs describe content without giving explicit counts.
Consider adding counts to improve auditability:
[List of GDDs and content types with "Unspecified"]
```

写入报告后，询问：

> "你希望为任何内容缺口创建积压故事吗？"

如果是：对于用户选择的每个系统，建议一个故事标题并指向
`/create-stories [epic-slug]` 或 `/quick-design`，取决于缺口的大小。

### --summary 模式

将缺口表和摘要直接打印到对话中。不要写入文件。
以："运行不带 `--summary` 的 `/content-audit` 以写入完整报告。"结束。

---

## 阶段 5 —— 下一步

审计后，推荐最高价值的后续操作：

- 如果任何系统是 `NOT STARTED` 且标记为 MVP → "运行 `/design-system [name]` 在实现开始之前向 GDD 添加缺失的内容计数。"
- 如果总缺口 >50% → "运行 `/sprint-plan` 在即将到来的 sprint 中分配内容工作。"
- 如果需要积压故事 → "为每个 HIGH PRIORITY 缺口运行 `/create-stories [epic-slug]`。"
- 如果使用了 `--summary` → "运行 `/content-audit`（无标志）将完整报告写入 `docs/`。"

裁决：**COMPLETE** —— 内容审计完成。
