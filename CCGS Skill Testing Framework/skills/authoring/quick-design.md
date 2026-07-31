# Skill Test Spec: /quick-design

## Skill Summary

`/quick-design` 为太小而不需要完整 8 章节 GDD 的功能生成轻量级设计规格。目标范围是单个系统功能的设计时间少于 4 小时。quick-design 规格不使用完整的 8 章节 GDD 格式，而是使用精简的 3 章节格式：Overview、Rules 和 Acceptance Criteria。

该 skill 没有 director gate——添加 gate 开销将违背轻量级设计工具的目的。该 skill 在将设计说明写入 `design/quick-notes/[name].md` 之前询问 "May I write"。如果功能范围对于 quick-design 来说太大，该 skill 会重定向到 `/design-system`。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：CREATED、BLOCKED、REDIRECTED
- [ ] 包含 "May I write" 协作协议语言（针对 quick-note 文件）
- [ ] 末尾有下一步交接说明
- [ ] 明确注明：无 director gate（按设计为轻量级 skill）
- [ ] 提及范围检查：如果范围超过 sub-4h 阈值则重定向到 `/design-system`

---

## Director Gate Checks

无 director gate——此 skill 不派生任何 director gate agent。quick-design 的轻量级性质意味着 director gate 开销被有意省略。对于 sub-4 小时单系统功能不需要完整 GDD 审查。

---

## Test Cases

### Case 1: Happy Path——小型 UI 更改产生 3 章节规格

**Fixture:**
- 目标功能无现有 quick-note
- 功能范围明确：单一 UI 元素更改，无跨系统影响

**Input:** `/quick-design [feature-name]`

**Expected behavior:**
1. Skill 询问范围问题：什么系统、什么更改、验收信号是什么
2. Skill 确定范围在 sub-4h 阈值内
3. Skill 起草 3 章节规格：Overview、Rules、Acceptance Criteria
4. 草案显示给用户
5. 询问 "May I write `design/quick-notes/[name].md`?"
6. 批准后写入文件

**Assertions:**
- [ ] 规格恰好包含 3 个章节：Overview、Rules、Acceptance Criteria
- [ ] 草案在 "May I write" 请求前显示给用户
- [ ] 在写入前询问 "May I write `design/quick-notes/[name].md`?"
- [ ] 文件写入正确路径：`design/quick-notes/[name].md`
- [ ] 成功写入后裁定为 CREATED

---

### Case 2: Failure Path——范围检查失败；重定向到 /design-system

**Fixture:**
- 描述的功能跨越多个系统或需要超过 4 小时的设计时间
  （例如，"redesign the entire combat system" 或 "new progression mechanic affecting all classes"）

**Input:** `/quick-design [large-feature]`

**Expected behavior:**
1. Skill 询问范围问题
2. Skill 确定范围超过 sub-4h / 单系统阈值
3. Skill 输出："This feature is too large for a quick-design. Use `/design-system [name]` for a full GDD."
4. Skill 不写入 quick-note 文件
5. 裁定为 REDIRECTED

**Assertions:**
- [ ] Skill 检测范围超出并在起草前停止
- [ ] 消息明确命名 `/design-system` 作为正确的替代方案
- [ ] 不写入 quick-note 文件
- [ ] 裁定为 REDIRECTED（不是 CREATED 或 BLOCKED）

---

### Case 3: Edge Case——文件已存在；提供更新选项

**Fixture:**
- `design/quick-notes/[name].md` 已从先前会话存在

**Input:** `/quick-design [name]`

**Expected behavior:**
1. Skill 检测现有 quick-note 文件并读取其当前内容
2. Skill 询问："[name].md already exists. Update it, or create a new version?"
3. 用户选择更新
4. Skill 显示现有规格并询问要修订哪个章节
5. 显示更新后的规格，询问 "May I write?"，批准后更新文件

**Assertions:**
- [ ] Skill 在提供更新选项前检测并读取现有文件
- [ ] 用户提供更新或创建新选项——不自动覆盖
- [ ] 仅更新修订的章节（或如果用户选择完整重写则更新整个规格）
- [ ] 在覆盖现有文件前询问 "May I write"

---

### Case 4: Edge Case——未提供参数

**Fixture:**
- `design/quick-notes/` 目录可能存在也可能不存在

**Input:** `/quick-design`（无参数）

**Expected behavior:**
1. Skill 检测未提供参数
2. Skill 输出用法错误："No feature name specified. Usage: /quick-design [feature-name]"
3. Skill 提供示例：`/quick-design pause-menu-settings`
4. 不创建文件

**Assertions:**
- [ ] Skill 在未提供参数时输出用法错误
- [ ] 显示具有正确格式的用法示例
- [ ] 不写入 quick-note 文件
- [ ] Skill 不静默选择功能名称或默认任何操作

---

### Case 5: Director Gate——不派生 gate；针对 sub-4h 功能明确注明

**Fixture:**
- 功能在 quick-design 范围内
- `production/session-state/review-mode.txt` 存在，内容为 `full`

**Input:** `/quick-design [feature-name]`

**Expected behavior:**
1. Skill 询问范围问题并确定范围在阈值内
2. Skill 不读取 `production/session-state/review-mode.txt`
3. Skill 不派生任何 director gate agent
4. 规格起草完成，询问 "May I write"，批准后写入文件
5. 输出明确注明："No director gate review — quick-design is for sub-4h features"

**Assertions:**
- [ ] 不派生 director gate agent（无 CD-、TD-、PR-、AD- 前缀的 gate）
- [ ] Skill 不读取 `production/session-state/review-mode.txt`
- [ ] 输出包含解释为何不需要 gate 审查的说明
- [ ] 审查模式对此 skill 的行为无影响
- [ ] 完整 GDD 审查路径（`/design-system`）被提及为更大功能的替代方案

---

## Protocol Compliance

- [ ] 范围检查在起草前运行（如果范围太大则重定向到 `/design-system`）
- [ ] 使用 3 章节格式（Overview、Rules、Acceptance Criteria）——不是 8 章节 GDD 格式
- [ ] 草案在 "May I write" 请求前显示给用户
- [ ] 在写入前询问 "May I write `design/quick-notes/[name].md`?"
- [ ] 无 director gate——不读取 review-mode.txt
- [ ] 以下一步交接说明结束（例如，继续实现或 `/dev-story`）

---

## Coverage Notes

- 范围阈值启发式（sub-4h，单系统）是判断问题——
  skill 的内部检查是权威定义，不通过计算小时数独立测试。
- `design/quick-notes/` 目录如果不存在会自动创建——
  此文件系统行为不在此独立测试。
- 与 story 管道的集成（quick-design 能否直接生成 story？）
  不在此 spec 范围内——quick-design 是独立的。
