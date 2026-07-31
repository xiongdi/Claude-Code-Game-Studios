---
name: propagate-design-change
description: "When a GDD is revised, scans all ADRs and the traceability index to identify which architectural decisions are now potentially stale. Produces a change impact report and guides the user through resolution."
argument-hint: "[path/to/changed-gdd.md]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Bash, Task
model: sonnet
agent: technical-director
---

# 传播设计变更

当 GDD 变更时，基于它编写的架构决策可能不再有效。此 skill 找出每个受影响的 ADR，将 ADR 假设的内容与 GDD 现在的内容进行比较，并引导用户完成解决。

**用法：** `/propagate-design-change design/gdd/combat-system.md`

---

## 1. 验证参数

GDD 路径参数是**必需的**。如果缺失，失败并显示：
> "用法：`/propagate-design-change design/gdd/[system].md`
> 提供已更改的 GDD 的路径。"

验证文件是否存在。如果不存在，失败并显示：
> "[路径] 未找到。检查路径并重试。"

---

## 2. 读取已更改的 GDD

完整读取当前 GDD。

---

## 3. 读取上一个版本

运行 git 获取上一个已提交的版本：

```bash
git show HEAD:design/gdd/[filename].md
```

如果文件没有 git 历史（新文件），报告：
> "git 中没有上一个版本 — 这似乎是一个新 GDD，而不是修订。
> 无需传播。"

如果 git 返回上一个版本，进行概念性 diff：
- 识别已更改的部分（新规则、删除的规则、修改的公式、
  变更的验收标准、变更的调优旋钮）
- 识别未更改的部分
- 生成变更摘要：

```
## 变更摘要: [GDD 文件名]
修订日期: [今天]

已更改的部分:
- [部分名称]: [发生了什么变更 — 新规则、删除的规则、公式修改等]

未更改的部分:
- [部分名称]

影响架构的关键变更:
- [变更 1 — 可能影响 ADR]
- [变更 2]
```

---

## 4. 加载架构输入

读取 `docs/architecture/` 中的所有 ADR：
- 对于每个 ADR，读取完整文件
- 提取"GDD Requirements Addressed"表格
- 记录每个 ADR 引用了哪些 GDD 文档和需求 ID

如果存在，读取 `docs/architecture/architecture-traceability.md`。

报告："已加载 [N] 个 ADR。[M] 个引用了 [gdd 文件名]。"

---

## 5. 影响分析

对于每个引用了已更改 GDD 的 ADR：

将 ADR 的"GDD Requirements Addressed"条目与 GDD 的已更改部分进行比较。对于每个引用的需求：

1. **定位需求** — 在当前 GDD 中是否仍然存在？
2. **比较**: ADR 编写时 GDD 说了什么 vs 现在说了什么？
3. **评估 ADR 决策**: 架构决策是否仍然有效？

将每个受影响的 ADR 分类为：

| 状态 | 含义 |
|--------|---------|
| ✅ **仍然有效** | GDD 变更不影响此 ADR 决定的内容 |
| ⚠️ **需要审查** | GDD 变更可能影响此 ADR — 需要人工判断 |
| 🔴 **可能已取代** | GDD 变更直接与此 ADR 假设的内容矛盾 |

对于每个受影响的 ADR，生成一个影响条目：

```
### ADR-NNNN: [标题]
状态: [仍然有效 / 需要审查 / 可能已取代]

此 ADR 对此 GDD 的假设:
  "[来自 ADR 的 GDD Requirements Addressed 部分的相关引用]"

GDD 现在的内容:
  "[来自当前 GDD 的相关引用]"

评估:
  [解释 ADR 决策是否仍然有效及其原因]

推荐操作:
  [保持原样 | 审查并更新 | 标记为已取代并编写新 ADR]
```

---

## 6. 展示影响报告

在请求任何操作之前，向用户展示完整的影响报告。格式：

```
## 设计变更影响报告
GDD: [文件名]
日期: [今天]
检测到的变更: [N 个部分已更改]
引用此 GDD 的 ADR: [M]

### 不受影响
[引用此 GDD 但其决策仍然有效的 ADR]

### 需要审查 ([数量])
[可能需要更新的 ADR]

### 可能已取代 ([数量])
[其假设现在被矛盾的 ADR]
```

---

## 6b. Director Gate — 技术影响审查

**审查模式检查** — 在 spawn TD-CHANGE-IMPACT 之前应用：
- `solo` → 跳过。注意："TD-CHANGE-IMPACT 已跳过 — Solo 模式。" 继续到 Phase 7。
- `lean` → 跳过。注意："TD-CHANGE-IMPACT 已跳过 — Lean 模式。" 继续到 Phase 7。
- `full` → 正常 spawn。

使用 gate **TD-CHANGE-IMPACT**（`.claude/docs/director-gates.md`）通过 Task spawn `technical-director`。

传递：Phase 6 的完整设计变更影响报告（变更摘要、所有受影响的 ADR 及其仍然有效/需要审查/可能已取代的分类，以及推荐操作）。

technical-director 审查：
- 影响分类是否正确（没有 ADR 被低估分类）
- 推荐操作在架构上是否合理
- 是否遗漏了对其他 ADR 或系统的任何级联效应

应用判定：
- **APPROVE** → 继续到 Phase 7 解决工作流
- **CONCERNS** → 展示被标记的具体 ADR 或推荐；使用 `AskUserQuestion` 提供选项：`修订影响评估` / `接受并记录关注点` / `进一步讨论`
- **REJECT** → 不继续到解决；在继续前重新分析影响

---

## 7. 解决工作流

对于每个标记为"需要审查"或"可能已取代"的 ADR，询问用户要做什么：

逐个询问每个 ADR：
> "ADR-NNNN ([标题]) — [状态]。您想做什么？"
> 选项：
> - "标记为已取代（我将编写新 ADR）" — 将 ADR 状态行更新为 `Superseded by: [pending]`
> - "就地更新（小修订）" — 打开 ADR 进行编辑；注明要修订的内容
> - "保持原样（变更实际上不影响此决策）"
> - "暂时跳过（稍后处理）"

对于标记为**已取代**的 ADR：
- 更新 ADR 的状态字段：`Superseded by ADR-[下一个编号]（pending — 见 change-impact-[日期]-[系统].md）`
- 询问："我可以更新 [ADR 文件名] 中的状态吗？"

---

## 8. 更新可追溯性索引

如果 `docs/architecture/architecture-traceability.md` 存在：
- 将已更改的 GDD 需求添加到"Superseded Requirements"表格：

```markdown
## Superseded Requirements
| 日期 | GDD | 需求 | 变更为 | 受影响的 ADR | 解决方案 |
|------|-----|-------------|------------|---------------|------------|
| [日期] | [gdd] | [旧需求文本] | [新需求文本] | ADR-NNNN | [已取代/已更新/有效] |
```

询问："我可以更新可追溯性索引吗？"

---

## 9. 输出变更影响文档

询问："我可以将变更影响报告写入 `docs/architecture/change-impact-[日期]-[系统-slug].md` 吗？"

文档包含：
- 步骤 3 的变更摘要
- 步骤 5 的完整影响分析
- 步骤 7 中做出的解决决策
- 需要编写或更新的 ADR 列表

如果用户批准：判定：**COMPLETE** — 变更影响报告已保存。
如果用户拒绝：判定：**BLOCKED** — 用户拒绝写入。

---

## 10. 后续操作

基于解决决策，建议：

- **标记为已取代的 ADR**："运行 `/architecture-decision [标题]` 编写替代 ADR。然后重新运行 `/propagate-design-change` 验证覆盖范围。"
- **要就地更新的 ADR**：列出每个 ADR 中要更新的具体字段
- **如果许多 ADR 受影响**："在所有 ADR 更新后运行 `/architecture-review`，验证完整的可追溯性矩阵仍然连贯。"

---

## 协作协议

1. **静默读取** — 在展示任何内容之前计算完整影响
2. **先展示完整报告** — 让用户在请求操作之前看到范围
3. **逐个 ADR 询问** — 不要批量决策；每个受影响的 ADR 可能需要不同的处理
4. **写入前询问** — 在修改任何文件之前始终确认
5. **非破坏性** — 永远不删除 ADR 内容；只添加"Superseded by"注释
