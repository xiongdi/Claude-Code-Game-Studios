# 技能测试规格: /day-one-patch

## 技能摘要

`/day-one-patch` 为发布时已知但从 v1.0 版本推迟的问题准备一个 day-one 补丁计划。它读取 `production/bugs/` 中的未解决 bug 报告、story 文件中的推迟验收标准（标记为 `Status: Done` 但带有推迟 AC 注释的 story），并生成一个按优先级排序的补丁计划，包含每个问题的预计修复时间。

补丁计划在 "May I write" 询问后写入 `production/releases/day-one-patch.md`。如果发现 P0（发布后关键）问题，该技能会触发运行 `/hotfix` 的指导，优先于补丁。不适用 director gate。裁决始终为 COMPLETE。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：COMPLETE
- [ ] 在写入计划前包含 "May I write" 协作协议语言
- [ ] 具有下一步交接（例如，针对 P0 问题的 `/hotfix`，或后续的 `/release-checklist`）

---

## 导演门控检查

无。`/day-one-patch` 是一个发布计划工具。不适用 director gate。

---

## 测试用例

### Case 1: Happy Path — 3 Known Issues, Patch Plan With Fix Estimates

**Fixture:**
- `production/bugs/` 包含 3 个具有严重性级别的未解决 bug：1 个 MEDIUM、2 个 LOW
- Sprint story 中没有推迟的 AC
- 所有 bug 都有复现步骤和系统识别

**Input:** `/day-one-patch`

**Expected behavior:**
1. Skill 读取所有 3 个未解决 bug
2. Skill 分配修复工作量估计：MEDIUM bug = 1-2 天，LOW bug = 各 4 小时
3. Skill 生成优先处理 MEDIUM bug 的补丁计划
4. 计划包括：优先级顺序、预计时间线、负责系统、修复描述
5. Skill 询问 "May I write to `production/releases/day-one-patch.md`?"
6. 文件已写入；裁决为 COMPLETE

**Assertions:**
- [ ] 所有 3 个 bug 都出现在计划中
- [ ] Bug 按严重性排序（MEDIUM 在 LOW 之前）
- [ ] 每个问题都提供了修复估计
- [ ] 在写入前询问 "May I write"
- [ ] 裁决为 COMPLETE

---

### Case 2: Critical Issue Discovered Post-Ship — P0, Triggers /hotfix Guidance

**Fixture:**
- 在 v1.0 发布后在 `production/bugs/` 中发现 CRITICAL 严重性 bug
- 该 bug 导致所有存档文件的数据丢失

**Input:** `/day-one-patch`

**Expected behavior:**
1. Skill 读取 bug 并识别 CRITICAL 严重性问题
2. Skill 升级："P0 ISSUE DETECTED — data loss bug requires immediate hotfix
   before patch planning can proceed"
3. Skill 不将 P0 问题包含在补丁计划时间线中
4. Skill 明确指示："Run `/hotfix` to resolve this issue first"
5. 发出 P0 指导后：仍生成并写入剩余低严重性 bug 的计划；裁决为 COMPLETE

**Assertions:**
- [ ] P0 升级消息在补丁计划前突出显示
- [ ] 针对 P0 问题明确指示 `/hotfix`
- [ ] P0 问题不排在补丁计划时间线中（需要立即行动）
- [ ] 非 P0 问题仍被计划；裁决为 COMPLETE

---

### Case 3: Deferred AC From Story-Done — Pulled Into Patch Plan Automatically

**Fixture:**
- `production/sprints/sprint-008.md` 有一个 `Status: Done` 的 story 并附带注释：
  "DEFERRED AC: Gamepad vibration on damage — deferred to post-launch patch"
- 同一系统没有未解决的 bug

**Input:** `/day-one-patch`

**Expected behavior:**
1. Skill 读取 sprint story 并检测到推迟的 AC 注释
2. 推迟的 AC 自动作为工作项包含在补丁计划中
3. 计划条目："Deferred from sprint-008: Gamepad vibration on damage"
4. 分配修复估计；补丁计划在 "May I write" 批准后写入
5. 裁决为 COMPLETE

**Assertions:**
- [ ] Story 文件中的推迟 AC 自动被拉入计划
- [ ] 推迟项按其来源 story（sprint-008）标记
- [ ] 推迟 AC 获得像 bug 条目一样的修复估计
- [ ] 裁决为 COMPLETE

---

### Case 4: No Known Issues — Empty Plan With Template Note

**Fixture:**
- `production/bugs/` 为空
- 没有 story 有推迟的 AC

**Input:** `/day-one-patch`

**Expected behavior:**
1. Skill 读取 bug——未找到
2. Skill 读取 story 推迟的 AC——未找到
3. Skill 生成一个带有注释的空补丁计划："No known issues at launch"
4. 保留模板结构（标题完整）以供将来使用
5. Skill 询问 "May I write to `production/releases/day-one-patch.md`?"
6. 文件已写入；裁决为 COMPLETE

**Assertions:**
- [ ] "No known issues at launch" 注释出现在写入的文件中
- [ ] 空计划中存在模板标题
- [ ] 当没有问题需要计划时 Skill 不会出错
- [ ] 裁决为 COMPLETE

---

### Case 5: Director Gate Check — No gate; day-one-patch is a planning utility

**Fixture:**
- production/bugs/ 中存在已知问题

**Input:** `/day-one-patch`

**Expected behavior:**
1. Skill 生成并写入补丁计划
2. 不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 无需任何 gate 检查即可达到 COMPLETE

---

## 协议合规

- [ ] 在生成计划前从 `production/bugs/` 读取未解决的 bug
- [ ] 扫描 story 文件中的推迟 AC 注释
- [ ] 用明确的 `/hotfix` 指导升级 CRITICAL（P0）bug
- [ ] 当不存在问题时生成带有注释的空计划（不是错误）
- [ ] 在写入前询问 "May I write to `production/releases/day-one-patch.md`?"
- [ ] 所有路径的裁决都为 COMPLETE

---

## 覆盖说明

- 存在多个 CRITICAL bug 的情况与 Case 2 处理方式相同；所有 P0 问题一起升级。
- 补丁的时间线估计（例如，"patch available in 3 天"）需要手动 QA 和构建时间估计；技能基于严重性使用粗略估计，而不是实际团队速度。
- 补丁说明玩家沟通文档（`/patch-notes`）是一个独立的技能，在补丁计划执行后调用。
