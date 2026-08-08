# 技能测试规格: /architecture-review

## 技能摘要

`/architecture-review` 是一个 Opus 层级的 skill，用于验证技术架构文档是否符合项目要求的 8 个架构章节，并检查其内部一致性、与现有 ADR 的一致性，以及是否正确针对已锁定的引擎版本。它产生 APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED 的裁定。

在 `full` 审查模式下，该 skill 会并行派生两个主管 gate agent：TD-ARCHITECTURE（technical-director）和 LP-FEASIBILITY（lead-programmer）。在 `lean` 或 `solo` 模式下，两个 gate 都会被跳过并注明。该 skill 是只读的——不写入任何文件。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：APPROVED、NEEDS REVISION、MAJOR REVISION NEEDED
- [ ] 不要求 "May I write" 语言（只读 skill）
- [ ] 末尾有下一步交接说明
- [ ] 记录 gate 行为：full 模式下为 TD-ARCHITECTURE + LP-FEASIBILITY；lean/solo 下被跳过

---

## 导演门控检查

在 `full` 模式下：TD-ARCHITECTURE（technical-director）和 LP-FEASIBILITY（lead-programmer）在 skill 读取架构文档后并行派生。

在 `lean` 模式下：两个 gate 都被跳过。输出注明：
"TD-ARCHITECTURE skipped — lean mode" 和 "LP-FEASIBILITY skipped — lean mode"。

在 `solo` 模式下：两个 gate 都被跳过，并附有同等说明。

---

## 测试用例

### Case 1: Happy Path——完整架构文档，full 模式

**Fixture:**
- `docs/architecture/architecture.md` 存在，所有 8 个必需章节均已填写
- 所有章节引用了 `docs/engine-reference/` 中正确的引擎版本
- 与 `docs/architecture/` 中现有 Accepted ADR 无矛盾
- `production/session-state/review-mode.txt` 内容为 `full`

**Input:** `/architecture-review docs/architecture/architecture.md`

**Expected behavior:**
1. Skill 读取架构文档
2. Skill 读取现有 ADR 进行交叉引用
3. Skill 读取引擎版本参考
4. TD-ARCHITECTURE 和 LP-FEASIBILITY gate agent 并行派生
5. 两个 gate 都返回 APPROVED
6. Skill 输出逐章节完整性检查（8/8 章节存在）
7. 裁定：APPROVED

**Assertions:**
- [ ] 所有 8 个必需章节都被检查并报告
- [ ] TD-ARCHITECTURE 和 LP-FEASIBILITY 并行派生（非顺序）
- [ ] 当所有章节存在且无矛盾时，裁定为 APPROVED
- [ ] Skill 不写入任何文件
- [ ] 存在到 `/create-control-manifest` 或 `/create-epics` 的下一步交接说明

---

### Case 2: Failure Path——缺少必需章节

**Fixture:**
- `docs/architecture/architecture.md` 存在但缺少至少 2 个必需章节
  （例如，无数据模型章节、无错误处理章节）
- `production/session-state/review-mode.txt` 内容为 `full`

**Input:** `/architecture-review docs/architecture/architecture.md`

**Expected behavior:**
1. Skill 读取文档并识别缺失章节
2. 章节完整性显示少于 8/8 章节存在
3. 缺失章节按名称列出，并附带具体修复指导
4. 裁定：MAJOR REVISION NEEDED（≥2 个缺失章节）

**Assertions:**
- [ ] 裁定为 MAJOR REVISION NEEDED（不是 APPROVED 或 NEEDS REVISION），当 ≥2 个章节缺失时
- [ ] 每个缺失章节在输出中明确命名
- [ ] 修复指导具体（说明要添加什么，而非仅"添加缺失章节"）
- [ ] Skill 不通过缺少必需章节的文档

---

### Case 3: Partial Path——架构与现有 ADR 矛盾

**Fixture:**
- `docs/architecture/architecture.md` 存在，所有 8 个章节均存在
- `docs/architecture/` 中的一个 Accepted ADR 建立了架构文档所矛盾的约束
  （例如，ADR-001 要求 ECS 模式；architecture.md 为同一系统描述了不同模式）

**Input:** `/architecture-review docs/architecture/architecture.md`

**Expected behavior:**
1. Skill 读取架构文档和所有现有 ADR
2. 检测到架构文档与指定 ADR 之间的矛盾
3. 矛盾条目命名：ADR 编号/标题、矛盾章节和影响
4. 裁定：NEEDS REVISION（存在矛盾，但结构其他方面健全）

**Assertions:**
- [ ] 裁定为 NEEDS REVISION（不是 MAJOR REVISION NEEDED），针对单一矛盾
- [ ] 矛盾条目中命名了具体的 ADR 编号和标题
- [ ] 识别了两个文档中的矛盾章节
- [ ] Skill 不自动解决矛盾

---

### Case 4: Edge Case——文件未找到

**Fixture:**
- 提供的路径在项目中不存在

**Input:** `/architecture-review docs/architecture/nonexistent.md`

**Expected behavior:**
1. Skill 尝试读取文件
2. 文件未找到
3. Skill 输出明确的错误，命名缺失的文件
4. Skill 建议检查 `docs/architecture/` 或运行 `/create-architecture`
5. Skill 不产生裁定

**Assertions:**
- [ ] Skill 在文件未找到时输出明确错误
- [ ] 不产生裁定（APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED）
- [ ] Skill 建议纠正操作
- [ ] Skill 不崩溃或不产生部分报告

---

### Case 5: Director Gate——full 模式派生两个 gate；solo 模式跳过两个

**Fixture (full 模式):**
- `docs/architecture/architecture.md` 存在，具有所有 8 个章节
- `production/session-state/review-mode.txt` 内容为 `full`

**Full mode expected behavior:**
1. TD-ARCHITECTURE gate 派生
2. LP-FEASIBILITY gate 与 TD-ARCHITECTURE 并行派生
3. 两个 gate 在裁定发布前完成

**Assertions (full 模式):**
- [ ] TD-ARCHITECTURE 和 LP-FEASIBILITY 在输出中都作为已完成的 gate 出现
- [ ] 两个 gate 并行派生（非一个接一个）
- [ ] 裁定反映 gate 反馈

**Fixture (solo 模式):**
- 同一架构文档
- `production/session-state/review-mode.txt` 内容为 `solo`

**Solo mode expected behavior:**
1. Skill 读取架构文档
2. Gate 不被派生
3. 输出注明："TD-ARCHITECTURE skipped — solo mode" 和 "LP-FEASIBILITY skipped — solo mode"
4. 裁定仅基于结构性检查

**Assertions (solo 模式):**
- [ ] TD-ARCHITECTURE 和 LP-FEASIBILITY 都不作为活动 gate 出现
- [ ] 两个被跳过的 gate 在输出中被注明
- [ ] 裁定仍仅基于结构性检查产生

---

## Protocol Compliance

- [ ] 不写入任何文件（只读 skill）
- [ ] 在发布裁定前呈现章节完整性检查
- [ ] TD-ARCHITECTURE 和 LP-FEASIBILITY 在 full 模式下并行派生
- [ ] 被跳过的 gate 在 lean/solo 输出中按名称和模式注明
- [ ] 裁定严格为以下之一：APPROVED、NEEDS REVISION、MAJOR REVISION NEEDED
- [ ] 以适合裁定的下一步交接说明结束

---

## 覆盖说明

- 8 个必需架构章节是项目特定的；测试使用 skill 正文中定义的章节列表——不在此重新列举。
- 引擎版本兼容性检查（交叉引用 `docs/engine-reference/`）是 Case 1 happy path 的一部分，但不单独进行 fixture 测试。
- RTM（requirement traceability matrix）模式是一个独立问题，由 `/architecture-review` skill 自身的 `rtm` 参数模式覆盖，不在此测试。
