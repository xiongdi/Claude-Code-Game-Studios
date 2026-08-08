# 技能测试规格: /onboard

## 技能摘要

`/onboard` 为新的团队成员生成一个针对特定上下文的项目 onboarding 摘要。它读取 CLAUDE.md、`technical-preferences.md`、活动的 sprint 文件、最近的 git 提交以及 `production/stage.txt`，生成一个结构化的入职文档。该技能在 Haiku 模型上运行（只读，格式化任务）且不生成任何文件写入——所有输出都是对话式的。

该技能可选地接受一个角色参数（例如，`/onboard artist`）以针对特定学科定制摘要。当项目处于早期阶段或未配置时，输出会适应以反映已知的信息。裁决始终为 ONBOARDING COMPLETE——该技能纯粹是信息性的。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：ONBOARDING COMPLETE
- [ ] 不包含 "May I write" 语言（技能是只读的）
- [ ] 具有建议相关后续技能的下一步交接

---

## 导演门控检查

无。`/onboard` 是一个只读的入职技能。不适用 director gate。

---

## 测试用例

### Case 1: Happy Path — Configured project in Production stage with active sprint

**Fixture:**
- `production/stage.txt` 包含 `Production`
- `technical-preferences.md` 已填充引擎、语言和专家
- `production/sprints/sprint-005.md` 存在并有进行中的 story
- Git 日志包含 5 个最近的提交

**Input:** `/onboard`

**Expected behavior:**
1. Skill 读取 stage.txt、technical-preferences.md、活动 sprint 和 git 日志
2. Skill 生成包含以下部分的 onboarding 摘要：Project Overview、Tech Stack、
   Current Stage、Active Sprint Summary、Recent Activity
3. 摘要格式化以提高可读性（标题、项目符号）
4. 下一步建议适合 Production 阶段（例如，`/sprint-status`、
   `/dev-story`）
5. 声明 ONBOARDING COMPLETE 裁决

**Assertions:**
- [ ] 输出包含来自 stage.txt 的当前阶段名称
- [ ] 输出包含来自 technical-preferences.md 的引擎和语言
- [ ] 活动 sprint 的 story 被总结（不仅仅是 sprint 文件名）
- [ ] 存在最近的提交上下文
- [ ] 裁决为 ONBOARDING COMPLETE
- [ ] 不写入任何文件

---

### Case 2: Fresh Project — No engine, no sprint, suggests /start

**Fixture:**
- `technical-preferences.md` 仅包含占位符（`[TO BE CONFIGURED]`）
- 没有 `production/stage.txt`
- 没有 sprint 文件
- 除了默认值外没有 CLAUDE.md 覆盖

**Input:** `/onboard`

**Expected behavior:**
1. Skill 读取所有配置文件并检测到未配置状态
2. Skill 生成一个最小摘要："This project has not been configured yet"
3. 输出解释 onboarding 工作流：`/start` → `/setup-engine` → `/brainstorm`
4. Skill 建议立即运行 `/start` 作为下一步
5. 裁决为 ONBOARDING COMPLETE（信息性的，不是失败）

**Assertions:**
- [ ] 输出明确提到项目尚未配置
- [ ] 推荐 `/start` 作为下一步
- [ ] Skill 不会出错——它优雅地处理空项目状态
- [ ] 裁决仍为 ONBOARDING COMPLETE

---

### Case 3: No CLAUDE.md Found — Error with remediation

**Fixture:**
- `CLAUDE.md` 文件不存在（已删除或从未创建）
- 所有其他文件可能存在也可能不存在

**Input:** `/onboard`

**Expected behavior:**
1. Skill 尝试读取 CLAUDE.md 并失败
2. Skill 输出错误："CLAUDE.md not found — cannot generate onboarding summary"
3. Skill 提供修复方案："Run `/start` to initialize the project configuration"
4. 不生成部分摘要

**Assertions:**
- [ ] 错误消息明确将缺失的文件识别为 CLAUDE.md
- [ ] 明确命名修复步骤（`/start`）
- [ ] 当根配置缺失时，Skill 不会生成部分输出
- [ ] 裁决为 ONBOARDING COMPLETE（带有错误上下文，不是崩溃）

---

### Case 4: Role-Specific Onboarding — User specifies "artist" role

**Fixture:**
- 完全配置的 Production 阶段项目
- `art-bible.md` 存在于 `design/` 中
- 活动 sprint 有 visual story 类型（动画、VFX）

**Input:** `/onboard artist`

**Expected behavior:**
1. Skill 读取所有标准文件以及任何与艺术相关的文档（art bible、asset spec）
2. 摘要针对艺术家角色定制：art bible 概述、资源管道、
   活动 sprint 中的当前 visual story
3. 技术架构细节（代码结构、ADR）被弱化
4. 摘要中突出显示艺术/音频的专家 agent
5. 裁决为 ONBOARDING COMPLETE

**Assertions:**
- [ ] 角色参数在输出中被确认（"Onboarding for: Artist"）
- [ ] 如果文件存在，包含 art bible 摘要
- [ ] 显示活动 sprint 中的当前 visual story
- [ ] 技术实现细节不是主要焦点
- [ ] 裁决为 ONBOARDING COMPLETE

---

### Case 5: Director Gate Check — No gate; onboard is read-only orientation

**Fixture:**
- 任何已配置的项目状态

**Input:** `/onboard`

**Expected behavior:**
1. Skill 完成完整的 onboarding 摘要
2. 任何时候都不派生 director agent
3. 输出中不出现 gate ID
4. 不出现 "May I write" 提示

**Assertions:**
- [ ] 未调用 director gate
- [ ] 未调用写入工具
- [ ] 不出现 gate 跳过消息
- [ ] 无需任何 gate 检查即可达到 ONBOARDING COMPLETE

---

## 协议合规

- [ ] 在生成输出前读取所有源文件（没有虚构的项目状态）
- [ ] 根据项目阶段调整输出（Production ≠ Concept）
- [ ] 提供时尊重角色参数
- [ ] 不写入任何文件
- [ ] 所有路径都以 ONBOARDING COMPLETE 裁决结束

---

## 覆盖说明

- `technical-preferences.md` 完全缺失（而不是有占位符）的情况未单独测试；行为遵循 Case 3 的优雅错误模式。
- 假设 git 历史记录读取可用；离线/无 git 场景未在此测试。
- "artist" 以外的学科角色（例如，programmer、designer、producer）遵循与 Case 4 相同的定制模式，未单独测试。
