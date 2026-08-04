# 技能测试规格: /design-system

## 技能摘要

`/design-system` 引导用户逐章节编写单个游戏系统的游戏设计文档（GDD）。所有 8 个必需章节都必须编写：Overview、Player Fantasy、Detailed Rules、Formulas、Edge Cases、Dependencies、Tuning Knobs 和 Acceptance Criteria。该 skill 使用 skeleton-first 方法——在填写任何内容之前创建包含所有 8 个章节标题的 GDD 文件——并在批准后逐个写入每个章节。

CD-GDD-ALIGN gate（creative-director）在 `full` 和 `lean` 模式下都运行。它仅在 `solo` 模式下被跳过。如果找到现有 GDD 文件，该 skill 提供改造模式以更新特定章节而非重写整个文档。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：APPROVED、NEEDS REVISION、MAJOR REVISION
- [ ] 包含 "May I write" 协作协议语言（逐章节批准）
- [ ] 末尾有下一步交接说明
- [ ] 记录 skeleton-first 方法（文件先创建标题后填内容）
- [ ] 记录 CD-GDD-ALIGN gate：在 full 和 lean 模式下活动；仅在 solo 下跳过
- [ ] 记录现有 GDD 文件的改造模式

---

## 导演门控检查

在 `full` 模式下：CD-GDD-ALIGN（creative-director）gate 在每个章节起草后、写入前运行。如果返回 MAJOR REVISION，该章节必须重写后才能继续。

在 `lean` 模式下：CD-GDD-ALIGN 仍然运行（此 gate 在 lean 模式下不被跳过——它在 full 和 lean 下都运行）。仅 solo 模式跳过它。

在 `solo` 模式下：CD-GDD-ALIGN 被跳过。输出注明：
"CD-GDD-ALIGN skipped — solo mode"。章节仅凭用户批准写入。

---

## 测试用例

### Case 1: Happy Path——新 GDD，skeleton-first，lean 模式下的 CD-GDD-ALIGN

**Fixture:**
- `design/gdd/` 中目标系统无现有 GDD
- `production/session-state/review-mode.txt` 内容为 `lean`

**Input:** `/design-system [system-name]`

**Expected behavior:**
1. Skill 创建骨架文件 `design/gdd/[system-name].md`，包含所有 8 个章节标题（空正文）
2. 对于每个章节：与用户讨论、起草内容、显示草案
3. CD-GDD-ALIGN gate 对每个章节草案运行（lean 模式——gate 是活动的）
4. Gate 对每个章节返回 APPROVED
5. 在 gate 批准后询问 "May I write [section]?"
6. 章节在用户批准后写入文件
7. 过程对所有 8 个章节重复

**Assertions:**
- [ ] 骨架文件先创建，包含所有 8 个章节标题，在任何内容写入之前
- [ ] CD-GDD-ALIGN 在 lean 模式下对每个章节运行（不被跳过）
- [ ] 逐章节询问 "May I write"（非一次性对所有章节）
- [ ] 每个章节在 gate + 用户批准后单独写入
- [ ] 最终 GDD 文件中存在所有 8 个章节

---

### Case 2: Retrofit Mode——现有 GDD，更新特定章节

**Fixture:**
- `design/gdd/[system-name].md` 已存在，所有 8 个章节已填写

**Input:** `/design-system [system-name]`

**Expected behavior:**
1. Skill 检测现有 GDD 文件并读取其当前内容
2. Skill 提供改造模式："GDD already exists. Which section would you like to update?"
3. 用户选择特定章节（例如，Formulas）
4. Skill 仅编写该章节，运行 CD-GDD-ALIGN，询问 "May I write?"
5. 仅所选章节更新——其他章节不修改

**Assertions:**
- [ ] Skill 在提供改造模式前检测并读取现有 GDD
- [ ] 用户被询问要更新哪个章节——不是被要求重写整个文档
- [ ] 仅所选章节重写——其他章节保持不变
- [ ] CD-GDD-ALIGN 仍对更新的章节运行
- [ ] 在更新章节前询问 "May I write"

---

### Case 3: Director Gate——CD-GDD-ALIGN 返回 MAJOR REVISION

**Fixture:**
- 正在编写新 GDD
- `production/session-state/review-mode.txt` 内容为 `lean`
- CD-GDD-ALIGN gate 对 Player Fantasy 章节返回 MAJOR REVISION

**Input:** `/design-system [system-name]`

**Expected behavior:**
1. Player Fantasy 章节起草完成
2. CD-GDD-ALIGN gate 运行并返回带有具体反馈的 MAJOR REVISION
3. Skill 向用户展示反馈
4. 当 MAJOR REVISION 未解决时，章节不写入文件
5. 用户与 skill 协作重写该章节
6. CD-GDD-ALIGN 对修订后的章节再次运行
7. 如果修订后的章节通过，询问 "May I write?" 并写入章节

**Assertions:**
- [ ] 当 CD-GDD-ALIGN 返回 MAJOR REVISION 时章节不写入
- [ ] Gate 反馈在请求修订前显示给用户
- [ ] CD-GDD-ALIGN 在章节修订后再次运行
- [ ] Skill 在 MAJOR REVISION 未解决时不自动继续到下一章节

---

### Case 4: Solo Mode——CD-GDD-ALIGN 被跳过；章节仅凭用户批准写入

**Fixture:**
- 正在编写新 GDD
- `production/session-state/review-mode.txt` 内容为 `solo`

**Input:** `/design-system [system-name]`

**Expected behavior:**
1. 骨架文件创建，包含 8 个章节标题
2. 对于每个章节：起草完成，显示给用户
3. CD-GDD-ALIGN 被跳过——逐章节注明："CD-GDD-ALIGN skipped — solo mode"
4. 用户审查草案后询问 "May I write [section]?"
5. 章节在用户批准后写入
6. 在任何阶段都无 gate 审查

**Assertions:**
- [ ] 每个章节注明 "CD-GDD-ALIGN skipped — solo mode"
- [ ] 章节仅凭用户批准后写入（不需要 gate）
- [ ] Skill 在 solo 模式下不派生任何 CD-GDD-ALIGN gate
- [ ] 在 solo 模式下完整 GDD 仅凭用户批准写入

---

### Case 5: Director Gate——空章节不写入文件

**Fixture:**
- GDD 编写进行中
- 用户和 skill 讨论一个章节但未产生任何批准的内容
  （例如，讨论结束时无决定，或用户说 "skip for now"）

**Input:** `/design-system [system-name]`

**Expected behavior:**
1. 章节讨论未产生批准的内容
2. Skill 不向章节写入空或占位符正文
3. 章节标题保留在骨架文件中但正文保持为空
4. Skill 不写入空章节而移动到下一章节
5. 最后，列出不完整的章节并提醒用户返回处理

**Assertions:**
- [ ] 空或未批准的章节不写入文件
- [ ] 骨架章节标题保留（保持结构）
- [ ] Skill 在会话结束时跟踪并列出不完整的章节
- [ ] Skill 未经用户批准不写入 "TBD" 或占位符内容

---

## 协议合规

- [ ] 骨架文件先创建，包含所有 8 个标题，在任何内容写入之前
- [ ] CD-GDD-ALIGN 在 full 和 lean 模式下都运行（不仅是 full）
- [ ] CD-GDD-ALIGN 仅在 solo 模式下被跳过——逐章节注明
- [ ] 逐章节询问 "May I write [section]?"（非一次性对整个文档）
- [ ] 来自 CD-GDD-ALIGN 的 MAJOR REVISION 阻塞章节写入直到解决
- [ ] 仅批准的非空章节写入文件
- [ ] 以下一步交接说明结束：`/review-all-gdds` 或 `/map-systems next`

---

## 覆盖说明

- 8 个必需章节根据 `CLAUDE.md` 中定义的项目设计文档标准进行验证——不在此重新列举。
- Skill 内部的章节排序逻辑（先编写哪个章节）不单独测试——顺序遵循标准 GDD 模板。
- CD-GDD-ALIGN 内的 pillar 对齐检查由 gate agent 整体评估——具体的 pillar 检查不在此进行 fixture 测试。
