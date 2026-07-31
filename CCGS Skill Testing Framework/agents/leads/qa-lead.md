# Agent Test Spec: qa-lead

## Agent Summary
**拥有的领域：** 测试策略、QL-STORY-READY gate、QL-TEST-COVERAGE gate、bug 严重性分流、发布质量 gate。
**不负责：** 功能实现（程序员）、游戏设计决策、创意方向、制作排期。
**模型层级：** Sonnet（单个系统分析 — 故事就绪性和覆盖评估）。
**处理的 gate ID：** QL-STORY-READY、QL-TEST-COVERAGE。

---

## 静态断言（结构性）

通过读取 agent 的 `.claude/agents/qa-lead.md` frontmatter 验证：

- [ ] `description:` 字段存在且特定于领域（引用测试策略、故事就绪性、覆盖率、bug 分流 — 不是泛泛的）
- [ ] `allowed-tools:` 列表以读取为中心；可能包含 Read 用于故事文件、测试文件和 coding-standards；仅在需要运行测试命令时包含 Bash
- [ ] 模型层级按 coordination-rules.md 为 `claude-sonnet-4-6`
- [ ] Agent 定义不声称对实现决策或游戏设计拥有权限

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出格式
**场景：** 提交了一个"玩家受到危险方块伤害"的故事进行就绪性检查。该故事有三个验收标准：(1) 玩家生命值按危险值减少，(2) 播放伤害视觉反馈，(3) 玩家在 0.5 秒内无法再次受到伤害（无敌窗口）。所有三个 AC 都是可测量和具体的。请求标记为 QL-STORY-READY。
**预期：** 返回 `QL-STORY-READY: ADEQUATE`，并附上确认所有三个 AC 存在、具体且可测试的理由。
**断言：**
- [ ] 裁决恰好是 ADEQUATE / INADEQUATE 之一
- [ ] 裁决标记格式为 `QL-STORY-READY: ADEQUATE`
- [ ] 理由引用 AC 的具体数量（3）并确认每个都是可测量的
- [ ] 输出保持在 QA 范围内 — 不对机制设计的好坏发表评论

### 用例 2：领域外请求 — 重定向或升级
**场景：** 开发者要求 qa-lead 为新物理系统实现自动化测试工具。
**预期：** Agent 拒绝实现测试代码，并重定向到相应的程序员（gameplay-programmer 或 lead-programmer）。
**断言：**
- [ ] 不编写或提出代码实现
- [ ] 明确命名 `lead-programmer` 或 `gameplay-programmer` 为实现的正确处理者
- [ ] 可能定义测试应该验证什么（测试策略），但将代码编写推迟给程序员

### 用例 3：Gate 裁决 — 正确词汇
**场景：** 提交了一个"战斗感觉响应迅速且有冲击力"的故事进行就绪性检查。唯一验收标准是："战斗应该让玩家感觉良好。"这是主观且不可测量的。请求标记为 QL-STORY-READY。
**预期：** 返回 `QL-STORY-READY: INADEQUATE`，并具体识别不可测量的 AC，以及关于如何使其可测试的指导（例如，"输入到命中反馈延迟 ≤ 100ms"）。
**断言：**
- [ ] 裁决恰好是 ADEQUATE / INADEQUATE 之一 — 不是自由格式文本
- [ ] 裁决标记格式为 `QL-STORY-READY: INADEQUATE`
- [ ] 理由识别未能满足可测量性要求的具体 AC
- [ ] 提供关于如何重写 AC 使其可测试的可操作指导

### 用例 4：冲突升级 — 正确的上级
**场景：** gameplay-programmer 和 qa-lead 对断言"敌人巡逻路径在 5 秒内访问所有路点"的测试是否足够确定性以成为有效自动化测试存在分歧。gameplay-programmer 认为时间变化使其不稳定；qa-lead 认为可以接受。
**预期：** qa-lead 承认技术不稳定性问题，并升级到 lead-programmer 就自动化测试的可接受确定性标准做出技术裁决。
**断言：**
- [ ] 升级到 `lead-programmer` 就确定性标准做出技术裁决
- [ ] 不单方面否决 gameplay-programmer 的不稳定性担忧
- [ ] 清晰地构建升级框架："这是技术标准问题，而非 QA 覆盖问题"
- [ ] 不放弃覆盖要求 — 如果当前方法被裁定为不稳定，要求提供确定性替代方案

### 用例 5：上下文传递 — 使用提供的上下文
**场景：** Agent 收到一个 gate 上下文块，包含 coding-standards.md 测试标准部分，其中规定：Logic 故事需要阻塞性自动化单元测试，Visual/Feel 故事需要截图 + 主管签字（建议性），Config/Data 故事需要通过冒烟检查（建议性）。提交了一个分类为"Logic"类型的故事，仅提供手动演练文档作为证据。
**预期：** 评估引用 coding-standards.md 的具体测试证据要求，识别"Logic"故事需要自动化单元测试（而非仅手动演练），并返回 INADEQUATE 并引用具体要求。
**断言：**
- [ ] 引用提供的上下文中具体的故事类型分类（"Logic"）
- [ ] 引用 coding-standards.md 中 Logic 故事的具体证据要求（自动化单元测试）
- [ ] 识别提交的证据类型（手动演练）对于此故事类型不足
- [ ] 不将建议性级别的要求应用为阻塞性要求

---

## 协议合规

- [ ] 仅使用 ADEQUATE / INADEQUATE 词汇返回 QL-STORY-READY 裁决
- [ ] 仅使用 ADEQUATE / INADEQUATE 词汇返回 QL-TEST-COVERAGE 裁决（或 PASS / FAIL 用于发布 gate）
- [ ] 在声明的 QA 和测试策略领域内保持
- [ ] 将技术标准争议升级到 lead-programmer
- [ ] 在输出中使用 gate ID（例如，`QL-STORY-READY: INADEQUATE`），而非散文式裁决
- [ ] 不做出约束性的实现或游戏设计决策

---

## 覆盖说明
- QL-TEST-COVERAGE（sprint 或里程碑的整体覆盖评估）未被覆盖 — 当覆盖报告可用时应添加专用用例。
- Bug 严重性分流（P0/P1/P2 分类）此处未覆盖 — 推迟到 /bug-triage 技能集成。
- 发布质量 gate 行为（PASS / FAIL 词汇变体）未被覆盖。
- QL-STORY-READY 与故事 Done 标准（/story-done 技能）之间的交互未被覆盖。
