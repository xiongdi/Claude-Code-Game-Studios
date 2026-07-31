# Skill 测试规格：/help

## Skill 摘要

`/help` 分析已完成的工作和项目工作流中接下来要做的事情。
它在 Haiku 模型上运行（只读、格式化任务），并读取 `production/stage.txt`、
活动 sprint 文件和最近的会话状态，以生成简明的情境指导摘要。
该 skill 可选地接受上下文查询（例如，`/help testing`）以针对特定主题展示相关 skill。

输出始终是信息性的——不写入任何文件，也不调用任何 director 关卡。
判定结果始终是 HELP COMPLETE。该 skill 充当工作流导航器，
根据当前项目状态建议 2-3 个下一步 skill。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：HELP COMPLETE
- [ ] 不需要"可以写入吗"语言（skill 是只读的）
- [ ] 有下一步交接（根据状态建议 2-3 个相关 skill）

---

## Director 关卡检查

无。`/help` 是只读的导航 skill。不适用 director 关卡。

---

## 测试用例

### 用例 1：正常路径——Production 阶段，有活动 sprint

**Fixture：**
- `production/stage.txt` 包含 `Production`
- `production/sprints/sprint-004.md` 存在，有进行中的 story
- `production/session-state/active.md` 有最近的检查点

**输入：** `/help`

**预期行为：**
1. Skill 读取 stage.txt 和活动 sprint
2. Skill 识别当前 sprint 编号和进行中 story 的数量
3. Skill 输出：当前阶段、sprint 摘要和 3 个建议的下一步 skill
   （例如，`/sprint-status`、`/dev-story`、`/story-done`）
4. 建议按与当前 sprint 状态的相关性排序
5. 判定为 HELP COMPLETE

**断言：**
- [ ] 显示当前阶段（Production）
- [ ] 提及活动 sprint 编号和 story 数量
- [ ] 恰好给出 2-3 个下一步 skill 建议（不是所有 skill 的列表）
- [ ] 建议适合 Production 阶段
- [ ] 判定为 HELP COMPLETE
- [ ] 未写入任何文件

---

### 用例 2：Concept 阶段——展示概念到系统设计的工作流路径

**Fixture：**
- `production/stage.txt` 包含 `Concept`
- 无 sprint 文件，无 GDD 文件
- `technical-preferences.md` 已配置（引擎已选择）

**输入：** `/help`

**预期行为：**
1. Skill 读取 stage.txt——检测到 Concept 阶段
2. Skill 输出 Concept 阶段工作流：brainstorm → map-systems → design-system
3. 建议的 skill：`/brainstorm`、`/map-systems`（如果概念存在）
4. 注意当前进度："Engine configured, concept not yet created"

**断言：**
- [ ] 阶段被识别为 Concept
- [ ] 工作流路径显示此阶段的预期顺序
- [ ] 建议不包含 Production 阶段的 skill（例如，`/dev-story`）
- [ ] 判定为 HELP COMPLETE

---

### 用例 3：无 stage.txt——展示完整工作流概览

**Fixture：**
- 无 `production/stage.txt`
- 无 sprint 文件
- `technical-preferences.md` 有占位符

**输入：** `/help`

**预期行为：**
1. Skill 无法从 stage.txt 确定阶段
2. Skill 运行 project-stage-detect 逻辑从产物推断阶段
3. 如果无法推断阶段：输出从 Concept 到 Release 的完整工作流概览
   作为参考地图
4. 主要建议是 `/start` 开始配置

**断言：**
- [ ] 当 stage.txt 缺失时 skill 不会崩溃
- [ ] 当无法确定阶段时显示完整工作流概览
- [ ] `/start` 或 `/project-stage-detect` 是首要建议
- [ ] 判定为 HELP COMPLETE

---

### 用例 4：上下文查询——用户询问测试相关帮助

**Fixture：**
- `production/stage.txt` 包含 `Production`
- 活动 sprint 有一个 `Status: In Review` 的 story

**输入：** `/help testing`

**预期行为：**
1. Skill 读取上下文查询："testing"
2. Skill 展示与测试相关的 skill：`/qa-plan`、`/smoke-check`、
   `/regression-suite`、`/test-setup`、`/test-evidence-review`
3. 输出聚焦于测试工作流，而非通用 sprint 导航
4. 当前审查中的 story 被突出显示为测试候选

**断言：**
- [ ] 输出中确认上下文查询（"Help topic: testing"）
- [ ] 列出至少 3 个测试相关 skill
- [ ] 通用 sprint skill（例如，`/sprint-plan`）不是主要建议
- [ ] 判定为 HELP COMPLETE

---

### 用例 5：Director 关卡检查——无关卡；help 是只读导航

**Fixture：**
- 任何项目状态

**输入：** `/help`

**预期行为：**
1. Skill 生成工作流指导摘要
2. 不派生任何 director agent
3. 输出中不出现 gate ID
4. 不调用写入工具

**断言：**
- [ ] 未调用 director 关卡
- [ ] 未调用写入工具
- [ ] 不出现 gate 跳过消息
- [ ] 判定为 HELP COMPLETE，无任何关卡检查

---

## 协议合规性

- [ ] 生成建议之前读取 stage、sprint 和会话状态
- [ ] 建议针对当前项目状态（不是通用的）
- [ ] 上下文查询（如提供）缩小建议集
- [ ] 不写入任何文件
- [ ] 所有情况下判定均为 HELP COMPLETE

---

## 覆盖说明

- 活动 sprint 已完成（所有 story 为 Done）的情况未单独测试；
  skill 会建议 `/sprint-plan` 用于下一个 sprint。
- `/help` skill 不验证建议的 skill 是否可用——
  它假设标准 skill 目录可用。
- 阶段检测回退（当 stage.txt 缺失时）委托给与 `/project-stage-detect` 相同的
  逻辑，此处不再详细重测。
