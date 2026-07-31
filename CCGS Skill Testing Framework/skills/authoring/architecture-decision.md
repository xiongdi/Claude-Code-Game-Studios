# Skill Test Spec: /architecture-decision

## Skill Summary

`/architecture-decision` 引导用户逐章节编写新的架构决策记录（ADR）。必需章节为：Status、Context、Decision、Consequences、Alternatives 和 Related ADRs。该 skill 还会将 `docs/engine-reference/` 中的引擎版本参考戳记到 ADR 中以供追溯。

在 `full` 审查模式下，TD-ADR（technical-director）和 LP-FEASIBILITY（lead-programmer）gate agent 在草案完成后派生。如果两个 gate 都返回 APPROVED，ADR 状态被设置为 Accepted。在 `lean` 或 `solo` 模式下，两个 gate 都被跳过，ADR 以 Status: Proposed 写入。该 skill 在编写过程中逐章节询问 "May I write"。ADR 写入 `docs/architecture/adr-NNN-[name].md`。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：ACCEPTED、PROPOSED、CONCERNS
- [ ] 包含 "May I write" 协作协议语言（逐章节批准）
- [ ] 末尾有下一步交接说明
- [ ] 记录 gate 行为：full 模式下为 TD-ADR + LP-FEASIBILITY；lean/solo 下被跳过
- [ ] 记录 ADR 状态为 Accepted（full，gate 批准）或 Proposed（其他情况）
- [ ] 提及来自 `docs/engine-reference/` 的引擎版本戳记

---

## Director Gate Checks

在 `full` 模式下：TD-ADR（technical-director）和 LP-FEASIBILITY（lead-programmer）
在 ADR 草案完成后派生。如果两者都返回 APPROVED，ADR 状态被设置为 Accepted。
如果任一返回 CONCERNS 或 FAIL，ADR 保持 Proposed。

在 `lean` 模式下：两个 gate 都被跳过。ADR 以 Status: Proposed 写入。
输出注明："TD-ADR skipped — lean mode" 和 "LP-FEASIBILITY skipped — lean mode"。

在 `solo` 模式下：两个 gate 都被跳过。ADR 以 Status: Proposed 写入。

---

## Test Cases

### Case 1: Happy Path——新 ADR，渲染方法，full 模式，gate 批准

**Fixture:**
- `docs/architecture/` 存在，无现有渲染 ADR
- `docs/engine-reference/[engine]/VERSION.md` 存在
- `production/session-state/review-mode.txt` 内容为 `full`

**Input:** `/architecture-decision rendering-approach`

**Expected behavior:**
1. Skill 引导用户完成每个必需章节（Status、Context、Decision、Consequences、Alternatives、Related ADRs）
2. 引擎版本从 `docs/engine-reference/` 戳记到 ADR 中
3. 对于每个章节：显示草案，询问 "May I write this section?"，获得批准
4. 所有章节后：TD-ADR 和 LP-FEASIBILITY gate 并行派生
5. 两个 gate 都返回 APPROVED
6. ADR 状态被设置为 Accepted
7. Skill 写入 `docs/architecture/adr-NNN-rendering-approach.md`
8. 如果定义了新的 TR-ID，`docs/architecture/tr-registry.yaml` 被更新

**Assertions:**
- [ ] 所有 6 个必需章节都被编写和写入
- [ ] 引擎版本参考被戳记在 ADR 中
- [ ] TD-ADR 和 LP-FEASIBILITY 并行派生（非顺序）
- [ ] 当 full 模式下两个 gate 都返回 APPROVED 时，ADR 状态为 Accepted
- [ ] 在编写过程中逐章节询问 "May I write"
- [ ] 文件写入 `docs/architecture/adr-NNN-[name].md`

---

### Case 2: Failure Path——TD-ADR 返回 CONCERNS

**Fixture:**
- ADR 草案已完成（所有章节已填写）
- `production/session-state/review-mode.txt` 内容为 `full`
- TD-ADR gate 返回 CONCERNS："The decision does not address [specific concern]"

**Input:** `/architecture-decision [topic]`

**Expected behavior:**
1. TD-ADR gate 派生并返回带有具体反馈的 CONCERNS
2. Skill 向用户展示 concerns
3. ADR 状态保持 Proposed（不是 Accepted）
4. 询问用户：修改决策以解决 concerns，或接受为 Proposed
5. 如果 concerns 未解决，ADR 以 Status: Proposed 写入

**Assertions:**
- [ ] TD-ADR concerns 逐字显示给用户
- [ ] 当 TD-ADR 返回 CONCERNS 时，ADR 状态为 Proposed（不是 Accepted）
- [ ] Skill 在 CONCERNS 未解决时不设置 Status: Accepted
- [ ] 用户被给予修改并重新运行 gate 的选项

---

### Case 3: Lean Mode——两个 gate 都被跳过；ADR 以 Proposed 写入

**Fixture:**
- `production/session-state/review-mode.txt` 内容为 `lean`
- ADR 草案为新技术决策编写

**Input:** `/architecture-decision [topic]`

**Expected behavior:**
1. Skill 引导用户完成所有 6 个章节
2. 草案完成后：TD-ADR 和 LP-FEASIBILITY 都被跳过
3. 输出注明："TD-ADR skipped — lean mode" 和 "LP-FEASIBILITY skipped — lean mode"
4. ADR 以 Status: Proposed 写入（不是 Accepted，因为 gate 未批准）
5. 在最终文件写入前仍询问 "May I write"

**Assertions:**
- [ ] 两个 gate 跳过说明都出现在输出中
- [ ] lean 模式下 ADR 状态为 Proposed（不是 Accepted）
- [ ] 在写入文件前仍询问 "May I write"
- [ ] Skill 在用户批准后写入 ADR

---

### Case 4: Edge Case——此主题的 ADR 已存在

**Fixture:**
- `docs/architecture/` 包含覆盖同一主题的现有 ADR
- 现有 ADR 状态为 Accepted

**Input:** `/architecture-decision [same-topic]`

**Expected behavior:**
1. Skill 检测到覆盖同一主题的现有 ADR
2. Skill 询问："An ADR for [topic] already exists ([filename]). Update it, or create a new superseding ADR?"
3. 用户选择更新或取代
4. Skill 不静默创建重复 ADR

**Assertions:**
- [ ] Skill 在编写开始前检测现有 ADR
- [ ] 用户提供更新或取代选项——无静默重复
- [ ] 如果更新：skill 打开现有 ADR 进行逐章节修订
- [ ] 如果取代：新 ADR 在 Related ADRs 章节中引用被取代的 ADR

---

### Case 5: Director Gate——根据模式和 gate 结果正确设置状态

**Fixture:**
- ADR 草案已完成
- 两种场景：(a) full 模式，两个 gate APPROVED；(b) full 模式，一个 gate CONCERNS

**Full mode, both APPROVED:**
- ADR 状态被设置为 Accepted

**Assertions (both approved):**
- [ ] ADR frontmatter/标题显示 `Status: Accepted`
- [ ] TD-ADR 和 LP-FEASIBILITY 在输出中都显示为 APPROVED

**Full mode, one gate returns CONCERNS:**
- ADR 状态保持 Proposed

**Assertions (CONCERNS):**
- [ ] ADR frontmatter/标题显示 `Status: Proposed`
- [ ] Concerns 在输出中列出
- [ ] Skill 在任何 gate 返回 CONCERNS 时不设置 Status: Accepted

**Lean/solo mode:**
- ADR 状态始终为 Proposed，无论内容质量如何

**Assertions (lean/solo):**
- [ ] lean 模式下 ADR 状态为 Proposed
- [ ] solo 模式下 ADR 状态为 Proposed
- [ ] lean 或 solo 模式下不出现 gate 输出

---

## Protocol Compliance

- [ ] 所有 6 个必需章节在 gate 审查前编写
- [ ] 引擎版本从 `docs/engine-reference/` 戳记到 ADR 中
- [ ] 在编写过程中逐章节询问 "May I write"
- [ ] TD-ADR 和 LP-FEASIBILITY 在 full 模式下并行派生
- [ ] 被跳过的 gate 在 lean/solo 输出中按名称和模式注明
- [ ] ADR 状态为 Accepted 仅当 full 模式且两个 gate 都 APPROVED
- [ ] 以下一步交接说明结束：`/architecture-review` 或 `/create-control-manifest`

---

## Coverage Notes

- ADR 编号（自动递增 NNN）不单独进行 fixture 测试——
  skill 读取现有 ADR 文件名以分配下一个编号。
- Related ADRs 章节链接（supersedes / related-to）通过 Case 4 进行结构性测试，
  但并非所有链接类型都单独验证。
- TR-registry 更新（当 ADR 中定义新的 TR-ID 时）是写入阶段的一部分——
  通过 Case 1 隐式测试。
