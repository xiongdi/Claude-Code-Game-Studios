# Agent 测试规格：creative-director

## Agent 摘要
**负责的领域：** 创意愿景、游戏支柱、GDD 对齐、系统分解反馈、叙事方向、试玩反馈解读、阶段门控（创意方面）。
**不负责：** 技术架构或实现细节（委托给 technical-director）、制作排期（producer）、视觉美术风格执行（委托给 art-director）。
**模型层级：** Opus（多文档综合、高风险阶段门控裁决）。
**处理的 Gate ID：** CD-PILLARS、CD-GDD-ALIGN、CD-SYSTEMS、CD-NARRATIVE、CD-PLAYTEST、CD-PHASE-GATE。

---

## 静态断言（结构性）

通过读取 agent 的 `.claude/agents/creative-director.md` frontmatter 验证：

- [ ] `description:` 字段存在且领域特定（引用创意愿景、支柱、GDD 对齐 — 非通用）
- [ ] `allowed-tools:` 列表以读取为主；除非创意工作流需要，否则不应包含 Bash
- [ ] 模型层级为 `claude-opus-4-6`（根据 coordination-rules.md，具有门控综合的 director = Opus）
- [ ] Agent 定义不声称对技术架构或制作排期有权限

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出格式
**场景：** 一个游戏概念文档提交进行支柱审查。该概念描述了一款围绕三个支柱构建的叙事生存游戏："涌现式故事"、"有意义的牺牲"和"沉浸感世界"。请求标记为 CD-PILLARS。
**预期：** 返回 `CD-PILLARS: APPROVE`，并附带理由引用每个支柱在概念中的体现，以及文档中发现的任何强化或削弱信号。
**断言：**
- [ ] 裁决严格为 APPROVE / CONCERNS / REJECT 之一
- [ ] 裁决标记格式为 `CD-PILLARS: APPROVE`（gate ID 前缀、冒号、裁决关键词）
- [ ] 理由引用三个具体支柱的名称，而非通用创意建议
- [ ] 输出保持在创意范围内 — 不评论引擎可行性或 sprint 排期

### 用例 2：领域外请求 — 重定向或升级
**场景：** 开发者要求 creative-director 审查用于存储玩家存档数据的 PostgreSQL schema 提案。
**预期：** Agent 拒绝评估该 schema 并重定向到 technical-director。
**断言：**
- [ ] 不对 schema 设计做出任何约束性决策
- [ ] 明确指出 `technical-director` 是正确的处理者
- [ ] 可能注意到数据模型是否有创意影响（例如，追踪哪些玩家数据），但将所有结构决策完全推迟

### 用例 3：门控裁决 — 正确的词汇
**场景：** "制作"系统的 GDD 提交审查。第 4 节（公式）定义了一个惩罚探索的资源衰减公式 — 与 Player Fantasy 节中"无恐惧地漫游的自由"相矛盾。请求标记为 CD-GDD-ALIGN。
**预期：** 返回 `CD-GDD-ALIGN: CONCERNS`，并具体引用公式行为与 Player Fantasy 陈述之间的矛盾。
**断言：**
- [ ] 裁决严格为 APPROVE / CONCERNS / REJECT 之一 — 非自由格式文本
- [ ] 裁决标记格式为 `CD-GDD-ALIGN: CONCERNS`
- [ ] 理由引用或直接参考 GDD 第 4 节（公式）和 Player Fantasy 节
- [ ] 不开具具体的公式修复方案 — 那属于 systems-designer

### 用例 4：冲突升级 — 正确的父级
**场景：** technical-director 提出担忧，认为核心循环机制（实时分支对话）实现成本过高，建议削减。creative-director 基于创意理由不同意。
**预期：** creative-director 承认技术限制，不覆盖 technical-director 的可行性评估，但保留定义创意目标的权限。对于冲突本身，creative-director 是最高级别的创意升级点，在实施可行性上 defer 给 technical-director，同时倡导设计意图。解决路径是两者共同向用户呈现权衡选项。
**断言：**
- [ ] 不单方面覆盖 technical-director 的可行性担忧
- [ ] 清楚区分"我们创意上想要什么"和"它如何构建"
- [ ] 提出向用户呈现权衡，而非单方面解决
- [ ] 不声称拥有实现决策的权限

### 用例 5：上下文传递 — 使用提供的上下文
**场景：** Agent 接收到一个门控上下文块，包含游戏支柱文档（`design/gdd/pillars.md`）和供审查的新机制规格。支柱文档将"玩家署名"、"后果永久性"和"世界响应性"定义为三个核心支柱。
**预期：** 评估使用提供的文档中的确切支柱词汇，而非通用创意启发。任何批准或问题都回溯到一个或多个命名的支柱。
**断言：**
- [ ] 使用提供的上下文文档中的确切支柱名称
- [ ] 不生成与提供的支柱无关的通用创意反馈
- [ ] 引用与审查中的机制最相关的具体支柱
- [ ] 不引用提供的文档中不存在的支柱

---

## 协议合规

- [ ] 仅使用 APPROVE / CONCERNS / REJECT 词汇返回裁决
- [ ] 保持在声明的创意领域内
- [ ] 通过向用户呈现权衡来升级冲突，而非单方面覆盖
- [ ] 在输出中使用 gate ID（例如 `CD-PILLARS: APPROVE`），而非内联散文裁决
- [ ] 不做出约束性的跨领域决策（技术、制作、美术执行）

---

## 覆盖说明
- 多门控场景（例如，单个提交同时触发 CD-PILLARS 和 CD-GDD-ALIGN）未在此覆盖 — 推迟到集成测试。
- CD-PHASE-GATE（完整阶段推进）涉及综合多个子门控结果；此复杂情况被推迟。
- 试玩报告解读（CD-PLAYTEST）未覆盖 — 当 playtest-report skill 产生结构化输出时应添加专用用例。
- 与 art-director 在视觉支柱对齐上的交互未覆盖。
