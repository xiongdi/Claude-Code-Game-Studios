# 技能测试规格: /create-architecture

## 技能摘要

`/create-architecture` 引导用户逐章节编写技术架构文档。它使用 skeleton-first 方法——文件先创建包含所有必需章节标题，然后任何内容才填充。每个章节在用户批准后单独讨论、起草和写入。如果架构文档已存在，该 skill 提供改造模式以更新特定章节。

在 `full` 审查模式下，TD-ARCHITECTURE（technical-director）和 LP-FEASIBILITY（lead-programmer）在完成草案后派生。在 `lean` 或 `solo` 模式下，两个 gate 都被跳过。该 skill 写入 `docs/architecture/architecture.md`。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：APPROVED、NEEDS REVISION、MAJOR REVISION NEEDED
- [ ] 包含 "May I write" 协作协议语言（逐章节批准）
- [ ] 末尾有下一步交接说明（`/architecture-review` 或 `/create-control-manifest`）
- [ ] 记录 skeleton-first 方法
- [ ] 记录 gate 行为：full 模式下为 TD-ARCHITECTURE + LP-FEASIBILITY；lean/solo 下被跳过
- [ ] 记录现有架构文档的改造模式

---

## 导演门控检查

在 `full` 模式下：TD-ARCHITECTURE（technical-director）和 LP-FEASIBILITY
（lead-programmer）在所有章节起草后、任何最终批准写入前并行派生。

在 `lean` 模式下：两个 gate 都被跳过。输出注明：
"TD-ARCHITECTURE skipped — lean mode" 和 "LP-FEASIBILITY skipped — lean mode"。

在 `solo` 模式下：两个 gate 都被跳过，并附有同等说明。

---

## 测试用例

### Case 1: Happy Path——新架构文档，skeleton-first，full 模式 gate 批准

**Fixture:**
- 不存在现有 `docs/architecture/architecture.md`
- `docs/architecture/` 包含供参考的 Accepted ADR
- `production/session-state/review-mode.txt` 内容为 `full`

**Input:** `/create-architecture`

**Expected behavior:**
1. Skill 创建骨架 `docs/architecture/architecture.md`，包含所有必需章节标题
2. 对于每个章节：起草内容、显示草案、询问 "May I write [section]?"、批准后写入
3. 所有章节起草后：TD-ARCHITECTURE 和 LP-FEASIBILITY 并行派生
4. 两个 gate 都返回 APPROVED
5. 询问最终 "May I confirm architecture is complete?"
6. 会话状态更新

**Assertions:**
- [ ] 骨架文件先创建，包含所有章节标题，在任何内容写入之前
- [ ] 在编写过程中逐章节询问 "May I write [section]?"
- [ ] TD-ARCHITECTURE 和 LP-FEASIBILITY 并行派生（非顺序）
- [ ] 两个 gate 在完成最终确认前完成
- [ ] 当两个 gate 都返回 APPROVED 时，裁定为 APPROVED
- [ ] 存在到 `/architecture-review` 或 `/create-control-manifest` 的下一步交接说明

---

### Case 2: Failure Path——TD-ARCHITECTURE 返回 MAJOR REVISION

**Fixture:**
- 架构文档完全起草（所有章节）
- `production/session-state/review-mode.txt` 内容为 `full`
- TD-ARCHITECTURE gate 返回 MAJOR REVISION："[specific structural issue]"

**Input:** `/create-architecture`

**Expected behavior:**
1. 所有章节起草并写入
2. TD-ARCHITECTURE gate 运行并返回带有具体反馈的 MAJOR REVISION
3. Skill 向用户展示反馈
4. 架构不标记为最终确定
5. 询问用户：修改标记的章节，或接受文档为草案

**Assertions:**
- [ ] 当 TD-ARCHITECTURE 返回 MAJOR REVISION 时架构不标记为最终确定
- [ ] Gate 反馈显示给用户，附带具体问题描述
- [ ] 用户被给予修改特定章节的选项
- [ ] Skill 尽管有 MAJOR REVISION 反馈不自动最终确定

---

### Case 3: Lean Mode——两个 gate 都被跳过；架构仅凭用户批准写入

**Fixture:**
- 不存在现有架构文档
- `production/session-state/review-mode.txt` 内容为 `lean`

**Input:** `/create-architecture`

**Expected behavior:**
1. 骨架文件创建
2. 所有章节编写完成，逐章节凭用户批准写入
3. 完成后：TD-ARCHITECTURE 和 LP-FEASIBILITY 被跳过
4. 输出注明："TD-ARCHITECTURE skipped — lean mode" 和 "LP-FEASIBILITY skipped — lean mode"
5. 架构基于用户批准单独被视为完成

**Assertions:**
- [ ] 两个 gate 跳过说明都出现在输出中
- [ ] 架构文档在 lean 模式下仅凭用户批准写入
- [ ] Skill 不因 gate 被跳过而阻塞完成
- [ ] 下一步交接说明仍然存在

---

### Case 4: Retrofit Mode——现有架构文档，用户更新一个章节

**Fixture:**
- `docs/architecture/architecture.md` 已存在，所有章节已填写

**Input:** `/create-architecture`

**Expected behavior:**
1. Skill 检测现有架构文档并读取其当前内容
2. Skill 提供改造模式："Architecture doc already exists. Which section would you like to update?"
3. 用户选择一个章节
4. Skill 仅编写该章节，询问 "May I write [section]?"
5. 仅所选章节更新——其他章节不变

**Assertions:**
- [ ] Skill 在提供改造前检测并读取现有架构文档
- [ ] 用户被询问要更新哪个章节——不是被要求重写整个文档
- [ ] 仅所选章节更新
- [ ] 在改造会话期间其他章节不修改

---

### Case 5: Director Gate——架构引用 Proposed ADR；标记为风险

**Fixture:**
- 正在编写架构文档
- 一个章节引用或依赖于状态为 `Status: Proposed` 的 ADR
- `production/session-state/review-mode.txt` 内容为 `full`

**Input:** `/create-architecture`

**Expected behavior:**
1. Skill 编写所有章节
2. 在编写过程中，skill 检测到对 Proposed ADR 的引用
3. Skill 标记："Note: [section] references ADR-NNN which is Proposed — this is a risk until the ADR is accepted"
4. 风险标记嵌入相关章节的内容中
5. TD-ARCHITECTURE 和 LP-FEASIBILITY 仍然运行——它们被告知 Proposed ADR 风险

**Assertions:**
- [ ] Proposed ADR 引用在章节编写期间被检测和标记
- [ ] 风险说明嵌入架构文档章节中
- [ ] TD-ARCHITECTURE 和 LP-FEASIBILITY 仍然派生（风险不阻塞 gate）
- [ ] 风险标记命名具体的 ADR 编号和标题

---

## 协议合规

- [ ] 骨架文件先创建，包含所有章节标题，在任何内容写入之前
- [ ] 在编写过程中逐章节询问 "May I write [section]?"
- [ ] TD-ARCHITECTURE 和 LP-FEASIBILITY 在 full 模式下并行派生
- [ ] 被跳过的 gate 在 lean/solo 输出中按名称和模式注明
- [ ] Proposed ADR 引用在文档中标记为风险
- [ ] 以下一步交接说明结束：`/architecture-review` 或 `/create-control-manifest`

---

## 覆盖说明

- 架构文档的必需章节列表在 skill 正文和 `/architecture-review` skill 中定义——不在此重新列举。
- 架构文档中的引擎版本戳记（类似于 ADR 戳记）是编写工作流的一部分——通过 Case 1 隐式测试。
- 在一次会话中更新多个章节的改造模式遵循相同的逐章节批准模式——不单独对多章节改造进行测试。
