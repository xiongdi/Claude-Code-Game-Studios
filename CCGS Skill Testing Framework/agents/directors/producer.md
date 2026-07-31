# Agent 测试规格：producer

## Agent 摘要
**负责的领域：** 范围管理、sprint 规划验证、里程碑追踪、epic 优先级排序、制作阶段门控。
**不负责：** 游戏设计决策（creative-director / game-designer）、技术架构（technical-director）、创意方向。
**模型层级：** Opus（多文档综合、高风险阶段门控裁决）。
**处理的 Gate ID：** PR-SCOPE、PR-SPRINT、PR-MILESTONE、PR-EPIC、PR-PHASE-GATE。

---

## 静态断言（结构性）

通过读取 agent 的 `.claude/agents/producer.md` frontmatter 验证：

- [ ] `description:` 字段存在且领域特定（引用范围、sprint、里程碑、制作 — 非通用）
- [ ] `allowed-tools:` 列表以读取为主；仅当 sprint/里程碑文件需要解析时才包含 Bash
- [ ] 模型层级为 `claude-opus-4-6`（根据 coordination-rules.md，具有门控综合的 director = Opus）
- [ ] Agent 定义不声称对设计决策或技术架构有权限

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出格式
**场景：** Sprint 7 的 sprint 计划提交审查。该计划包括 2 周内 4 名团队成员的 12 个 story points。过去 3 个 sprint 的历史速度平均为 11.5 points。请求标记为 PR-SPRINT。
**预期：** 返回 `PR-SPRINT: REALISTIC`，并附带理由指出计划在历史速度的一个标准差范围内，且容量似乎匹配。
**断言：**
- [ ] 裁决严格为 REALISTIC / CONCERNS / UNREALISTIC 之一
- [ ] 裁决标记格式为 `PR-SPRINT: REALISTIC`
- [ ] 理由引用具体的 story point 数量和历史速度数据
- [ ] 输出保持在制作范围内 — 不评论故事是否设计良好或技术是否合理

### 用例 2：领域外请求 — 重定向或升级
**场景：** 团队成员要求 producer 评估游戏的"基于重量的库存"机制是否感觉有趣且引人入胜。
**预期：** Agent 拒绝评估游戏手感并重定向到 game-designer 或 creative-director。
**断言：**
- [ ] 不对机制的设计质量做出任何约束性评估
- [ ] 明确指出 `game-designer` 或 `creative-director` 是正确的处理者
- [ ] 可能注意到机制的范围是否有制作影响（例如，对其他系统的依赖），但推迟所有设计评估

### 用例 3：门控裁决 — 正确的词汇
**场景：** 一个新功能提案向一个原本只规划了两个系统的里程碑添加了三个新系统（制作、天气和派系声望）。这些添加均未出现在当前里程碑计划中。请求标记为 PR-SCOPE。
**预期：** 返回 `PR-SCOPE: CONCERNS`，并具体识别三个未规划的系统及其不在里程碑范围文档中。
**断言：**
- [ ] 裁决严格为 REALISTIC / CONCERNS / UNREALISTIC 之一 — 非自由格式文本
- [ ] 裁决标记格式为 `PR-SCOPE: CONCERNS`
- [ ] 理由命名三个超出范围的具体系统
- [ ] 不评估系统是否是好的设计 — 只评估它们是否符合计划

### 用例 4：冲突升级 — 正确的父级
**场景：** game-designer 想要添加一个后期机制（影响所有游戏系统的动态天气），technical-director 警告说这将需要额外的 3 个 sprint。game-designer 和 technical-director 就是否继续进行存在分歧。
**预期：** Producer 不就机制是否值得添加（设计决策）或是否可行（技术决策）选边。Producer 量化制作影响（3 个 sprint 的延迟、里程碑滑移风险），向用户呈现权衡，并遵循 coordination-rules.md 冲突解决：升级到共同父级（在这种情况下，由于 creative-director 和 technical-director 都是顶级，将冲突呈现给用户决策）。
**断言：**
- [ ] 以具体术语量化制作影响（sprint 数量、里程碑日期滑移）
- [ ] 不做出约束性的设计或技术决策
- [ ] 向用户呈现冲突，清楚说明范围影响
- [ ] 引用 coordination-rules.md 冲突解决协议（升级到共同父级或用户）

### 用例 5：上下文传递 — 使用提供的上下文
**场景：** Agent 接收到一个门控上下文块，包含当前里程碑截止日期（8 周后）和过去 4 个 sprint 的速度数据（8、10、9、11 points）。一个包含 14 个 story points 的 sprint 计划提交审查。
**预期：** 评估使用提供的速度数据来预测 14 points 是否可实现，并参考 8 周里程碑窗口来评估当前 sprint 的范围是否留有足够的缓冲。
**断言：**
- [ ] 使用提供的上下文中的具体速度数据（非通用估计）
- [ ] 在容量评估中参考 8 周截止日期
- [ ] 计算或估计里程碑窗口内的剩余 sprint 数量
- [ ] 不提供与提供的截止日期和速度数据无关的通用范围建议

---

## 协议合规

- [ ] 仅使用 REALISTIC / CONCERNS / UNREALISTIC 词汇返回裁决
- [ ] 保持在声明的制作领域内
- [ ] 通过量化范围影响并呈现给用户来升级设计/技术冲突
- [ ] 在输出中使用 gate ID（例如 `PR-SPRINT: REALISTIC`），而非内联散文裁决
- [ ] 不做出约束性的游戏设计或技术架构决策

---

## 覆盖说明
- PR-EPIC（epic 级别优先级排序）未覆盖 — 当 /create-epics skill 产生结构化 epic 文档时应添加专用用例。
- PR-MILESTONE（里程碑健康审查）未覆盖 — 推迟到与 /milestone-review skill 集成。
- PR-PHASE-GATE（完整制作阶段推进）涉及综合多个子门控结果，被推迟。
- 多 sprint 燃尽和速度趋势分析未在此覆盖。
