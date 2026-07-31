# Skill 测试规格：/patch-notes

## Skill 摘要

`/patch-notes` 是 Haiku 层级的 skill，从现有 changelog 内容生成面向玩家的 patch notes，
将内部任务 ID 和技术术语替换为通俗语言。它过滤条目仅保留与玩家相关的
（可见的功能和 bug 修复；内部重构被排除）。
不使用 director 关卡。该 skill 在持久化前询问"可以写入
`docs/patch-notes-vX.X.md` 吗？"。判定结果始终是 COMPLETE。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE
- [ ] 包含"可以写入吗"语言（skill 写入 patch notes 文件）
- [ ] 有下一步交接（例如，与 community manager 分享）

---

## Director 关卡检查

无。Patch notes 生成是快速编译任务；不调用关卡。

---

## 测试用例

### 用例 1：正常路径——Changelog 过滤为面向玩家的条目

**Fixture：**
- `docs/CHANGELOG.md` 存在，有 5 个条目：
  - "Add dual-wield melee system"（Features——面向玩家）
  - "Fix crash on level transition"（Fixes——面向玩家）
  - "Add enemy patrol AI"（Features——面向玩家）
  - "Refactor input handler to use event bus"（Fixes——仅内部）
  - "Update dependency: Godot 4.6"（仅内部）
- 版本为 `v0.4.0`

**输入：** `/patch-notes v0.4.0`

**预期行为：**
1. Skill 读取 `docs/CHANGELOG.md`
2. Skill 过滤为 3 个面向玩家的条目；排除 2 个内部条目
3. Skill 用通俗语言重写条目（无任务 ID，无技术术语）
4. Skill 向用户展示草稿
5. Skill 询问"可以写入 `docs/patch-notes-v0.4.0.md` 吗？"
6. 用户批准；文件写入；判定 COMPLETE

**断言：**
- [ ] 只有 3 个条目出现在 patch notes 中（2 个内部条目被排除）
- [ ] 条目用通俗语言书写，无内部任务 ID
- [ ] 文件路径匹配 `docs/patch-notes-v0.4.0.md`
- [ ] 文件写入前出现"可以写入吗"提示
- [ ] 写入后判定为 COMPLETE

---

### 用例 2：未找到 Changelog——引导先运行 /changelog

**Fixture：**
- `docs/CHANGELOG.md` 不存在

**输入：** `/patch-notes v0.4.0`

**预期行为：**
1. Skill 尝试读取 `docs/CHANGELOG.md`——未找到
2. Skill 输出："No changelog found——run /changelog first to generate one"
3. 不生成 patch notes；不写入文件

**断言：**
- [ ] 当 changelog 缺失时 skill 不会崩溃
- [ ] 输出明确引导用户运行 `/changelog`
- [ ] 不出现"可以写入吗"提示（无内容可写）
- [ ] 判定为 BLOCKED（依赖未满足）

---

### 用例 3：来自设计文件夹的语调指导——融入输出

**Fixture：**
- `docs/CHANGELOG.md` 存在，有面向玩家的条目
- `design/community/tone-guide.md` 存在，指导："upbeat, encouraging tone; avoid passive voice"

**输入：** `/patch-notes v0.4.0`

**预期行为：**
1. Skill 读取 changelog
2. Skill 检测到 `design/community/tone-guide.md` 中的语调指导
3. Skill 在重写条目为通俗语言时应用语调指导
4. Patch notes 使用乐观、主动语态的措辞
5. Skill 展示草稿，请求写入，批准后写入

**断言：**
- [ ] Skill 检查 `design/` 中的社区或语调指导文件
- [ ] 语调指导内容影响 patch notes 条目的措辞
- [ ] 输出在适用处反映主动语态和乐观语调
- [ ] Skill 注明已应用语调指导

---

### 用例 4：存在 Patch Note 模板——使用而非生成结构

**Fixture：**
- `.claude/docs/templates/patch-notes-template.md` 存在，有结构化的头部格式
- `docs/CHANGELOG.md` 存在，有面向玩家的条目

**输入：** `/patch-notes v0.4.0`

**预期行为：**
1. Skill 读取 changelog 并检测到模板存在
2. Skill 用面向玩家的条目填充模板
3. 模板头部/底部结构在输出中保留
4. Skill 询问"可以写入吗"并在批准后写入

**断言：**
- [ ] Skill 在从头生成前检查 patch notes 模板
- [ ] 找到模板时使用模板结构（不被默认格式覆盖）
- [ ] 面向玩家的条目插入到正确的模板部分
- [ ] 输出注释确认使用了模板

---

### 用例 5：关卡合规性——无关卡；community-manager 是单独的

**Fixture：**
- `docs/CHANGELOG.md` 存在，有面向玩家的条目
- `review-mode.txt` 包含 `full`

**输入：** `/patch-notes v0.4.0`

**预期行为：**
1. Skill 在 full 模式下编译 patch notes
2. 不调用 director 关卡（社区审查是单独的手动步骤）
3. Skill 在 Haiku 模型上运行——快速编译
4. Skill 在输出中注明："Consider sharing draft with community manager before publishing"
5. Skill 询问用户批准并在确认后写入

**断言：**
- [ ] 无论审查模式如何，不调用 director 关卡
- [ ] 输出建议（但不要求）community manager 审查
- [ ] Skill 直接从编译进入"可以写入吗"提示
- [ ] 判定为 COMPLETE

---

## 协议合规性

- [ ] 生成 patch notes 前读取 `docs/CHANGELOG.md`
- [ ] 过滤条目仅为面向玩家的项目
- [ ] 用通俗语言重写条目，无内部 ID
- [ ] 写入 patch notes 文件前始终询问"可以写入吗"
- [ ] 不调用 director 关卡
- [ ] 在 Haiku 模型层级运行（快速、低成本）

---

## 覆盖说明

- 所有 changelog 条目都是内部的（零个面向玩家的项目）的情况未测试；
  行为是空的 patch notes 草稿并附带警告。
- 从 changelog 头部解析版本号是实现细节，
  未在此处验证。
- 用例 5 中注明的 community manager 咨询是建议性的；单独的
  skill 或手动审查处理该步骤。
