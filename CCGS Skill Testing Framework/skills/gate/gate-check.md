# Skill 测试规格：/gate-check

## Skill 摘要

`/gate-check` 验证项目是否已准备好进入下一个开发阶段。
它检查必需的产物，运行质量检查，就不可验证的项目询问用户，
并生成 PASS/CONCERNS/FAIL 判定。在 PASS 且用户确认后，
它将新阶段名称写入 `production/stage.txt`。它管理所有 6 个阶段转换，
是管道中最关键的关卡守护 skill。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题（编号 Phase N 或 ## 部分）
- [ ] 包含判定关键词：PASS、CONCERNS、FAIL
- [ ] 包含"可以写入吗"协作协议语言
- [ ] 末尾有下一步交接（Follow-Up Actions 部分）

---

## 测试用例

### 用例 1：正常路径——所有 Concept 产物存在，推进到 Systems Design

**Fixture：**
- `design/gdd/game-concept.md` 存在，包含所有必需部分的内容
- `design/gdd/game-pillars.md` 存在（或支柱在概念文档内定义）
- 尚无 systems index（此阶段正确）

**输入：** `/gate-check systems-design`

**预期行为：**
1. Skill 读取 `design/gdd/game-concept.md` 并验证其有内容
2. Skill 检查游戏支柱（在概念中或单独文件）
3. Skill 检查质量项目（核心循环已描述、目标受众已识别）
4. Skill 输出结构化检查清单，所有项目已标记
5. Skill 展示 PASS/CONCERNS/FAIL 判定
6. 如果 PASS：skill 询问"可以将 `production/stage.txt` 更新为 'Systems Design' 吗？"

**断言：**
- [ ] Skill 使用 Glob 或 Read 在标记为已检查前验证 `design/gdd/game-concept.md` 存在
- [ ] 输出包含"Required Artifacts"部分，每个项目有检查状态
- [ ] 输出包含"Quality Checks"部分，每个项目有检查状态
- [ ] 输出包含"Verdict"行，为 PASS / CONCERNS / FAIL 之一
- [ ] Skill 就不可验证的质量项目询问用户（例如，"Has this been reviewed?"）而不是假设 PASS
- [ ] Skill 在更新 `production/stage.txt` 前询问"可以写入吗"
- [ ] Skill 未经用户明确确认不写入 `production/stage.txt`

---

### 用例 2：失败路径——缺少 Concept → Systems Design 的必需产物

**Fixture：**
- `design/gdd/game-concept.md` 不存在
- 无游戏支柱文档
- `design/gdd/` 目录为空或不存在

**输入：** `/gate-check systems-design`

**预期行为：**
1. Skill 尝试读取 `design/gdd/game-concept.md`——文件未找到
2. Skill 将必需产物标记为缺失（不存在）
3. Skill 输出 FAIL 判定
4. Skill 列出阻塞项："No game concept document found"
5. Skill 建议修复方案：运行 `/brainstorm` 创建一个

**断言：**
- [ ] 当缺少必需产物时判定为 FAIL（不是 PASS 或 CONCERNS）
- [ ] 输出明确命名 `design/gdd/game-concept.md` 为缺失
- [ ] 输出包含"Blockers"部分，至少有 1 个项目
- [ ] 输出推荐 `/brainstorm` 作为修复操作
- [ ] 当判定为 FAIL 时，skill 不写入 `production/stage.txt`

---

### 用例 3：无参数——自动检测当前阶段

**Fixture：**
- `production/stage.txt` 包含 `Concept`
- `design/gdd/game-concept.md` 存在且有内容
- 尚无 systems index

**输入：** `/gate-check`（无参数）

**预期行为：**
1. Skill 读取 `production/stage.txt` 以确定当前阶段
2. Skill 确定下一个关卡是 Concept → Systems Design
3. Skill 继续执行 Systems Design 关卡检查
4. 输出清楚地说明正在验证哪个转换

**断言：**
- [ ] Skill 读取 `production/stage.txt`（或使用 project-stage-detect 启发式方法）以确定当前阶段
- [ ] 输出头部命名当前和目标阶段（例如，"Gate Check: Concept → Systems Design"）
- [ ] 如果当前阶段可确定，skill 不询问用户要检查哪个关卡

---

### 用例 4：边缘情况——手动检查项目正确标记

**Fixture：**
- Concept → Systems Design 的所有必需产物都存在
- 无 playtest 或审查记录（无法自动验证质量检查）

**输入：** `/gate-check systems-design`

**预期行为：**
1. Skill 验证所有产物文件存在
2. Skill 遇到质量检查："Game concept reviewed (not MAJOR REVISION NEEDED)"
3. 由于无审查记录，skill 将项目标记为 MANUAL CHECK NEEDED
4. Skill 询问用户："Has the game concept been reviewed for design quality?"
5. Skill 在最终确定判定前等待用户输入

**断言：**
- [ ] 无法自动验证的项目标记为 `[?] MANUAL CHECK NEEDED` 而不是假设 PASS
- [ ] Skill 对至少一个不可验证的质量项目使用向用户提问
- [ ] Skill 默认不将不可验证的项目标记为 PASS

---

### 用例 5：Director 关卡——lean vs full vs solo 模式

**Fixture：**
- `production/session-state/review-mode.txt` 存在（或等效状态文件）
- 目标关卡的所有必需产物都存在
- `design/gdd/game-concept.md` 存在

**用例 5a——full 模式：**
- `review-mode.txt` 包含 `full`

**输入：** `/gate-check systems-design`（full 模式激活）

**预期行为：**
1. Skill 读取审查模式——确定为 `full`
2. Skill 并行派生所有 4 个 PHASE-GATE director 提示：
   - CD-PHASE-GATE（creative-director）
   - TD-PHASE-GATE（technical-director）
   - PR-PHASE-GATE（producer）
   - AD-PHASE-GATE（art-director）
3. 如果一个 director 返回 CONCERNS → 整体关卡判定至少为 CONCERNS
4. 在生成最终输出前收集所有 4 个判定

**断言（5a）：**
- [ ] Skill 在决定派生哪些 director 之前读取审查模式
- [ ] 派生所有 4 个 PHASE-GATE director 提示（不只是 1 或 2 个）
- [ ] Director 并行派生（同时，不是顺序）
- [ ] 来自任一个 director 的 CONCERNS 判定传播到整体判定
- [ ] 如果有任何 director 返回 CONCERNS 或 REJECT，判定不自动为 PASS

**用例 5b——solo 模式：**
- `review-mode.txt` 包含 `solo`

**输入：** `/gate-check systems-design`（solo 模式激活）

**预期行为：**
1. Skill 读取审查模式——确定为 `solo`
2. 每个 director 被注明为跳过："[CD-PHASE-GATE] skipped——Solo mode"
3. 关卡判定仅从产物/质量检查得出
4. 不派生 director 关卡

**断言（5b）：**
- [ ] solo 模式下不派生 director 关卡
- [ ] 每个跳过的关卡在输出中明确注明："[GATE-ID] skipped——Solo mode"
- [ ] 判定仅基于产物和质量检查

**关于用例 3 修正的说明：**
用例 3 的断言先前声明"如果当前阶段可确定，skill 不询问用户要检查哪个关卡。"
这是正确的。然而，skill 确实使用 AskUserQuestion 在运行完整检查之前确认自动检测到的转换——
这是一个确认步骤，不是关卡选择。用例 3 的断言不应将此确认视为失败。

---

## 协议合规性

- [ ] 更新 `production/stage.txt` 前使用"可以写入吗"
- [ ] 请求写入批准前展示完整检查清单报告
- [ ] 以"Follow-Up Actions"部分结束，列出每个判定对应的下一步
- [ ] 未经用户明确确认绝不推进阶段
- [ ] 如果 `production/stage.txt` 不存在，绝不自动创建（不询问）

---

## 覆盖说明

- Production → Polish 和 Polish → Release 关卡不在此处覆盖，
  因为它们需要复杂的多产物设置（sprint 计划、playtest 数据、QA 签署）；
  这些被延期到专门的后续规格。
- "CONCERNS"判定路径（小间隙，不阻塞）未在此处明确测试；
  它介于用例 1 和用例 2 之间，遵循相同模式。
- Vertical Slice 验证块（Pre-Production → Production 关卡）不覆盖，
  因为它需要可玩的构建上下文，无法表示为文档 fixture。
