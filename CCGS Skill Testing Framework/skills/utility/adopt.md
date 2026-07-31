# Skill Test Spec: /adopt

## Skill Summary

`/adopt` 审计现有项目的资源——GDD、ADR、story、基础设施文件和 `technical-preferences.md`——
检查其格式是否符合模板的 skill 管道。它将每个差距按严重性分类（BLOCKING / HIGH / MEDIUM / LOW），
编写编号的有序迁移计划，并在通过 `AskUserQuestion` 获得用户明确批准后写入 `docs/adoption-plan-[date].md`。

此 skill 与 `/project-stage-detect`（检查存在什么）不同。
`/adopt` 检查存在的内容是否实际能与模板的 skill 一起工作。

不适用 director gate。该 skill 不调用任何 director agent。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含严重性层级关键词：BLOCKING、HIGH、MEDIUM、LOW
- [ ] 在写入采用计划前包含 "May I write" 或 `AskUserQuestion` 语言
- [ ] 末尾有下一步交接说明（例如，提供立即修复最高优先级差距的选项）

---

## Director Gate Checks

无。`/adopt` 是 brownfield 审计实用工具。不适用 director gate。

---

## Test Cases

### Case 1: Happy Path——所有 GDD 合规，无差距，COMPLIANT

**Fixture:**
- `design/gdd/` 包含 3 个 GDD 文件；每个都有所有 8 个必需章节并包含内容
- `docs/architecture/adr-0001.md` 存在，具有 `## Status`、`## Engine Compatibility`
  和所有其他必需章节
- `production/stage.txt` 存在
- `docs/architecture/tr-registry.yaml` 和 `docs/architecture/control-manifest.md` 存在
- 引擎在 `technical-preferences.md` 中配置

**Input:** `/adopt`

**Expected behavior:**
1. Skill 发出 "Scanning project artifacts..." 然后静默读取所有资源
2. 报告检测到的阶段、GDD 数量、ADR 数量、story 数量
3. Phase 2 审计：所有 3 个 GDD 都有所有 8 个章节，Status 字段存在且有效
4. ADR 审计：所有必需章节存在
5. 基础设施审计：所有关键文件存在
6. Phase 3：零 BLOCKING、零 HIGH、零 MEDIUM、零 LOW 差距
7. 总结报告："No blocking gaps — this project is template-compatible"
8. 使用 `AskUserQuestion` 询问关于写入计划；用户选择写入
9. 采用计划写入 `docs/adoption-plan-[date].md`
10. Phase 7 提供下一步操作：无阻塞差距，提供下一步选项

**Assertions:**
- [ ] Skill 在呈现任何输出前静默读取
- [ ] "Scanning project artifacts..." 出现在静默读取阶段之前
- [ ] 差距计数显示 0 BLOCKING、0 HIGH、0 MEDIUM（或仅 LOW）
- [ ] 在写入采用计划前使用 `AskUserQuestion`
- [ ] 采用计划文件写入 `docs/adoption-plan-[date].md`
- [ ] Phase 7 提供具体的下一步操作（不仅仅是列表）

---

### Case 2: Non-Compliant Documents——GDD 缺少章节，NEEDS MIGRATION

**Fixture:**
- `design/gdd/` 包含 2 个 GDD 文件：
  - `combat.md` — 缺少 `## Acceptance Criteria` 和 `## Formulas` 章节
  - `movement.md` — 所有 8 个章节存在
- 一个 ADR（`adr-0001.md`）缺少 `## Status` 章节
- `docs/architecture/tr-registry.yaml` 不存在

**Input:** `/adopt`

**Expected behavior:**
1. Skill 扫描所有资源
2. Phase 2 审计发现：
   - `combat.md`：2 个缺失章节（Acceptance Criteria、Formulas）
   - `adr-0001.md`：缺少 `## Status`——BLOCKING 影响
   - `tr-registry.yaml`：缺失——HIGH 影响
3. Phase 3 分类：
   - BLOCKING：`adr-0001.md` 缺少 `## Status`（story-readiness 静默通过）
   - HIGH：`tr-registry.yaml` 缺失；`combat.md` 缺少 Acceptance Criteria（无法生成 story）
   - MEDIUM：`combat.md` 缺少 Formulas
4. Phase 4 构建有序迁移计划：
   - Step 1 (BLOCKING)：向 `adr-0001.md` 添加 `## Status`——命令：`/architecture-decision retrofit`
   - Step 2 (HIGH)：运行 `/architecture-review` 引导 tr-registry.yaml
   - Step 3 (HIGH)：向 `combat.md` 添加 Acceptance Criteria——命令：`/design-system retrofit`
   - Step 4 (MEDIUM)：向 `combat.md` 添加 Formulas
5. Gap Preview 将 BLOCKING 项目显示为项目符号（实际文件名），HIGH/MEDIUM 显示为计数
6. `AskUserQuestion` 询问写入计划；批准后写入
7. Phase 7 提供立即修复最高优先级差距（ADR Status）的选项

**Assertions:**
- [ ] BLOCKING 差距在 Gap Preview 中列为明确的文件名项目符号
- [ ] HIGH 和 MEDIUM 在 Gap Preview 中显示为计数
- [ ] 迁移计划项目按 BLOCKING 优先顺序排列
- [ ] 每个计划项目包含修复命令或手动步骤
- [ ] 在写入前使用 `AskUserQuestion`
- [ ] Phase 7 提供立即改造第一个 BLOCKING 项目的选项

---

### Case 3: Mixed State——部分文档合规，部分不合规，部分报告

**Fixture:**
- 4 个 GDD 文件：2 个完全合规，2 个有差距（一个缺少 Tuning Knobs，一个缺少 Edge Cases）
- ADR：3 个文件——2 个合规，1 个缺少 `## ADR Dependencies`
- Story：5 个文件——3 个有 TR-ID 引用，2 个没有
- 基础设施：所有关键文件存在；`technical-preferences.md` 完全配置

**Input:** `/adopt`

**Expected behavior:**
1. Skill 审计所有资源类型
2. 审计总结显示总数："4 GDDs (2 fully compliant, 2 with gaps); 3 ADRs
   (2 fully compliant, 1 with gaps); 5 stories (3 with TR-IDs, 2 without)"
3. 差距分类：
   - 无 BLOCKING 差距
   - HIGH：1 个 ADR 缺少 `## ADR Dependencies`
   - MEDIUM：2 个 GDD 缺少章节；2 个 story 缺少 TR-ID
   - LOW：无
4. 迁移计划先列出 HIGH 差距，然后按顺序列出 MEDIUM 差距
5. 包含说明："Existing stories continue to work — do not regenerate stories
   that are in progress or done"
6. `AskUserQuestion` 写入计划；批准后写入

**Assertions:**
- [ ] 显示每个资源的合规计数（N 个合规，M 个有差距）
- [ ] 现有 story 兼容性说明包含在计划中
- [ ] 无 BLOCKING 差距导致迁移计划中无 BLOCKING 章节
- [ ] HIGH 差距在计划排序中先于 MEDIUM 差距
- [ ] 在写入前使用 `AskUserQuestion`

---

### Case 4: No Artifacts Found——新项目，指导运行 /start

**Fixture:**
- 仓库在 `design/gdd/`、`docs/architecture/`、`production/epics/` 中无文件
- `production/stage.txt` 不存在
- `src/` 目录不存在或少于 10 个文件
- 无 game-concept.md，无 systems-index.md

**Input:** `/adopt`

**Expected behavior:**
1. Phase 1 存在检查未发现资源
2. Skill 推断 "Fresh"——无 brownfield 工作可迁移
3. 使用 `AskUserQuestion`：
   - "This looks like a fresh project — no existing artifacts found. `/adopt` is for
     projects with work to migrate. What would you like to do?"
   - 选项："Run `/start`"、"My artifacts are in a non-standard location"、"Cancel"
4. Skill 停止——无论用户选择如何都不继续审计

**Assertions:**
- [ ] 在未找到资源时使用 `AskUserQuestion`（不是纯文本消息）
- [ ] `/start` 作为命名选项呈现
- [ ] Skill 在问题后停止——不运行审计阶段
- [ ] 不写入采用计划文件

---

### Case 5: Director Gate Check——无 gate；adopt 是实用审计 skill

**Fixture:**
- 具有合规和非合规 GDD 混合的项目

**Input:** `/adopt`

**Expected behavior:**
1. Skill 完成完整审计并生成迁移计划
2. 在任何时候都不派生 director agent
3. 输出中不出现 gate ID（CD-*、TD-*、AD-*、PR-*）
4. 在 skill 运行期间不调用 `/gate-check`

**Assertions:**
- [ ] 不调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] Skill 无需任何 gate 裁定即达到计划编写或取消

---

## Protocol Compliance

- [ ] 在静默读取阶段前发出 "Scanning project artifacts..."
- [ ] 在呈现任何结果前静默读取所有资源
- [ ] 在请求写入前显示 Adoption Audit Summary 和 Gap Preview
- [ ] 在写入采用计划文件前使用 `AskUserQuestion`
- [ ] 采用计划写入 `docs/adoption-plan-[date].md`——不写入任何其他路径
- [ ] 迁移计划项目排序：BLOCKING 优先，HIGH 第二，MEDIUM 第三，LOW 最后
- [ ] Phase 7 始终提供单一具体的下一步操作（不是通用列表）
- [ ] 永远不重新生成现有资源——仅填补存在内容中的差距
- [ ] 在任何时候都不调用 director gate

---

## Coverage Notes

- `gdds`、`adrs`、`stories` 和 `infra` 参数模式缩小审计范围；
  每个都遵循与完整审计相同的模式但限于该资源类型。
  不在此单独进行 fixture 测试。
- systems-index.md 括号状态值检查（BLOCKING）是一个特殊情况，
  在写入计划前触发立即修复提供；不单独测试。
- review-mode.txt 提示（Phase 6b）在计划写入后运行，如果 `production/review-mode.txt`
  不存在；不在此单独测试。
