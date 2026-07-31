# Agent 测试规格：technical-director

## Agent 摘要
**负责的领域：** 系统架构决策、技术可行性评估、ADR 监督和批准、引擎风险评估、技术阶段门控。
**不负责：** 游戏设计决策（creative-director / game-designer）、创意方向、视觉美术风格、制作排期（producer）。
**模型层级：** Opus（多文档综合、高风险架构和阶段门控裁决）。
**处理的 Gate ID：** TD-SYSTEM-BOUNDARY、TD-FEASIBILITY、TD-ARCHITECTURE、TD-ADR、TD-ENGINE-RISK、TD-PHASE-GATE。

---

## 静态断言（结构性）

通过读取 agent 的 `.claude/agents/technical-director.md` frontmatter 验证：

- [ ] `description:` 字段存在且领域特定（引用架构、可行性、ADR — 非通用）
- [ ] `allowed-tools:` 列表可能包含用于架构文档的 Read；仅当技术检查需要时才包含 Bash
- [ ] 模型层级为 `claude-opus-4-6`（根据 coordination-rules.md，具有门控综合的 director = Opus）
- [ ] Agent 定义不声称对游戏设计决策或创意方向有权限

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出格式
**场景：** "战斗系统"的架构文档提交审查。它描述了一个分层设计：输入层 → 游戏逻辑层 → 表示层，各层之间具有明确定义的接口。请求标记为 TD-ARCHITECTURE。
**预期：** 返回 `TD-ARCHITECTURE: APPROVE`，并附带理由确认系统边界正确分离且接口定义良好。
**断言：**
- [ ] 裁决严格为 APPROVE / CONCERNS / REJECT 之一
- [ ] 裁决标记格式为 `TD-ARCHITECTURE: APPROVE`
- [ ] 理由具体引用分层结构和接口定义 — 非通用架构建议
- [ ] 输出保持在技术范围内 — 不评论机制是否有趣或是否符合创意愿景

### 用例 2：领域外请求 — 重定向或升级
**场景：** Writer 要求 technical-director 审查和批准游戏开场过场动画的对话脚本。
**预期：** Agent 拒绝评估对话质量并重定向到 narrative-director。
**断言：**
- [ ] 不对对话内容或结构做出任何约束性决策
- [ ] 明确指出 `narrative-director` 是正确的处理者
- [ ] 可能注意到影响对话的技术限制（例如，本地化字符串限制、数据格式），但推迟所有内容决策

### 用例 3：门控裁决 — 正确的词汇
**场景：** 一个提议的多玩家机制需要每帧对所有活跃实体进行光线投射以检测视线。在预期玩家数量下（大区域中有 1000 个实体），这是每帧 O(n²)。请求标记为 TD-FEASIBILITY。
**预期：** 返回 `TD-FEASIBILITY: CONCERNS`，并具体引用 O(n²) 复杂度和在目标帧率下使此不可行的实体数量。
**断言：**
- [ ] 裁决严格为 APPROVE / CONCERNS / REJECT 之一 — 非自由格式文本
- [ ] 裁决标记格式为 `TD-FEASIBILITY: CONCERNS`
- [ ] 理由包括具体的算法复杂度问题和实体数量阈值
- [ ] 建议至少一种替代方法（例如，空间分区、兴趣管理），而不强制选择哪一种

### 用例 4：冲突升级 — 正确的父级
**场景：** game-designer 想要为每个库存物品添加实时物理模拟（屏幕上同时有数百个物品）。technical-director 评估这在技术上成本高昂，并提议简化模拟。game-designer 不同意，认为这对游戏手感至关重要。
**预期：** technical-director 清楚地陈述技术成本和约束，提出可以实现类似手感的替代实现方法，但明确将最终设计优先级决策推迟给 creative-director 作为玩家体验权衡的裁决者。
**断言：**
- [ ] 用具体细节表达技术担忧（例如，性能预算、估计成本）
- [ ] 提出至少一种可以在保留意图的同时降低成本的替代方案
- [ ] 明确将"这是否值得成本"的决策推迟给 creative-director — 不单方面削减功能
- [ ] 不声称有权覆盖 game-designer 的设计意图

### 用例 5：上下文传递 — 使用提供的上下文
**场景：** Agent 接收到一个门控上下文块，包含目标平台限制：移动端、60fps 目标、2GB RAM 上限、无计算着色器。一个提议的架构包含 GPU 驱动的渲染管线。
**预期：** 评估引用上下文中的具体硬件限制，将计算着色器依赖识别为与所述平台限制不兼容，并返回 CONCERNS 或 REJECT 裁决并引用这些具体细节。
**断言：**
- [ ] 引用提供的具体平台限制（移动端、2GB RAM、无计算着色器）
- [ ] 不提供与提供的限制无关的通用性能建议
- [ ] 正确识别与平台限制冲突的架构组件
- [ ] 裁决包含与提供的上下文相关的理由，而非样板警告

---

## 协议合规

- [ ] 仅使用 APPROVE / CONCERNS / REJECT 词汇返回裁决
- [ ] 保持在声明的技术领域内
- [ ] 将设计优先级冲突推迟给 creative-director
- [ ] 在输出中使用 gate ID（例如 `TD-FEASIBILITY: CONCERNS`），而非内联散文裁决
- [ ] 不做出约束性的游戏设计或创意方向决策

---

## 覆盖说明
- TD-ADR（架构决策记录批准）未覆盖 — 当 /architecture-decision skill 产生 ADR 文档时应添加专用用例。
- 特定引擎版本（例如 Godot 4.6 截止后 API）的 TD-ENGINE-RISK 评估未覆盖 — 推迟到 engine-specialist 集成测试。
- TD-PHASE-GATE（完整技术阶段推进）涉及综合多个子门控结果，被推迟。
- 多领域架构审查（例如同时涉及 TD-ARCHITECTURE 和 TD-ENGINE-RISK）未在此覆盖。
