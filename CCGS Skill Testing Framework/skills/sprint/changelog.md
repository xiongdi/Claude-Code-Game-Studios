# Skill 测试规格：/changelog

## Skill 摘要

`/changelog` 是 Haiku 层级的 skill，通过读取自上一个发布标签以来的 git 提交历史
和已关闭的 sprint story 来自动生成面向开发者的 changelog。
它将条目组织为 features、fixes 和 known issues。不使用 director 关卡。
该 skill 在持久化前询问"可以写入 `docs/CHANGELOG.md` 吗？"。
判定结果始终是 COMPLETE。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE
- [ ] 包含"可以写入吗"语言（skill 写入 changelog）
- [ ] 有下一步交接（例如，运行 /patch-notes 生成面向玩家的版本）

---

## Director 关卡检查

无。Changelog 生成是快速编译任务；不调用关卡。

---

## 测试用例

### 用例 1：正常路径——自上一个发布标签以来有多个 sprint

**Fixture：**
- Git 历史在三个 sprint 前有标签 `v0.3.0`
- 自该标签以来：跨 sprint 006、007、008 有 12 个提交
- Sprint story 文件引用与提交消息匹配的任务 ID
- `docs/CHANGELOG.md` 尚不存在

**输入：** `/changelog`

**预期行为：**
1. Skill 读取自 `v0.3.0` 标签以来的 git log
2. Skill 读取 sprint story 以交叉引用任务 ID
3. Skill 将条目编译为 Features、Fixes 和 Known Issues 部分
4. Skill 向用户展示草稿
5. Skill 询问"可以写入 `docs/CHANGELOG.md` 吗？"
6. 用户批准；文件写入；判定 COMPLETE

**断言：**
- [ ] Changelog 涵盖自最近 git 标签以来的提交
- [ ] 条目组织为 Features / Fixes / Known Issues 部分
- [ ] Sprint story 引用用于丰富提交描述
- [ ] 文件写入前出现"可以写入吗"提示
- [ ] 写入后判定为 COMPLETE

---

### 用例 2：未找到 Git 标签——使用所有提交，注明版本基线

**Fixture：**
- Git 仓库有提交但无标签
- 历史中有跨 3 个 sprint 的 20 个提交

**输入：** `/changelog`

**预期行为：**
1. Skill 检查 git 标签——未找到
2. Skill 使用历史中的所有提交作为基线
3. Skill 在输出中注明："No version tag found——using full commit history; version baseline is unset"
4. Skill 仍从可用提交编译有组织的 changelog
5. Skill 询问"可以写入吗"并在批准后写入

**断言：**
- [ ] 当无 git 标签存在时 skill 不会报错
- [ ] 输出明确注明未找到版本基线
- [ ] 使用完整的提交历史作为源
- [ ] 尽管缺少标签，changelog 仍组织为多个部分

---

### 用例 3：无任务 ID 的提交消息——按日期分组并注明

**Fixture：**
- 自上一个标签以来的 git log 有 8 个提交
- 5 个提交在消息中没有任务 ID（例如，"fix typo"、"tweak values"）
- 3 个提交引用与 sprint story 匹配的任务 ID

**输入：** `/changelog`

**预期行为：**
1. Skill 读取提交和 sprint story
2. 3 个提交匹配到 sprint story 并放入适当部分
3. 5 个未标记的提交按日期分组在 "Misc" 或 "Other Changes" 部分下
4. 输出注明："5 commits without task IDs——grouped by date"
5. Skill 在批准后写入 changelog

**断言：**
- [ ] 有任务 ID 的提交放入适当部分（Features 或 Fixes）
- [ ] 无任务 ID 的提交单独分组并注明
- [ ] 输出标记缺少任务引用的提交数量
- [ ] 没有提交被静默丢弃

---

### 用例 4：现有 CHANGELOG.md——新部分前置，旧条目保留

**Fixture：**
- `docs/CHANGELOG.md` 已存在，包含 `v0.2.0` 和 `v0.3.0` 部分
- 自 `v0.3.0` 标签以来有新提交

**输入：** `/changelog`

**预期行为：**
1. Skill 检测到 `docs/CHANGELOG.md` 已存在
2. Skill 编译自 `v0.3.0` 以来期间的新条目
3. Skill 展示草稿，新部分前置于现有内容之上
4. Skill 询问"可以写入 `docs/CHANGELOG.md` 吗？"（确认前置策略）
5. 用户批准；新内容前置，旧条目完好；判定 COMPLETE

**断言：**
- [ ] Skill 在写入前读取现有 changelog 以检测先前内容
- [ ] 新部分前置（不是追加或覆盖）现有条目
- [ ] v0.2.0 和 v0.3.0 的旧 changelog 条目在写入文件中保留
- [ ] "可以写入吗"提示反映前置操作

---

### 用例 5：关卡合规性——无关卡；读取后写入并需批准

**Fixture：**
- Git 历史自上一个标签以来有提交
- `review-mode.txt` 包含 `full`

**输入：** `/changelog`

**预期行为：**
1. Skill 在 full 模式下编译 changelog
2. 不调用 director 关卡（changelog 生成是编译，不是交付关卡）
3. Skill 在 Haiku 模型上运行——快速编译
4. Skill 询问用户批准并在确认后写入文件

**断言：**
- [ ] 无论审查模式如何，不调用 director 关卡
- [ ] 输出不引用任何关卡结果
- [ ] Skill 直接从编译进入"可以写入吗"提示
- [ ] 判定为 COMPLETE

---

## 协议合规性

- [ ] 编译前读取 git log 和 sprint story 文件
- [ ] 写入 changelog 前始终询问"可以写入吗"
- [ ] 不调用 director 关卡
- [ ] 判定始终是 COMPLETE
- [ ] 在 Haiku 模型层级运行（快速、低成本）

---

## 覆盖说明

- 仓库中未初始化 git 的情况未测试；
  行为取决于 git 命令失败处理。
- 这些测试中未明确区分 merge commits vs. squash commits；
  这是 git log 解析阶段的实现细节。
- `/patch-notes` skill 应在 `/changelog` 之后运行以生成面向玩家的
  输出；该交接在 patch-notes 规格中验证。
