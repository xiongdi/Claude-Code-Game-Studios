# 架构可追溯性索引

<!-- 活文档 — 每次审查运行后由 /architecture-review 更新。
     不要手动编辑，除非纠正错误。 -->

## 文档状态

- **最后更新**: [YYYY-MM-DD]
- **引擎**: [例如 Godot 4.6]
- **已索引GDD**: [N]
- **已索引ADR**: [M]
- **最后审查**: [链接到 docs/architecture/architecture-review-[date].md]

## 覆盖率摘要

| 状态 | 数量 | 百分比 |
|--------|-------|-----------|
| ✅ 已覆盖 | [X] | [%] |
| ⚠️ 部分 | [Y] | [%] |
| ❌ 空白 | [Z] | [%] |
| **总计** | **[N]** | |

---

## 可追溯性矩阵

<!-- 每个从GDD提取的技术要求一行。
     "技术要求"是任何暗示特定架构决策的GDD陈述：
     数据结构、性能约束、需要引擎能力、跨系统通信、状态持久化。 -->

| 要求ID | GDD | 系统 | 要求摘要 | ADR(s) | 状态 | 备注 |
|--------|-----|--------|---------------------|--------|--------|-------|
| TR-[gdd]-001 | [filename] | [system name] | [one-line summary] | [ADR-NNNN] | ✅ | |
| TR-[gdd]-002 | [filename] | [system name] | [one-line summary] | — | ❌ GAP | 需要 `/architecture-decision [title]` |

---

## 已知空白

按层级优先级排序（Foundation优先）的要求，无ADR覆盖：

### Foundation层空白（阻塞 — 编码前必须解决）
- [ ] TR-[id]: [requirement] — GDD: [file] — 建议ADR: "[title]"

### Core层空白（相关系统构建前必须解决）
- [ ] TR-[id]: [requirement] — GDD: [file] — 建议ADR: "[title]"

### Feature层空白（功能sprint前应该解决）
- [ ] TR-[id]: [requirement] — GDD: [file] — 建议ADR: "[title]"

### Presentation层空白（可以推迟到实施）
- [ ] TR-[id]: [requirement] — GDD: [file] — 建议ADR: "[title]"

---

## 跨ADR冲突

<!-- 做出矛盾声明的ADR对。必须解决。 -->

| 冲突ID | ADR A | ADR B | 类型 | 状态 |
|-------------|-------|-------|------|--------|
| CONFLICT-001 | ADR-NNNN | ADR-MMMM | 数据所有权 | 🔴 未解决 |

---

## ADR → GDD 覆盖率（反向索引）

<!-- 对于每个ADR，它解决了哪些GDD要求？ -->

| ADR | 标题 | 解决的GDD要求 | 引擎风险 |
|-----|-------|---------------------------|-------------|
| ADR-0001 | [title] | TR-combat-001, TR-combat-002 | 高 |

---

## 被取代的要求

<!-- 当编写ADR时GDD中存在的需求，但GDD后来发生了变化。
     ADR可能需要更新。 -->

| 要求ID | GDD | 更改 | 受影响的ADR | 状态 |
|--------|-----|--------|-------------|--------|
| TR-[id] | [file] | [what changed] | ADR-NNNN | 🔴 ADR需要更新 |

---

## 如何使用本文档

**编写新ADR时**: 将其添加到"ADR → GDD覆盖率"表，并将满足的要求标记为✅。

**批准GDD更改时**: 扫描该GDD要求的矩阵，检查更改是否使任何现有ADR失效。如果是，添加到"被取代的要求"。

**运行 `/architecture-review` 时**: 该技能将自动更新本文档的当前状态。

**Gate检查**: 预生产gate要求本文档存在且Foundation层空白为零。
