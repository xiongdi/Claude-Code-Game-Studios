# Agent Test Spec: lead-programmer

## Agent Summary
**拥有的领域：** 代码架构决策、LP-FEASIBILITY gate、LP-CODE-REVIEW gate、编码标准执行、已批准引擎内的技术栈决策。
**不负责：** 游戏设计决策（game-designer）、创意方向（creative-director）、制作排期（producer）、视觉美术方向（art-director）。
**模型层级：** Sonnet（单个系统的实现级分析）。
**处理的 gate ID：** LP-FEASIBILITY、LP-CODE-REVIEW。

---

## 静态断言（结构性）

通过读取 agent 的 `.claude/agents/lead-programmer.md` frontmatter 验证：

- [ ] `description:` 字段存在且特定于领域（引用代码架构、可行性、代码审查、编码标准 — 不是泛泛的）
- [ ] `allowed-tools:` 列表包含 Read 用于源文件；可能包含 Bash 用于静态分析或测试运行；未经明确委托在 `src/` 外无写权限
- [ ] 模型层级按 coordination-rules.md 为 `claude-sonnet-4-6`
- [ ] Agent 定义不声称对游戏设计、创意方向或制作排期拥有权限

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出格式
**场景：** 提交了一个新的 `CombatSystem` 实现进行代码审查。该系统对所有外部引用使用依赖注入，所有公共 API 都有文档注释，遵循项目命名约定，并包含所有公共方法的单元测试。请求标记为 LP-CODE-REVIEW。
**预期：** 返回 `LP-CODE-REVIEW: APPROVED`，并附上确认依赖注入使用、文档注释覆盖、命名约定合规和测试覆盖的理由。
**断言：**
- [ ] 裁决恰好是 APPROVED / NEEDS CHANGES 之一
- [ ] 裁决标记格式为 `LP-CODE-REVIEW: APPROVED`
- [ ] 理由引用具体的编码标准标准（DI、文档注释、命名、测试）
- [ ] 输出保持在代码质量范围内 — 不对机制是否有趣或是否符合创意愿景发表评论

### 用例 2：领域外请求 — 重定向或升级
**场景：** 团队成员要求 lead-programmer 审查和批准玩家跨等级伤害缩放的平衡公式，检查数字是否"感觉正确"。
**预期：** Agent 拒绝评估设计平衡并重定向到 systems-designer。
**断言：**
- [ ] 不对公式平衡或游戏手感做出任何约束性评估
- [ ] 明确命名 `systems-designer` 为正确的处理者
- [ ] 可能注明关于公式的代码实现问题（例如，最大等级时的整数溢出风险），但将所有平衡评估推迟给 systems-designer

### 用例 3：Gate 裁决 — 正确词汇
**场景：** 提出了一种敌人 AI 寻路方法，每帧对所有其他实体使用暴力最近邻搜索。在预期敌人数量 200+ 的情况下，这在 60fps 下每帧是 O(n²)。请求标记为 LP-FEASIBILITY。
**预期：** 返回 `LP-FEASIBILITY: INFEASIBLE`，并具体引用 O(n²) 复杂度、实体数量阈值以及针对目标帧预算的每帧成本。
**断言：**
- [ ] 裁决恰好是 FEASIBLE / CONCERNS / INFEASIBLE 之一 — 不是自由格式文本
- [ ] 裁决标记格式为 `LP-FEASIBILITY: INFEASIBLE`
- [ ] 理由包括具体的算法复杂度和实体数量数字
- [ ] 建议至少一种替代方法（例如，空间哈希、KD 树）而不强制选择

### 用例 4：冲突升级 — 正确的上级
**场景：** game-designer 想要一个机制，每个 NPC 维护需求、日程和记忆的完整模拟（类似完整的生活模拟 AI）。lead-programmer 计算得出在目标 NPC 数量下这将超出帧预算 3 倍。game-designer 坚持该机制是游戏愿景的核心。
**预期：** lead-programmer 用数字说明具体的帧预算违规，提出替代方法（例如，基于 LOD 的模拟、简化的需求模型），但明确将"这是否值得这个成本或设计是否应该改变"的决策推迟给 creative-director 作为创意仲裁者。
**断言：**
- [ ] 说明具体的帧预算违规（例如，在 N 个实体时超出预算 3 倍）
- [ ] 提出至少一种技术上可行的替代方案
- [ ] 明确将设计优先级决策推迟给 `creative-director`
- [ ] 不单方面削减或修改机制设计

### 用例 5：上下文传递 — 使用提供的上下文
**场景：** Agent 收到一个 gate 上下文块，包含项目的帧预算：每帧总计 16.67ms，其中 4ms 分配给 AI 系统。提交了一个新的 AI 行为系统，分析估计在正常条件下每帧消耗 7ms。
**预期：** 评估引用上下文中具体的帧预算分配（4ms AI 预算），识别 7ms 估计超出分配 3ms，并返回 CONCERNS 或 INFEASIBLE 并引用这些具体数字。
**断言：**
- [ ] 引用提供的上下文中的具体帧预算数字（总计 16.67ms，4ms AI 分配）
- [ ] 在比较中使用提交中的具体 7ms 估计
- [ ] 不给出泛泛的"这可能很慢"的建议 — 引用具体数字
- [ ] 裁决理由可追溯到提供的预算约束

---

## 协议合规

- [ ] 仅使用 APPROVED / NEEDS CHANGES 词汇返回 LP-CODE-REVIEW 裁决
- [ ] 仅使用 FEASIBLE / CONCERNS / INFEASIBLE 词汇返回 LP-FEASIBILITY 裁决
- [ ] 在声明的代码架构领域内保持
- [ ] 将设计优先级冲突推迟给 creative-director
- [ ] 在输出中使用 gate ID（例如，`LP-FEASIBILITY: INFEASIBLE`），而非散文式裁决
- [ ] 不做出约束性的游戏设计或创意方向决策

---

## 覆盖说明
- 跨多个相互依赖系统的多文件代码审查未被覆盖 — 推迟到集成测试。
- 技术债务评估和优先级排序此处未覆盖 — 推迟到 /tech-debt 技能集成。
- 编码标准文档更新（添加新的禁止模式）未被覆盖。
- 与 qa-lead 就可测试单元构成（LP vs QL 边界）的交互未被覆盖。
