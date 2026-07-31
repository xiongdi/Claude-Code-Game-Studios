# Skill 测试规格：/story-done

## Skill 摘要

`/story-done` 在设计和实现之间闭环。在实现 story 的末尾运行，
它读取 story 文件并根据实现验证每个验收标准。它检查 GDD 和 ADR 偏差，
提示代码审查，将 story 状态更新为 `Complete`，记录任何 tech debt，
并从 sprint 中展示下一个就绪的 story。它生成 COMPLETE / COMPLETE WITH NOTES / BLOCKED
判定，并写入 story 文件以及可选地写入 `docs/tech-debt-register.md`。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥5 个阶段标题（复杂 skill，如适用需要 `context: fork`）
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含"可以写入吗"协作协议语言（写入 story 文件和技术债务注册表）
- [ ] 有下一步交接（从 sprint 中展示下一个 story）

---

## 测试用例

### 用例 1：正常路径——所有验收标准满足，无偏差

**Fixture：**
- Story 文件位于 `production/epics/core/story-light-pickup.md`，包含：
  - 3 个验收标准，均按描述实现
  - `TR-ID: TR-light-001` 引用 GDD 需求
  - `ADR: docs/architecture/adr-003-inventory.md`（Accepted）
  - `Status: In Progress`
- Story 中列出的实现文件存在于 `src/` 中
- TR-light-001 的 GDD 需求文本与功能的实现方式匹配
- 遵循了 ADR 指导（无偏差）

**输入：** `/story-done production/epics/core/story-light-pickup.md`

**预期行为：**
1. Skill 读取 story 文件并提取所有关键字段
2. Skill 从 `tr-registry.yaml` 重新读取 GDD 需求（不是从 story 的引用文本）
3. Skill 读取引用的 ADR 以了解实现约束
4. Skill 评估每个验收标准（尽可能自动，不能则手动提示）
5. Skill 检查 GDD 需求偏差
6. Skill 检查 ADR 指导偏差
7. Skill 提示用户："Please provide the code review outcome for this story"
8. Skill 展示 COMPLETE 判定
9. Skill 询问"可以将 story Status 更新为 Complete 并添加 Completion Notes 吗？"
10. 如果同意：skill 更新 story 文件
11. Skill 从 sprint 中展示下一个 `Ready for Dev` 的 story

**断言：**
- [ ] Skill 读取 `docs/architecture/tr-registry.yaml` 获取 TR-ID 需求文本（不只是 story）
- [ ] Skill 读取引用的 ADR 文件（不只是 story 引用）
- [ ] 每个验收标准都列出 VERIFIED / DEFERRED / FAILED 状态
- [ ] Skill 提示用户提供代码审查结果（不跳过此步骤）
- [ ] 当所有标准已验证且无偏差时判定为 COMPLETE
- [ ] Skill 在更新 story 文件前询问"可以写入吗"
- [ ] Skill 未经用户确认不会自动更新 story 状态
- [ ] 完成后，skill 从 `production/sprints/` 中展示下一个就绪的 story

---

### 用例 2：阻塞路径——验收标准无法验证

**Fixture：**
- Story 文件有一个验收标准："Player sees correct animation on pickup"
- 此标准无自动化测试
- 未执行手动验证
- 所有其他标准均满足

**输入：** `/story-done production/epics/core/story-light-pickup.md`

**预期行为：**
1. Skill 处理所有验收标准
2. 到达动画标准——无法自动验证
3. Skill 询问用户："Acceptance criterion 'Player sees correct animation on
   pickup' cannot be auto-verified. Has this been manually tested?"
4. 如果用户说否：标准标记为 DEFERRED，判定变为 COMPLETE WITH NOTES
5. Skill 在完成记录中记录被延期的标准
6. 询问"可以写入更新了延期标准的 story 吗？"

**断言：**
- [ ] Skill 对无法验证的标准询问用户，而不是假设 PASS
- [ ] 延期标准导致 COMPLETE WITH NOTES（不是 COMPLETE 或 BLOCKED）
- [ ] 延期标准在完成记录中明确命名
- [ ] Skill 在更新 story 文件前仍询问"可以写入吗"

---

### 用例 3：阻塞路径——检测到 GDD 偏差

**Fixture：**
- Story TR-ID 指向需求："Player can carry max 3 light sources"
- `src/` 中的实现使用变量 `MAX_CARRIED_LIGHTS = 5`
- 这是对 GDD 的故意偏差

**输入：** `/story-done production/epics/core/story-light-pickup.md`

**预期行为：**
1. Skill 读取 GDD 需求文本（max 3）
2. Skill 检测到需求与实现值（5）之间的差异
3. Skill 将其标记为 GDD 偏差并要求用户分类：
   - INTENTIONAL：记录偏差和原因
   - ERROR：在 story 标记为 Complete 之前必须修复实现
   - OUT OF SCOPE：需求已更改，GDD 需要更新
4. 如果 INTENTIONAL：skill 在完成记录中记录偏差，判定为 COMPLETE WITH NOTES
5. 如果 ERROR：判定为 BLOCKED 直到实现被纠正

**断言：**
- [ ] Skill 检测到 GDD 需求与实现值之间的不匹配
- [ ] Skill 要求用户分类偏差（不自动假设任一方）
- [ ] INTENTIONAL 偏差 → COMPLETE WITH NOTES（不是 BLOCKED）
- [ ] ERROR 偏差 → BLOCKED 判定直到修复
- [ ] 检测到的偏差记录在完成记录或技术债务注册表中

---

### 用例 4：边缘情况——无参数，自动检测当前 story

**Fixture：**
- `production/session-state/active.md` 包含对
  `production/epics/core/story-oxygen-drain.md` 作为活动 story 的引用
- 该 story 文件存在，状态为 `Status: In Progress`

**输入：** `/story-done`（无参数）

**预期行为：**
1. Skill 读取 `production/session-state/active.md`
2. Skill 找到活动 story 引用
3. Skill 读取该 story 文件并正常继续
4. 输出确认自动检测到哪个 story

**断言：**
- [ ] 当未给出参数时，skill 读取 `production/session-state/active.md`
- [ ] Skill 在继续前识别并确认自动检测到的 story
- [ ] 如果在会话状态中未找到 story，skill 要求用户提供路径

---

### 用例 5：Director 关卡——跨审查模式的 LP-CODE-REVIEW 行为

**Fixture：**
- Story 文件位于 `production/epics/core/story-light-pickup.md`
- 所有验收标准已验证，无 GDD 偏差
- `production/session-state/review-mode.txt` 存在

**用例 5a——full 模式：**
- `review-mode.txt` 包含 `full`

**输入：** `/story-done production/epics/core/story-light-pickup.md`（full 模式）

**预期行为：**
1. Skill 读取审查模式——确定为 `full`
2. 实现验证后，skill 调用 LP-CODE-REVIEW 关卡
3. Lead programmer 审查实现
4. 如果 LP 判定为 NEEDS CHANGES → story 不能标记为 Complete
5. 如果 LP 判定为 APPROVED → skill 继续标记 story Complete

**断言（5a）：**
- [ ] Skill 在决定是否调用 LP-CODE-REVIEW 之前读取审查模式
- [ ] LP-CODE-REVIEW 关卡在 full 模式下实现检查后调用
- [ ] LP NEEDS CHANGES 判定阻止 story 被标记为 Complete
- [ ] 输出中注明关卡结果："Gate: LP-CODE-REVIEW——[result]"
- [ ] 即使 LP 批准，skill 在更新 story 状态前仍询问"可以写入吗"

**用例 5b——lean 或 solo 模式：**
- `review-mode.txt` 包含 `lean` 或 `solo`

**预期行为：**
1. Skill 读取审查模式——确定为 `lean` 或 `solo`
2. LP-CODE-REVIEW 关卡被跳过
3. 输出注明跳过："[LP-CODE-REVIEW] skipped——Lean/Solo mode"
4. Story 完成仅基于验收标准检查进行

**断言（5b）：**
- [ ] LP-CODE-REVIEW 关卡在 lean 或 solo 模式下不派生
- [ ] 输出明确注明跳过
- [ ] Skill 在标记 story Complete 前仍需要"可以写入吗"批准

---

## 协议合规性

- [ ] 更新 story 文件前使用"可以写入吗"
- [ ] 添加条目到 `docs/tech-debt-register.md` 前使用"可以写入吗"
- [ ] 请求批准前展示完整发现（标准检查、偏差检查）
- [ ] 以从 sprint 计划中展示下一个就绪的 story 结束
- [ ] 如果有任何标准处于 ERROR 状态，不将 story 标记为 Complete
- [ ] 不跳过代码审查提示

---

## 覆盖说明

- Skill 的完整 8 阶段流程在用例 1-3 中执行；并非每个阶段内的所有
  边缘情况都被覆盖。
- Tech debt 记录（延期项目写入 `docs/tech-debt-register.md`）
  在用例 2 中提到，但不是主要断言焦点；
  专用覆盖被延期。
- `sprint-status.yaml` 更新（skill 中的第 7 阶段）由用例 1 隐含，
  但不是主要断言；假设遵循相同的"可以写入吗"模式。
- 具有多个 TR-ID 或多个 ADR 的 story 未明确测试。
