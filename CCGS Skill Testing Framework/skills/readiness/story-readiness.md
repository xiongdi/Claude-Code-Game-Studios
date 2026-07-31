# Skill 测试规格：/story-readiness

## Skill 摘要

`/story-readiness` 验证 story 文件是否已准备好供开发者
接手实现。它检查四个维度：Design（嵌入的 GDD 需求）、
Architecture（ADR 引用和状态）、Scope（清晰的边界和 DoD）
以及 Definition of Done（可测试的标准）。它生成
READY / NEEDS WORK / BLOCKED 判定。它是只读 skill，在任何开发者接手 story 之前运行。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题或编号检查部分
- [ ] 包含判定关键词：READY、NEEDS WORK、BLOCKED
- [ ] 不需要"可以写入吗"语言（只读 skill）
- [ ] 有下一步连接（判定后做什么）

---

## 测试用例

### 用例 1：正常路径——完全就绪的 story

**Fixture：**
- Story 文件存在于 `production/epics/core/story-light-pickup.md`
- Story 包含：
  - `TR-ID: TR-light-001`（GDD 需求引用）
  - `ADR: docs/architecture/adr-003-inventory.md`
  - 引用的 ADR 存在且状态为 `Accepted`
  - 引用的 TR-ID 存在于 `docs/architecture/tr-registry.yaml` 中
  - Story 有 `## Acceptance Criteria`，包含 ≥3 个可测试项目
  - Story 有 `## Definition of Done` 部分
  - Story 有 `Status: Ready for Dev`
  - Story 头部的 Manifest 版本与当前 `docs/architecture/control-manifest.md` 匹配

**输入：** `/story-readiness production/epics/core/story-light-pickup.md`

**预期行为：**
1. Skill 读取 story 文件
2. Skill 读取引用的 ADR——验证状态为 `Accepted`
3. Skill 读取 `docs/architecture/tr-registry.yaml`——验证 TR-ID 存在
4. Skill 读取 `docs/architecture/control-manifest.md`——验证 manifest 版本匹配
5. Skill 评估所有 4 个维度（Design、Architecture、Scope、DoD）
6. Skill 输出 READY 判定，所有检查通过

**断言：**
- [ ] Skill 读取引用的 ADR 文件（不只是 story）
- [ ] Skill 验证 ADR 状态为 `Accepted`（不是 `Proposed`）
- [ ] Skill 读取 `tr-registry.yaml` 验证 TR-ID 存在
- [ ] 输出包含所有 4 个维度的检查结果
- [ ] 所有检查通过时判定为 READY
- [ ] Skill 不写入任何文件

---

### 用例 2：阻塞路径——引用的 ADR 是 Proposed（不是 Accepted）

**Fixture：**
- Story 文件存在，包含 `ADR: docs/architecture/adr-005-light-system.md`
- `adr-005-light-system.md` 存在但状态为 `Status: Proposed`
- 所有其他 story 内容完整

**输入：** `/story-readiness production/epics/core/story-light-system.md`

**预期行为：**
1. Skill 读取 story
2. Skill 读取 `adr-005-light-system.md`——发现 `Status: Proposed`
3. Skill 将其标记为阻塞性问题（不能针对未接受的 ADR 实现）
4. Skill 输出 BLOCKED 判定
5. Skill 建议：在接受或拒绝 ADR 之前不要接手 story

**断言：**
- [ ] 当 ADR 为 Proposed 时判定为 BLOCKED（不是 NEEDS WORK 或 READY）
- [ ] 输出明确命名 Proposed ADR 为阻塞项
- [ ] 输出建议在继续前解决 ADR 状态
- [ ] 无论其他检查是否通过，skill 不输出 READY

---

### 用例 3：需要工作——缺少 Acceptance Criteria

**Fixture：**
- Story 文件存在但没有 `## Acceptance Criteria` 部分
- ADR 引用存在且为 `Accepted`
- TR-ID 存在于 registry 中
- Manifest 版本匹配

**输入：** `/story-readiness production/epics/core/story-oxygen-drain.md`

**预期行为：**
1. Skill 读取 story
2. Skill 发现无 Acceptance Criteria 部分
3. Skill 将其标记为 NEEDS WORK 问题（story 不完整，不是阻塞）
4. Skill 输出 NEEDS WORK 判定
5. Skill 命名缺失的部分并建议添加可测量的标准

**断言：**
- [ ] 当 Acceptance Criteria 部分缺失时判定为 NEEDS WORK（不是 BLOCKED 或 READY）
- [ ] 输出明确识别缺失的 Acceptance Criteria 部分
- [ ] 输出建议添加可测试/可测量的标准
- [ ] Skill 区分 NEEDS WORK（无需外部依赖即可修复）和 BLOCKED（需要外部操作）

---

### 用例 4：边缘情况——过期的 manifest 版本

**Fixture：**
- Story 文件头部有 `Manifest Version: 2026-01-15`
- `docs/architecture/control-manifest.md` 有 `Manifest Version: 2026-03-10`
- 版本不匹配（story 是在 manifest 更新前创建的）

**输入：** `/story-readiness production/epics/core/story-mirror-rotation.md`

**预期行为：**
1. Skill 读取 story 并提取 manifest 版本 `2026-01-15`
2. Skill 读取 control manifest 头部并提取当前版本 `2026-03-10`
3. Skill 检测到版本不匹配
4. Skill 将其标记为 ADVISORY 问题（不是阻塞，但值得注意）
5. 判定为 NEEDS WORK，并注明 manifest 过期

**断言：**
- [ ] Skill 读取 `docs/architecture/control-manifest.md` 获取当前版本
- [ ] Skill 将 story 中嵌入的 manifest 版本与当前 manifest 版本进行比较
- [ ] 过期的 manifest 版本导致 NEEDS WORK（不是 BLOCKED，不是 READY）
- [ ] 输出解释 story 中嵌入的指导可能已过时

---

### 用例 5：Director 关卡——跨审查模式的 QL-STORY-READY 行为

**Fixture：**
- Story 文件存在且为 READY（所有 4 个维度通过，ADR Accepted，标准存在）
- `production/session-state/review-mode.txt` 存在

**用例 5a——full 模式：**
- `review-mode.txt` 包含 `full`

**输入：** `/story-readiness production/epics/core/story-light-pickup.md`（full 模式）

**预期行为：**
1. Skill 读取审查模式——确定为 `full`
2. 完成自己的 4 维度检查后，skill 调用 QL-STORY-READY 关卡
3. QA 负责人审查 story 的就绪情况
4. 如果 QA 负责人判定为 INADEQUATE → story 判定为 BLOCKED，无论 4 维度结果如何
5. 如果 QA 负责人判定为 ADEQUATE → 判定正常进行

**断言（5a）：**
- [ ] Skill 在决定是否调用 QL-STORY-READY 之前读取审查模式
- [ ] QL-STORY-READY 关卡在 full 模式下 4 维度检查完成后调用
- ] QA 负责人 INADEQUATE 判定覆盖 READY 4 维度结果 → 最终判定 BLOCKED
- [ ] 输出中注明关卡调用："Gate: QL-STORY-READY——[result]"

**用例 5b——lean 或 solo 模式：**
- `review-mode.txt` 包含 `lean` 或 `solo`

**预期行为：**
1. Skill 读取审查模式——确定为 `lean` 或 `solo`
2. QL-STORY-READY 关卡被跳过
3. 输出注明跳过："[QL-STORY-READY] skipped——Lean/Solo mode"
4. 判定仅基于 4 维度检查

**断言（5b）：**
- [ ] QL-STORY-READY 关卡在 lean 或 solo 模式下不派生
- [ ] 输出明确注明跳过
- [ ] 判定仅基于 4 维度检查

---

## 协议合规性

- [ ] 不使用 Write 或 Edit 工具（只读 skill）
- [ ] 判定前展示完整检查结果
- [ ] 不请求批准（无文件写入）
- [ ] 以推荐的下一步结束（修复问题或继续实现）
- [ ] 清楚区分三个判定级别（READY vs NEEDS WORK vs BLOCKED）

---

## 覆盖说明

- TR-ID 完全不在 registry 中的情况未在此处明确测试；
  它遵循与用例 3 相同的 NEEDS WORK 模式。
- "无参数"路径（skill 自动检测当前 story）未测试，
  因为它依赖于 `production/session-state/active.md` 内容，
  难以可靠地进行 fixture 设置。
- 具有多个 ADR 引用的 story 未测试；行为假定为
  累加性（所有 ADR 必须为 Accepted 才能获得 READY 判定）。
