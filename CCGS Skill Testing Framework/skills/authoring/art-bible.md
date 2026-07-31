# Skill Test Spec: /art-bible

## Skill Summary

`/art-bible` 是一个引导式逐章节 art bible 编写 skill。它生成涵盖以下内容的全面视觉方向文档：Visual Style 概述、Color Palette、Typography、Character Design Rules、Environment Style 和 UI Visual Language。该 skill 遵循 skeleton-first 模式：立即创建包含所有章节标题的文件，然后通过讨论填写每个章节，并在用户批准后逐个写入磁盘。

在 `full` 审查模式下，AD-ART-BIBLE director gate（art director）在草案完成后、任何章节写入前运行。在 `lean` 和 `solo` 模式下，AD-ART-BIBLE 被跳过，仅需要用户批准。当所有章节写入后裁定为 COMPLETE。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：COMPLETE
- [ ] 包含逐章节的 "May I write" 语言
- [ ] 记录 AD-ART-BIBLE director gate 及其模式行为
- [ ] 具有下一步交接说明（例如，`/asset-spec` 或 `/design-system`）

---

## Director Gate Checks

| Gate ID      | 触发条件                       | 模式保护                    |
|--------------|--------------------------------|-----------------------|
| AD-ART-BIBLE | 草案完成后                     | 仅 full（非 lean/solo）     |

---

## Test Cases

### Case 1: Happy Path——full 模式，art bible 起草完成，AD-ART-BIBLE 批准

**Fixture:**
- 不存在现有 `design/art-bible.md`
- `production/session-state/review-mode.txt` 内容为 `full`
- `design/gdd/game-concept.md` 存在，描述了视觉色调

**Input:** `/art-bible`

**Expected behavior:**
1. Skill 创建骨架 `design/art-bible.md`，包含所有章节标题
2. Skill 与用户协作讨论和起草每个章节
3. 所有章节起草后，调用 AD-ART-BIBLE gate（art director 审查）
4. AD-ART-BIBLE 返回 APPROVED
5. Skill 逐章节询问 "May I write section [N] to `design/art-bible.md`?"
6. 所有章节批准后写入；裁定为 COMPLETE

**Assertions:**
- [ ] 骨架文件首先创建（在任何章节内容写入之前）
- [ ] AD-ART-BIBLE gate 在 full 模式下草案完成后调用
- [ ] Gate 批准在 "May I write" 章节请求之前
- [ ] 最终文件中存在所有章节
- [ ] 裁定为 COMPLETE

---

### Case 2: AD-ART-BIBLE Returns CONCERNS——章节在写入前修订

**Fixture:**
- Art bible 草案完成
- `production/session-state/review-mode.txt` 内容为 `full`
- AD-ART-BIBLE gate 返回 CONCERNS："Color palette clashes with the dark
  atmospheric tone described in the game concept"

**Input:** `/art-bible`

**Expected behavior:**
1. AD-ART-BIBLE gate 返回带有具体调色板反馈的 CONCERNS
2. Skill 向用户展示反馈："Art director has concerns about the color palette"
3. Skill 返回 Color Palette 章节进行修改
4. 用户和 skill 修改调色板以与游戏概念色调对齐
5. AD-ART-BIBLE 不被重新调用（用户在修改后决定继续）
6. 修订后的章节在 "May I write" 批准后写入；裁定为 COMPLETE

**Assertions:**
- [ ] CONCERNS 在任何章节写入前显示给用户
- [ ] Skill 返回受影响的章节进行修改（不是所有章节）
- [ ] 写入修订后的内容（不是原始内容）
- [ ] 修改和批准后裁定为 COMPLETE

---

### Case 3: Lean Mode——AD-ART-BIBLE 被跳过，仅凭用户批准写入

**Fixture:**
- 不存在现有 art bible
- `production/session-state/review-mode.txt` 内容为 `lean`

**Input:** `/art-bible`

**Expected behavior:**
1. Skill 读取审查模式——确定为 `lean`
2. Skill 与用户协作起草所有章节
3. AD-ART-BIBLE gate 被跳过：输出注明 "[AD-ART-BIBLE] skipped — lean mode"
4. Skill 请求用户直接批准每个章节
5. 章节在用户确认后写入；裁定为 COMPLETE

**Assertions:**
- [ ] AD-ART-BIBLE gate 在 lean 模式下不被调用
- [ ] 跳过被明确注明："[AD-ART-BIBLE] skipped — lean mode"
- [ ] 仍需要用户逐章节批准（gate 跳过 ≠ 批准跳过）
- [ ] 裁定为 COMPLETE

---

### Case 4: Existing Art Bible——Retrofit Mode

**Fixture:**
- `design/art-bible.md` 已存在，所有章节已填写
- 用户想更新 Character Design Rules 章节

**Input:** `/art-bible`

**Expected behavior:**
1. Skill 读取现有 art bible 并检测所有章节已填写
2. Skill 提供改造："Art bible exists — which section would you like to update?"
3. 用户选择 Character Design Rules
4. Skill 起草更新内容；在 full 模式下，AD-ART-BIBLE 在写入前为修订后的章节调用
5. Skill 询问 "May I write Character Design Rules to `design/art-bible.md`?"
6. 仅该章节更新；其他章节保留；裁定为 COMPLETE

**Assertions:**
- [ ] 检测现有 art bible 并提供改造选项
- [ ] 仅所选章节更新
- [ ] 在 full 模式下：即使对于单章节改造，AD-ART-BIBLE gate 也运行
- [ ] 其他章节保留
- [ ] 裁定为 COMPLETE

---

### Case 5: Solo Mode——AD-ART-BIBLE 被跳过，在输出中注明

**Fixture:**
- 不存在现有 art bible
- `production/session-state/review-mode.txt` 内容为 `solo`

**Input:** `/art-bible`

**Expected behavior:**
1. Skill 读取审查模式——确定为 `solo`
2. Art bible 起草完成，仅凭用户批准写入
3. AD-ART-BIBLE gate 被跳过：输出注明 "[AD-ART-BIBLE] skipped — solo mode"
4. 不派生 director agent
5. 裁定为 COMPLETE

**Assertions:**
- [ ] AD-ART-BIBLE gate 在 solo 模式下不被调用
- [ ] 跳过被明确注明，带有 "solo mode" 标签
- [ ] 不派生任何类型的 director agent
- [ ] 裁定为 COMPLETE

---

## Protocol Compliance

- [ ] 立即创建骨架文件，包含所有章节标题
- [ ] 一次讨论和起草一个章节
- [ ] AD-ART-BIBLE gate 在 full 模式下所有章节起草后运行
- [ ] AD-ART-BIBLE 在 lean 和 solo 模式下被跳过——按名称注明
- [ ] 逐章节询问 "May I write section [N]"
- [ ] 当所有章节写入后裁定为 COMPLETE

---

## Coverage Notes

- AD-ART-BIBLE 返回 REJECT（不仅仅是 CONCERNS）的情况不单独测试；
  skill 会阻塞写入并询问用户如何继续（修改或覆盖）。
- Typography 章节被列为必需的 art bible 章节，但其具体内容要求不在此进行断言测试。
- Art bible 输入到 `/asset-spec`——此关系在交接说明中注明但不作为此 skill spec 的一部分测试。
