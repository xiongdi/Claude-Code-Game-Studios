# Skill 测试规格：/retrospective

## Skill 摘要

`/retrospective` 生成结构化的 sprint 或 milestone retrospective，
涵盖三个类别：什么做得好、什么做得不好和行动项。
它读取 sprint 文件和会话日志来编译观察结果，然后生成 retrospective 文档。
不使用 director 关卡——retrospective 是团队自我反思产物。
该 skill 在持久化前询问"可以写入 `production/retrospectives/retro-sprint-NNN.md` 吗？"。
判定结果始终是 COMPLETE（retrospective 是结构化输出，不是通过/失败评估）。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE
- [ ] 包含"可以写入吗"语言（skill 写入 retrospective 文档）
- [ ] 有下一步交接（retrospective 写入后做什么）

---

## Director 关卡检查

无。Retrospective 是团队自我反思文档；不调用关卡。

---

## 测试用例

### 用例 1：正常路径——混合结果的 sprint

**Fixture：**
- `production/sprints/sprint-005.md` 存在，有 6 个 story（4 Complete、1 Blocked、1 Deferred）
- `production/session-logs/` 包含 sprint 期间的日志条目
- 不存在针对 sprint-005 的先前 retrospective

**输入：** `/retrospective sprint-005`

**预期行为：**
1. Skill 读取 sprint-005 和会话日志
2. Skill 编译三个 retrospective 类别：做得好（4 个 story 已交付）、
   不好（1 个阻塞、1 个延期）和行动项（解决阻塞根本原因）
3. Skill 向用户展示 retrospective 草稿
4. Skill 询问"可以写入 `production/retrospectives/retro-sprint-005.md` 吗？"
5. 用户批准；文件写入；判定 COMPLETE

**断言：**
- [ ] Retrospective 包含所有三个类别（做得好 / 不好 / 行动）
- [ ] 阻塞和延期 story 出现在"什么做得不好"部分
- [ ] 至少从阻塞 story 生成一个行动项
- [ ] Skill 在写入文件前询问"可以写入吗"
- [ ] 成功写入后判定为 COMPLETE

---

### 用例 2：无 Sprint 数据——手动输入回退

**Fixture：**
- 用户调用 `/retrospective sprint-009`
- `production/sprints/sprint-009.md` 不存在
- 无会话日志引用 sprint-009

**输入：** `/retrospective sprint-009`

**预期行为：**
1. Skill 尝试读取 sprint-009——未找到
2. Skill 通知用户未找到 sprint-009 的 sprint 数据
3. Skill 提示用户手动提供 retrospective 输入（做得好、不好、行动）
4. 用户提供输入；skill 将其格式化为 retrospective 结构
5. Skill 询问"可以写入吗"并在批准后写入文档

**断言：**
- [ ] 当 sprint 文件缺失时 skill 不会崩溃或生成空文档
- [ ] 提示用户提供手动输入
- [ ] 手动输入格式化为三类别结构
- [ ] 文件写入前仍出现"可以写入吗"提示

---

### 用例 3：存在先前的 Retrospective——提供追加或替换选项

**Fixture：**
- `production/retrospectives/retro-sprint-005.md` 已存在，有内容
- 用户在更改后重新运行 `/retrospective sprint-005`

**输入：** `/retrospective sprint-005`

**预期行为：**
1. Skill 检测到 `retro-sprint-005.md` 已存在
2. Skill 向用户展示选择：追加新观察结果或替换现有文件
3. 用户选择"替换"；skill 编译新的 retrospective
4. Skill 询问"可以写入 `production/retrospectives/retro-sprint-005.md` 吗？"（确认覆盖）
5. 文件被覆盖；判定 COMPLETE

**断言：**
- [ ] Skill 在编译前检查现有 retrospective 文件
- [ ] 向用户提供追加或替换选择——不静默覆盖
- [ ] "可以写入吗"提示反映覆盖场景
- [ ] 无论追加 vs. 替换，写入后判定均为 COMPLETE

---

### 用例 4：边缘情况——先前 retrospective 中未解决的行动项

**Fixture：**
- `production/retrospectives/retro-sprint-004.md` 存在，有 2 个标记为 `[ ]` 的行动项（未完成）
- 用户运行 `/retrospective sprint-005`

**输入：** `/retrospective sprint-005`

**预期行为：**
1. Skill 读取最近的先前 retrospective（retro-sprint-004）
2. Skill 检测到 sprint-004 的 2 个未检查行动项
3. Skill 在新的 retrospective 中包含一个"Carry-over from Sprint 004"部分
4. 未解决的项目被列出并注明它们未被跟进

**断言：**
- [ ] Skill 读取最近的先前 retrospective 以检查未解决的行动项
- [ ] 未解决的行动项在新 retrospective 中的延期部分下出现
- [ ] 延期项目与新生成的行动项区分开
- [ ] 输出注明这些项目在先前的 sprint 中未被跟进

---

### 用例 5：关卡合规性——任何模式下都不调用关卡

**Fixture：**
- `production/sprints/sprint-005.md` 存在，有完成的 story
- `production/session-state/review-mode.txt` 包含 `full`

**输入：** `/retrospective sprint-005`

**预期行为：**
1. Skill 在 full 模式下编译 retrospective
2. 不调用 director 关卡（retrospective 是团队自我反思，不是交付关卡）
3. Skill 询问用户批准并在确认后写入文件
4. 判定为 COMPLETE

**断言：**
- [ ] 无论审查模式如何，不调用 director 关卡
- [ ] 输出不包含任何关卡调用或关卡结果注释
- [ ] Skill 直接从编译进入"可以写入吗"提示
- [ ] 审查模式文件内容与此 skill 的行为无关

---

## 协议合规性

- [ ] 请求写入前始终展示 retrospective 草稿
- [ ] 写入 retrospective 文件前始终询问"可以写入吗"
- [ ] 不调用 director 关卡
- [ ] 判定始终是 COMPLETE（不是通过/失败 skill）
- [ ] 检查先前 retrospective 中未解决的行动项

---

## 覆盖说明

- Milestone retrospective（相对于 sprint retrospective）遵循
  相同的模式，但读取里程碑文件而不是 sprint 文件；不在此处
  单独测试。
- 会话日志为空的情况类似于用例 2（无数据）；
  skill 在两种情况下都回退到手动输入。
