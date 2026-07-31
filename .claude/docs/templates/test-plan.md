# QA计划：[Sprint/功能名称]

> **日期**: [date]
> **生成者**: /qa-plan
> **范围**: [N个故事跨N个系统]
> **引擎**: [引擎名称和版本]
> **Sprint文件**: [path to sprint plan]

---

## 故事覆盖率摘要

| 故事 | 类型 | 需要自动化测试 | 需要手动验证 |
|-------|------|------------------------|------------------------------|
| [故事标题] | Logic | 单元测试 — `tests/unit/[system]/` | 无 |
| [故事标题] | Integration | 集成测试 — `tests/integration/[system]/` | 冒烟检查 |
| [故事标题] | Visual/Feel | 无（无法自动化） | 截图+负责人签字 |
| [故事标题] | UI | 无（无法自动化） | 手动步骤演练 |
| [故事标题] | Config/Data | 数据验证（可选） | 抽查游戏内值 |

**总计**: [N] Logic, [N] Integration, [N] Visual/Feel, [N] UI, [N] Config/Data

---

## 需要自动化测试

### [故事标题] — logic

**测试文件路径**: `tests/unit/[system]/[story-slug]_test.[ext]`

**要测试的内容**:
- [来自GDD公式部分的公式或规则 — 例如"damage = base * multiplier where multiplier ∈ [0.5, 3.0]"]
- [每个命名状态转换]
- [每个应该/不应该发生的副作用]

**要覆盖的边缘情况**:
- 零/最小输入值
- 最大/边界输入值
- 无效或空输入
- [GDD指定的边缘情况]

**估计测试数量**: 约[N]个单元测试

---

### [故事标题] — integration

**测试文件路径**: `tests/integration/[system]/[story-slug]_test.[ext]`

**要测试的内容**:
- [跨系统交互 — 例如"应用buff更新CharacterStats并触发UI刷新"]
- [往返 — 例如"保存→加载恢复所有字段"]

---

## 手动QA检查清单

### [故事标题] — Visual/Feel

**验证方法**: 截图 + [设计师/美术负责人]签字
**证据文件**: `production/qa/evidence/[story-slug]-evidence.md`
**谁必须签字**: [设计师/ lead-programmer/美术负责人]

- [ ] [特定可观察条件 — 例如"命中闪烁出现在撞击帧，而不是之后一帧"]
- [ ] [另一个可 falsify 的条件]

### [故事标题] — UI

**验证方法**: 手动步骤演练
**证据文件**: `production/qa/evidence/[story-slug]-evidence.md`

- [ ] [将每个验收标准转化为手动检查项]

---

## 冒烟测试范围

QA交接前要验证的关键路径（通过 `/smoke-check` 运行）:

1. 游戏启动到主菜单无崩溃
2. 可以开始新游戏/会话
3. [本sprint引入或改变的主要机制]
4. [有本sprint更改回归风险的系统]
5. 保存/加载周期完成无数据丢失（如果保存系统存在）
6. 性能在目标硬件上在预算内

---

## 测试需求

| 故事 | 测试目标 | 最少会话数 | 目标玩家类型 |
|-------|--------------|--------------|-------------------|
| [故事] | [必须回答什么问题？] | [N] | [新玩家/有经验的/等] |

签字要求：测试笔记 → `production/session-logs/playtest-[sprint]-[story-slug].md`

如果没有测试会话要求：*本sprint不需要测试会话。*

---

## 完成定义 — 本Sprint

当以下所有都为真时，故事为DONE：

- [ ] 所有验收标准已验证 — 自动化测试结果或文档化的手动证据
- [ ] 所有Logic和Integration故事存在测试文件且通过
- [ ] 所有Visual/Feel和UI故事存在手动证据文档
- [ ] 冒烟测试通过（QA交接前运行 `/smoke-check sprint`）
- [ ] 无回归引入 — 上一sprint的功能仍然通过
- [ ] 代码已审查（通过 `/code-review` 或文档化的同行审查）
- [ ] 故事文件通过 `/story-done` 更新为 `Status: Complete`

**关闭前需要测试签字的故事**: [列表，或"无"]

---

---

*结果和签字在 `/team-qa` 生成的 QA 签字报告中追踪 — 不在此计划文件中。*

*模板: `.claude/docs/templates/test-plan.md`*
*由: `/qa-plan` 生成 — 不要编辑此行*
