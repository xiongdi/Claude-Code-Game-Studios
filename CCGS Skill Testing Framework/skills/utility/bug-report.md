# 技能测试规格: /bug-report

## 技能摘要

`/bug-report` 从用户描述创建结构化 bug 报告文档。它生成具有以下必需字段的报告：Title、Repro Steps、Expected Behavior、Actual Behavior、Severity（CRITICAL/HIGH/MEDIUM/LOW）、Affected System(s) 和 Build/Version。如果用户的初始描述缺少任何必需字段，该 skill 会提出后续问题以在产生草案前填补空白。

该 skill 检查可能的重复报告（通过与 `production/bugs/` 中的现有文件比较）并提供链接而非创建新报告的选项。每份报告在 "May I write" 请求后写入 `production/bugs/bug-[date]-[slug].md`。不使用 director gate——bug 报告是操作性的实用工具。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：COMPLETE
- [ ] 在写入报告前包含 "May I write" 协作协议语言
- [ ] 具有下一步交接说明（例如，`/bug-triage` 重新确定优先级，`/hotfix` 用于关键问题）

---

## 导演门控检查

无。`/bug-report` 是操作文档 skill。不适用 director gate。

---

## 测试用例

### Case 1: Happy Path——用户描述崩溃，生成完整报告

**Fixture:**
- `production/bugs/` 目录存在且为空
- 无类似现有报告

**Input:** `/bug-report`（用户描述："Game crashes when player enters the boss arena"）

**Expected behavior:**
1. Skill 提取：Title = "Game crashes when entering boss arena"
2. Skill 将崩溃报告识别为 CRITICAL 严重性
3. Skill 与用户确认复现步骤、预期（无崩溃）、实际（崩溃）、受影响的系统
   （arena/boss）和构建版本
4. Skill 起草完整的结构化报告
5. Skill 询问 "May I write to `production/bugs/bug-2026-04-06-game-crashes-boss-arena.md`?"
6. 批准后写入文件；裁定为 COMPLETE

**Assertions:**
- [ ] 报告中存在所有 7 个必需字段
- [ ] 崩溃报告的严重性为 CRITICAL
- [ ] 文件名遵循 `bug-[date]-[slug].md` 约定
- [ ] "May I write" 使用完整文件路径询问
- [ ] 裁定为 COMPLETE

---

### Case 2: Minimal Input——Skill 提出后续问题以填补缺失字段

**Fixture:**
- 用户提供："Sometimes the audio cuts out"
- 无现有报告

**Input:** `/bug-report`

**Expected behavior:**
1. Skill 识别缺失的必需字段：复现步骤、预期与实际、
   严重性、受影响的系统、构建版本
2. Skill 针对每个缺失字段提出有针对性的后续问题（一次一个
   或结构化提示）
3. 用户提供答案
4. Skill 从答案编译完整报告
5. Skill 询问 "May I write?" 并在批准后写入

**Assertions:**
- [ ] 至少提出 3 个后续问题以填补缺失字段
- [ ] 每个必需字段在报告最终化前都已填写
- [ ] 在所有必需字段存在前不写入报告
- [ ] 所有字段填写且文件写入后裁定为 COMPLETE

---

### Case 3: Possible Duplicate——提供链接而非创建新报告

**Fixture:**
- `production/bugs/bug-2026-03-20-audio-cut-out.md` 已存在，具有类似标题和 MEDIUM 严重性

**Input:** `/bug-report`（用户描述："Audio randomly stops working"）

**Expected behavior:**
1. Skill 扫描现有报告并找到类似的音频 bug
2. Skill 报告："A similar bug report exists: bug-2026-03-20-audio-cut-out.md"
3. Skill 呈现选项：链接为重复（在现有文件中添加说明）、仍然创建新报告
4. 如果用户选择链接：skill 在现有文件中添加交叉引用说明
   （询问 "May I update the existing report?"）
5. 如果用户选择创建新报告：正常报告创建继续

**Assertions:**
- [ ] 在创建新报告前展示现有类似报告
- [ ] 用户被给予选择（不强制链接或创建）
- [ ] 如果链接：在修改现有文件前询问 "May I update"
- [ ] 任一路径下裁定都为 COMPLETE

---

### Case 4: Multi-System Bug——创建具有多个系统标签的报告

**Fixture:**
- 无现有报告

**Input:** `/bug-report`（用户描述："After finishing a level, the save system
  freezes and the UI doesn't show the completion screen"）

**Expected behavior:**
1. Skill 从描述中识别 2 个受影响的系统：Save System 和 UI
2. 报告起草时在 Affected System(s) 下列出两个系统
3. 评估严重性（可能为 HIGH——保存冻结导致的数据丢失风险）
4. Skill 询问 "May I write"，使用适当的文件名
5. 报告写入时标记了两个系统；裁定为 COMPLETE

**Assertions:**
- [ ] 报告中列出了两个受影响的系统
- [ ] 创建单一报告（不是每个系统一个）
- [ ] 严重性反映最具影响的组件（保存冻结 → HIGH 或 CRITICAL）
- [ ] 裁定为 COMPLETE

---

### Case 5: Director Gate Check——无 gate；bug 报告是操作性的

**Fixture:**
- 提供了任何 bug 描述

**Input:** `/bug-report`

**Expected behavior:**
1. Skill 创建并写入 bug 报告
2. 不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 不调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] Skill 无需任何 gate 检查即达到 COMPLETE

---

## 协议合规

- [ ] 在起草报告前收集所有 7 个必需字段
- [ ] 对任何缺失的必需字段提出后续问题
- [ ] 在创建新报告前检查类似的现有报告
- [ ] 在写入前询问 "May I write to `production/bugs/bug-[date]-[slug].md`?"
- [ ] 当报告文件写入后裁定为 COMPLETE

---

## 覆盖说明

- 用户提供的严重性对于描述的影响来说似乎太低的情况（例如，崩溃为 LOW）不在此测试；
  skill 可能建议更高的严重性但最终尊重用户输入。
- Build/Version 字段是必需的，但如果用户不知道可能是 "unknown"——
  这被接受为有效值，不单独测试。
- 报告 slug 生成（将标题清理为文件名）是实现细节，不在此进行断言测试。
