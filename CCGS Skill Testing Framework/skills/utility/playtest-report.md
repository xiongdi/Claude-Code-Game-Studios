# Skill 测试规格：/playtest-report

## Skill 摘要

`/playtest-report` 从会话笔记或用户输入生成结构化的 playtest 报告。
报告分为四个部分：Feel/Accessibility、Bugs Observed、Design Feedback 和 Next Steps。
当有多个测试者参与时，skill 汇总反馈并区分多数意见和少数意见。
当报告的 bug 与 `production/bugs/` 中的文件匹配时，skill 链接到现有 bug 报告。

报告在"可以写入吗"询问后写入 `production/qa/playtest-[date].md`。
此处不适用 director 关卡——CD-PLAYTEST director 关卡（如需要）是
单独的调用。判定结果为 COMPLETE 当报告写入时。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE
- [ ] 在写入报告前包含"可以写入吗"协作协议语言
- [ ] 有下一步交接（例如，`/bug-report` 处理新发现的问题，`/design-review` 处理反馈）

---

## Director 关卡检查

无。`/playtest-report` 是文档工具。CD-PLAYTEST 关卡是
单独的调用，不是此 skill 的一部分。

---

## 测试用例

### 用例 1：正常路径——用户提供 playtest 笔记，生成结构化报告

**Fixture：**
- 用户提供单次会话的打字 playtest 笔记
- 笔记涵盖：游戏手感、一个 bug（帧率下降）和设计关切
  （教程太长）
- `production/bugs/` 存在但为空（bug 尚未报告）

**输入：** `/playtest-report`（用户粘贴会话笔记）

**预期行为：**
1. Skill 读取提供的笔记并将其结构化为 4 部分模板
2. Feel/Accessibility：提取手感观察
3. Bugs：记录帧率下降及可用的复现细节
4. Design Feedback：记录教程长度关切
5. Next Steps：建议对帧率问题使用 `/bug-report`，对教程反馈使用
   `/design-review`
6. Skill 询问"可以写入 `production/qa/playtest-2026-04-06.md` 吗？"
7. 批准后报告写入；判定为 COMPLETE

**断言：**
- [ ] 报告中包含全部 4 个部分
- [ ] Bug 列在 Bugs 部分（不是 Design Feedback 部分）
- [ ] Next Steps 适当（崩溃用 bug report，反馈用 design review）
- [ ] 写入前询问"可以写入吗"
- [ ] 判定为 COMPLETE

---

### 用例 2：空输入——引导式逐部分提示

**Fixture：**
- 调用时用户未提供笔记

**输入：** `/playtest-report`

**预期行为：**
1. Skill 检测到空输入
2. Skill 逐部分提示：
   a. "Describe the overall feel and any accessibility observations"
   b. "Were any bugs observed? Describe them"
   c. "What design feedback did testers provide?"
3. 用户回答每个提示
4. Skill 从回答中编译报告并询问"可以写入吗"
5. 批准后报告写入；判定为 COMPLETE

**断言：**
- [ ] 至少询问 3 个引导性问题（每个主要部分一个）
- [ ] 直到所有部分都有输入（或用户明确跳过）才创建报告
- [ ] 文件写入后判定为 COMPLETE

---

### 用例 3：多个测试者——汇总反馈并标注多数/少数意见

**Fixture：**
- 用户提供 3 个测试者的笔记
- 2/3 测试者认为操控"直观"
- 1/3 测试者认为 UI 字体太小
- 所有 3 人都注意到相同的 bug（玩家卡在窗沿）

**输入：** `/playtest-report`（3 测试者会话）

**预期行为：**
1. Skill 在输入中识别 3 个不同的测试者视角
2. 操控直观性 → 标注为 "Majority (2/3): controls intuitive"
3. 字体大小 → 标注为 "Minority (1/3): UI font size concern"
4. 卡窗沿 bug → 标注为 "All testers: player stuck on ledge (confirmed)"
5. Skill 生成带有多数/少数标签的汇总报告
6. "可以写入吗"批准后报告写入；判定为 COMPLETE

**断言：**
- [ ] 多数意见 (2/3) 被标注为多数
- [ ] 少数意见 (1/3) 被标注为少数
- [ ] 一致报告的 bug 被标注为所有测试者确认
- [ ] 判定为 COMPLETE

---

### 用例 4：Bug 匹配现有报告——链接到现有文件

**Fixture：**
- `production/bugs/bug-2026-03-30-player-stuck-ledge.md` 存在
- 用户的 playtest 笔记描述"player gets stuck on ledges near walls"

**输入：** `/playtest-report`

**预期行为：**
1. Skill 结构化报告并识别卡窗沿 bug
2. Skill 扫描 `production/bugs/` 并找到 `bug-2026-03-30-player-stuck-ledge.md`
3. 在 Bugs 部分，报告包含："See existing report:
   production/bugs/bug-2026-03-30-player-stuck-ledge.md"
4. Skill 不建议为此问题创建新 bug 报告
5. 报告写入；判定为 COMPLETE

**断言：**
- [ ] 在 playtest 报告中找到并链接现有 bug 报告
- [ ] 对已报告的问题不建议 `/bug-report`
- [ ] Bugs 部分出现对现有文件的交叉引用
- [ ] 判定为 COMPLETE

---

### 用例 5：Director 关卡检查——无关卡；CD-PLAYTEST 是单独调用

**Fixture：**
- 提供了 playtest 笔记

**输入：** `/playtest-report`

**预期行为：**
1. Skill 生成并写入 playtest 报告
2. 不派生任何 director agent（CD-PLAYTEST 不在此处调用）
3. 输出中不出现 gate ID

**断言：**
- [ ] 未调用 director 关卡
- [ ] 不出现 CD-PLAYTEST gate 跳过消息
- [ ] 判定为 COMPLETE，无任何关卡检查

---

## 协议合规性

- [ ] 将输出结构化为全部 4 个部分（Feel、Bugs、Design Feedback、Next Steps）
- [ ] 涉及多个测试者时标注多数 vs. 少数意见
- [ ] bug 匹配时交叉引用现有 bug 报告
- [ ] 写入前询问"可以写入 `production/qa/playtest-[date].md` 吗？"
- [ ] 报告写入后判定为 COMPLETE

---

## 覆盖说明

- CD-PLAYTEST director 关卡（创意总监审查 playtest 洞察
  以获取设计影响）是单独的调用，不在此处测试。
- 视频录制或截图附件未测试；报告是纯文本文档。
- 测试者身份未知（匿名反馈）的情况遵循与用例 3 相同的汇总模式，
  但不带测试者标签。
