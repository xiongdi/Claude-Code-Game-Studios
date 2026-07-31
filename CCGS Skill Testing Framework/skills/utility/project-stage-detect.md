# Skill 测试规格：/project-stage-detect

## Skill 摘要

`/project-stage-detect` 自动分析项目产物以确定当前开发阶段。它在 Haiku 模型上运行（只读），检查 `production/stage.txt`（如果存在）、`design/` 中的设计文档、`src/` 中的源代码、`production/` 中的 sprint 和里程碑文件，以及引擎配置的存在情况，将项目分类为七个阶段之一：Concept、Systems Design、Technical Setup、Pre-Production、Production、Polish 或 Release。

该 skill 是建议性的——它从不写入 `stage.txt`。该文件仅在 `/gate-check` 通过且用户确认推进时才会更新。该 skill 报告其置信度（如果直接读取了 stage.txt 则为 HIGH，如果从产物推断则为 MEDIUM，如果发现冲突信号则为 LOW）。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含所有七个阶段名称：Concept、Systems Design、Technical Setup、Pre-Production、Production、Polish、Release
- [ ] 不包含 "May I write" 语言（该 skill 仅用于检测）
- [ ] 有下一步交接（例如，`/gate-check` 以正式推进阶段）

---

## Director 关卡检查

无。`/project-stage-detect` 是一个只读检测工具。不适用 director gate。

---

## 测试用例

### 用例 1：stage.txt 存在——直接读取并交叉检查产物

**Fixture：**
- `production/stage.txt` 包含 `Production`
- `design/gdd/` 有 4 个 GDD 文件
- `src/` 有源代码文件
- `production/sprints/sprint-002.md` 存在

**输入：** `/project-stage-detect`

**预期行为：**
1. Skill 读取 `production/stage.txt`——检测到阶段 `Production`
2. Skill 交叉检查产物：GDD 存在、源代码存在、sprint 存在
3. 产物与 Production 阶段一致
4. Skill 报告：Stage = Production，Confidence = HIGH（来自 stage.txt，经产物确认）
5. 下一步：继续执行 `/sprint-plan` 或 `/dev-story`

**断言：**
- [ ] 检测到的阶段是 Production
- [ ] 当 stage.txt 存在时置信度报告为 HIGH
- [ ] 注明交叉检查结果（一致 vs. 不一致）
- [ ] 不写入任何文件
- [ ] 裁决明确说明检测到的阶段

---

### 用例 2：无 stage.txt 但 GDD 和 Epic 存在——推断为 Production

**Fixture：**
- 没有 `production/stage.txt`
- `design/gdd/` 有 3 个 GDD 文件
- `production/epics/` 有 2 个 epic 文件
- `src/` 有源代码文件
- `production/sprints/sprint-001.md` 存在

**输入：** `/project-stage-detect`

**预期行为：**
1. Skill 未找到 stage.txt——切换到产物推断模式
2. Skill 找到 GDD（Systems Design 完成）、epic（Pre-Production 完成）、源代码和 sprints（Production 活跃）
3. Skill 推断：Stage = Production
4. 置信度为 MEDIUM（从产物推断，而非来自 stage.txt）
5. Skill 建议运行 `/gate-check` 以正式化并写入 stage.txt

**断言：**
- [ ] 推断的阶段是 Production
- [ ] 置信度为 MEDIUM（不是 HIGH，因为 stage.txt 不存在）
- [ ] 存在运行 `/gate-check` 的建议
- [ ] 该 skill 不写入 stage.txt

---

### 用例 3：无 stage.txt、无文档、无源代码——推断为 Concept

**Fixture：**
- 没有 `production/stage.txt`
- `design/` 目录存在但为空
- `src/` 存在但不包含代码文件
- `technical-preferences.md` 仅有占位符

**输入：** `/project-stage-detect`

**预期行为：**
1. Skill 未找到 stage.txt
2. 产物扫描：无 GDD、无源代码、无 epic、无 sprint、引擎未配置
3. Skill 推断：Stage = Concept
4. 置信度为 MEDIUM
5. Skill 建议 `/start` 以开始 onboarding 工作流

**断言：**
- [ ] 推断的阶段是 Concept
- [ ] 输出列出了被检查的产物（并发现不存在）
- [ ] 建议 `/start` 作为下一步
- [ ] 不写入任何文件

---

### 用例 4：不一致——stage.txt 显示为 Production 但无源代码

**Fixture：**
- `production/stage.txt` 包含 `Production`
- `design/gdd/` 有 GDD 文件
- `src/` 目录存在但不包含源代码文件
- 不存在 sprint 文件

**输入：** `/project-stage-detect`

**预期行为：**
1. Skill 读取 stage.txt——检测到 `Production`
2. 交叉检查发现：无源代码、无 sprint——与 Production 不一致
3. Skill 标记不一致："stage.txt says Production but no source code or sprints found"
4. Skill 报告检测到的阶段为 Production（遵循 stage.txt），但由于产物不匹配，置信度降至 LOW
5. Skill 建议手动审查 stage.txt 或运行 `/gate-check`

**断言：**
- [ ] 输出中明确标记了不一致
- [ ] 当产物与 stage.txt 矛盾时置信度为 LOW
- [ ] stage.txt 值不会被静默覆盖
- [ ] 建议用户手动验证不一致

---

### 用例 5：Director 关卡检查——无 gate；检测是建议性的

**Fixture：**
- 任何有或无 stage.txt 的项目状态

**输入：** `/project-stage-detect`

**预期行为：**
1. Skill 完成完整的阶段检测
2. 任何时候都不派生 director agent
3. 输出中不出现 gate ID
4. 不调用写入工具

**断言：**
- [ ] 未调用 director gate
- [ ] 未调用写入工具
- [ ] 检测输出纯粹是建议性的
- [ ] 裁决命名检测到的阶段而不触发任何 gate

---

## 协议合规性

- [ ] 如果存在则读取 stage.txt；如果不存在则回退到产物推断
- [ ] 始终报告置信度（HIGH / MEDIUM / LOW）
- [ ] 针对产物交叉检查 stage.txt 并标记不一致
- [ ] 不写入 stage.txt（那是 `/gate-check` 的职责）
- [ ] 以适合检测到的阶段的下一步建议结束

---

## 覆盖说明

- Technical Setup 阶段（引擎已配置，尚无 GDD）和 Pre-Production 阶段（GDD 完成，尚无 epic）遵循与用例 2 和 3 相同的产物推断模式，未单独进行 fixture 测试。
- Polish 和 Release 阶段未在此进行 fixture 测试；它们遵循相同的高置信度（stage.txt 存在）或推断逻辑。
- 置信度是建议性的——skill 不会在其上设置任何操作 gate。
