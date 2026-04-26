# 技能规格: /[skill-name]

> **类别**: [gate | review | authoring | readiness | pipeline | analysis | team | sprint | utility]
> **优先级**: [critical | high | medium | low]
> **规格编写**: [YYYY-MM-DD]

## 技能摘要

[一段描述此技能做什么、接受什么输入、产生什么输出的文字。]

---

## 静态断言

这些应在任何行为测试之前通过：

- [ ] Frontmatter 有所有必需字段（`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`）
- [ ] 找到2+个阶段标题
- [ ] 至少存在一个裁决关键词（`PASS`、`FAIL`、`CONCERNS`、`APPROVED`、`BLOCKED`、`COMPLETE`、`READY`）
- [ ] 如果 `allowed-tools` 包含 Write/Edit：存在"May I write"语言
- [ ] 末尾存在下一步交接部分

---

## Director Gate 检查

[描述此技能触发哪些 director gates（如果有），以及在什么审查模式条件下。]

- **Full 模式**: [触发的 gates — 如 CD-PHASE-GATE, TD-PHASE-GATE, PR-PHASE-GATE, AD-PHASE-GATE]
- **Lean 模式**: [仅阶段 gates — 如仅 CD-PHASE-GATE，或无]
- **Solo 模式**: [无 gates — 技能运行无需 director 审查]
- **N/A**: [如果此技能从不触发 gates，解释原因]

---

## 测试用例

### 案例 1: 快乐路径 — [简要名称]

**Fixture**（假设的项目状态）：
- [文件/条件 1]
- [文件/条件 2]

**预期行为**:
1. [步骤 1]
2. [步骤 2]
3. [步骤 3]

**断言**:
- [ ] [断言 1]
- [ ] [断言 2]
- [ ] [断言 3]

**案例裁决**: PASS / FAIL / PARTIAL

---

### 案例 2: 失败 / 阻塞 — [简要名称]

**Fixture**:
- [缺失或无效的条件]

**预期行为**:
1. [技能检测到问题]
2. [技能报告 FAIL/BLOCKED]
3. [技能不继续]

**断言**:
- [ ] 技能提前停止，不产生输出
- [ ] 显示正确的错误/阻塞消息
- [ ] 未经用户批准不写入文件

**案例裁决**: PASS / FAIL / PARTIAL

---

### 案例 3: 模式变体 — [简要名称]

**Fixture**:
- [标准项目状态]
- [设置特定模式或标志]

**预期行为**:
1. [行为因模式而与快乐路径不同]

**断言**:
- [ ] [模式特定断言]
- [ ] [输出与案例 1 正确不同]

**案例裁决**: PASS / FAIL / PARTIAL

---

### 案例 4: 边缘情况 — [简要名称]

**Fixture**:
- [不寻常或边界条件]

**预期行为**:
1. [技能优雅处理]

**断言**:
- [ ] [边缘情况处理无崩溃或静默失败]
- [ ] [正确的输出或消息]

**案例裁决**: PASS / FAIL / PARTIAL

---

### 案例 5: Director Gate — [简要名称]

**Fixture**:
- [触发 gate 检查的项目状态]
- 审查模式: [full | lean | solo]

**预期行为**:
1. [Gate 基于模式触发/不触发]
2. [正确派生或跳过相关 director agents]

**断言**:
- [ ] 在 full 模式：[特定 gates 派生]
- [ ] 在 lean 模式：[仅阶段 gates，或跳过]
- [ ] 在 solo 模式：不派生 director gates
- [ ] 技能在 CONCERNS 或 FAIL 裁决后不自动推进

**案例裁决**: PASS / FAIL / PARTIAL

---

## 协议合规性

- [ ] 在任何文件写入前使用"May I write"（或是只读的，跳过此项）
- [ ] 在请求批准前向用户展示发现/草稿
- [ ] 以推荐的下一步或后续操作结束
- [ ] 未经用户批准不自动创建文件

---

## 覆盖范围备注

[覆盖范围中的任何差距、已知未测试的边缘情况，或需要 live 技能运行才能验证的条件。]
