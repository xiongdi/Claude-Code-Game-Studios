---
name: team-polish
description: "编排打磨团队：协调 performance-analyst、technical-artist、sound-designer 和 qa-tester，优化、打磨和加固功能或区域以达到发布质量。"
argument-hint: "[feature or area to polish] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
model: sonnet
---
如果没有提供参数，输出用法指导并退出，不生成任何 agent：
> Usage: `/team-polish [feature or area]` — 指定要打磨的功能或区域（如 `combat`、`main menu`、`inventory system`、`level-1`）。此处不要使用 `AskUserQuestion`；直接输出指导。

当此 skill 带参数调用时，通过结构化管线编排打磨团队。

**决策点：** 在每个阶段转换时，使用 `AskUserQuestion` 向用户
展示子 agent 的提案作为可选项。在对话中写入 agent 的
完整分析，然后用简洁标签捕获决策。
用户必须批准才能进入下一阶段。

## Phase 0: 解析 Review 模式

1. 如果传入了 `--review [mode]` 作为参数，使用该模式。
2. 否则读取 `production/review-mode.txt` — 使用那里写的内容。
3. 否则默认为 `lean`。

模式：
- `full` — 按所述生成所有 director 和 lead gates
- `lean` — 跳过 director gates，除非它们是 PHASE-GATE 类型（CD-PHASE-GATE, TD-PHASE-GATE, PR-PHASE-GATE, AD-PHASE-GATE）
- `solo` — 完全跳过所有 director gate 生成；在没有任何 agent gates 的情况下运行 skill

存储解析后的模式供所有后续阶段使用。

**Director gate 跳过规则**：在生成任何 Tier 1 director 或 lead 进行审查之前（PHASE-GATE 触发器之外），应用解析后的模式：如果是 solo 模式则跳过；如果是 lean 模式且这不是 PHASE-GATE 则跳过。

## 团队组成
- **performance-analyst** — 分析、优化、内存分析、帧预算
- **engine-programmer** — 引擎级瓶颈：渲染管线、内存、资源加载（当 performance-analyst 识别出底层根本原因时调用）
- **technical-artist** — VFX 打磨、着色器优化、视觉质量
- **sound-designer** — 音频打磨、混音、环境层、反馈声音
- **tools-programmer** — 内容管线工具验证、编辑器工具稳定性、自动化修复（当打磨区域涉及内容创作工具时调用）
- **qa-tester** — 边界情况测试、回归测试、soak 测试

## 如何委托

使用 Task 工具将每个团队成员生成为子 agent：
- `subagent_type: performance-analyst` — 分析、优化、内存分析
- `subagent_type: engine-programmer` — 渲染、内存、资源加载的引擎级修复
- `subagent_type: technical-artist` — VFX 打磨、着色器优化、视觉质量
- `subagent_type: sound-designer` — 音频打磨、混音、环境层
- `subagent_type: tools-programmer` — 内容管线和编辑器工具验证
- `subagent_type: qa-tester` — 边界情况测试、回归测试、soak 测试

始终在每个 agent 的提示中提供完整上下文（目标功能/区域、性能预算、已知问题）。在管线允许的地方并行启动独立 agent（如 Phase 3 和 Phase 4 可以同时运行）。

## 管线

### Phase 1: 评估
委托给 **performance-analyst**：
- 使用 `/perf-profile` 分析目标功能/区域
- 识别性能瓶颈和帧预算违规
- 测量内存使用并检查泄漏
- 对照目标硬件规格进行基准测试
- 输出：带有优先级排序优化列表的性能报告

### Phase 2: 优化
委托给 **performance-analyst**（根据需要连同相关程序员）：
- 修复 Phase 1 中识别的性能热点
- 优化绘制调用，减少过度绘制
- 修复内存泄漏并减少分配压力
- 验证优化不改变游戏行为
- 输出：带有前后指标的优化代码

如果 Phase 1 识别出引擎级根本原因（渲染管线、资源加载、内存分配器），并行委托给 **engine-programmer** 进行这些修复：
- 优化引擎系统中的热路径
- 修复核心循环中的分配压力
- 输出：带有分析器验证的引擎级修复

### Phase 3: 视觉打磨（与 Phase 2 并行）
委托给 **technical-artist**：
- 审查 VFX 的质量和与 art bible 的一致性
- 优化粒子系统和着色器效果
- 在适当的地方添加屏幕抖动、摄像机效果和视觉 juice
- 确保效果在较低设置下优雅降级
- 输出：打磨后的视觉效果

### Phase 4: 音频打磨（与 Phase 2 并行）
委托给 **sound-designer**：
- 审查音频事件的完整性（是否有任何动作缺少声音反馈？）
- 检查音频混音级别 — 相对于混音没有太大声或太安静的内容
- 添加环境音频层以营造氛围
- 验证音频在空间定位下正确播放
- 输出：音频打磨列表和混音说明

### Phase 5: 加固
委托给 **qa-tester**：
- 测试所有边界情况：边界条件、快速输入、异常序列
- Soak 测试：长时间运行功能检查退化
- 压力测试：最大实体数、最坏情况场景
- 回归测试：验证打磨更改没有破坏现有功能
- 在最低规格硬件上测试（如可用）
- 输出：带有任何剩余问题的测试结果

### Phase 6: 签字
- 收集团队成员的所有结果
- 对照预算比较性能指标
- 报告：READY FOR RELEASE / NEEDS MORE WORK
- 列出任何剩余问题，附严重程度和建议

## 错误恢复协议

如果任何生成的 agent（通过 Task）返回 BLOCKED、错误或无法完成：

1. **立即展示**：在继续到依赖阶段之前向用户报告"[AgentName]: BLOCKED — [原因]"
2. **评估依赖关系**：检查被阻塞 agent 的输出是否是后续阶段所需的。如果是，未经用户输入不要超过该依赖点。
3. **通过 AskUserQuestion 提供选项**，选项为：
   - 跳过此 agent 并在最终报告中注明缺口
   - 以更窄的范围重试
   - 在此停止并首先解决阻塞
4. **始终生成部分报告** — 输出任何已完成的内容。绝不因为一个 agent 阻塞而丢弃工作。

常见阻塞：
- 输入文件缺失（story 未找到、GDD 缺失） → 重定向到创建它的 skill
- ADR 状态为 Proposed → 不要实现；先运行 `/architecture-decision`
- 范围太大 → 通过 `/create-stories` 拆分为两个 stories
- ADR 和 story 之间的冲突指令 → 展示冲突，不要猜测

## 文件写入协议

所有文件写入（性能报告、测试结果、证据文档）都委托给
通过 Task 生成的子 agent。每个子 agent 执行"May I write to [path]?"
协议。此编排器不直接写入文件。

## 输出

涵盖以下内容的摘要报告：性能前后指标、视觉打磨更改、音频打磨更改、测试结果，以及发布准备评估。

## 后续步骤

- 如果 READY FOR RELEASE：运行 `/release-checklist` 进行最终的发布前验证。
- 如果 NEEDS MORE WORK：在 `/sprint-plan update` 中安排剩余问题并在修复后重新运行 `/team-polish`。
- 在交接给发布之前运行 `/gate-check` 获取正式阶段关卡裁决。
