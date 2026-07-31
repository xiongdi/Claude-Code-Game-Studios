---
name: team-audio
description: "编排音频团队：audio-director + sound-designer + technical-artist + gameplay-programmer，实现从指导到实现的完整音频管线。"
argument-hint: "[feature or area to design audio for] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
model: sonnet
---

如果没有提供参数，输出用法指导并退出，不生成任何 agent：
> Usage: `/team-audio [feature or area]` — 指定要为其设计音频的功能或区域（如 `combat`、`main menu`、`forest biome`、`boss encounter`）。此处不要使用 `AskUserQuestion`；直接输出指导。

当此 skill 带参数调用时，通过结构化管线编排音频团队。

**决策点：** 在每个步骤转换时，使用 `AskUserQuestion` 向用户
展示子 agent 的提案作为可选项。在对话中写入 agent 的
完整分析，然后用简洁标签捕获决策。
用户必须批准才能进入下一步。

## Phase 0: 解析 Review 模式

1. 如果传入了 `--review [mode]` 作为参数，使用该模式。
2. 否则读取 `production/review-mode.txt` — 使用那里写的内容。
3. 否则默认为 `lean`。

模式：
- `full` — 按所述生成所有 director 和 lead gates
- `lean` — 跳过 director gates，除非它们是 PHASE-GATE 类型（CD-PHASE-GATE, TD-PHASE-GATE, PR-PHASE-GATE, AD-PHASE-GATE）
- `solo` — 完全跳过所有 director gate 生成；在没有任何 agent gates 的情况下运行 skill

存储解析后的模式供所有后续阶段使用。

1. **读取参数**了解目标功能或区域（如 `combat`、
   `main menu`、`forest biome`、`boss encounter`）。

2. **收集上下文**：
   - 读取 `design/gdd/` 中该功能的相关设计文档
   - 如果存在，读取 `design/gdd/sound-bible.md` 中的 sound bible
   - 读取 `assets/audio/` 中现有的音频资产列表
   - 读取该区域任何现有的 sound design 文档

## 如何委托

使用 Task 工具将每个团队成员生成为子 agent：
- `subagent_type: audio-director` — 声音身份、情感基调、音频调色板
- `subagent_type: sound-designer` — SFX 规范、音频事件、混音组
- `subagent_type: technical-artist` — 音频中间件、bus 结构、内存预算
- `subagent_type: [primary engine specialist]` — 验证引擎的音频集成模式
- `subagent_type: gameplay-programmer` — 音频管理器、游戏触发器、自适应音乐

始终在每个 agent 的提示中提供完整上下文（功能描述、现有音频资产、设计文档引用）。

3. **按顺序编排音频团队**：

### 步骤 1：音频指导（audio-director）
生成 `audio-director` agent 来：
- 定义此功能/区域的声音身份
- 指定情感基调和音频调色板
- 设置音乐方向（自适应层、stems、过渡）
- 定义音频优先级和混音目标
- 建立任何自适应音频规则（战斗强度、探索、紧张感）

### 步骤 2：Sound Design 和音频无障碍（并行）
生成 `sound-designer` agent 来：
- 为每个音频事件创建详细的 SFX 规范
- 定义声音类别（环境、UI、游戏、音乐、对话）
- 指定每个声音的参数（音量范围、音高变化、衰减）
- 规划带有触发条件的音频事件列表
- 定义混音组和闪避规则

并行生成 `accessibility-specialist` agent 来：
- 识别哪些音频事件携带关键游戏信息（受到伤害、敌人在附近、目标完成）并需要为听障玩家提供视觉替代方案
- 指定字幕要求：哪些音频事件需要字幕、什么文本格式、屏幕显示时长
- 检查没有游戏状态仅通过音频传达（所有必须有视觉回退）
- 审查音频事件列表中是否有任何可能对听觉敏感玩家造成问题的（高频警报、突然的大声事件）
- 输出：音频无障碍需求列表，集成到音频事件规范中

### 步骤 3：技术实现（并行）
生成 `technical-artist` agent 来：
- 设计音频中间件集成（Wwise/FMOD/原生）
- 定义音频 bus 结构和路由
- 指定每个平台的音频资产内存预算
- 规划流式传输 vs 预加载资产策略
- 设计任何音频反应视觉效果

并行生成**主要引擎专家**（来自 `.claude/docs/technical-preferences.md` 的 Engine Specialists）来验证集成方法：
- 提议的音频中间件集成对引擎来说是惯用的吗？（如 Godot 内置的 AudioStreamPlayer vs FMOD、Unity 的 Audio Mixer vs Wwise、Unreal 的 MetaSounds vs FMOD）
- 应该使用哪些引擎专属的音频节点/组件模式？
- 固定引擎版本中是否有影响集成计划的已知音频系统变更？
- 输出：引擎音频集成说明，与技术艺术家的计划合并

如果未配置引擎，跳过专家生成。

### 步骤 4：代码集成（gameplay-programmer）
生成 `gameplay-programmer` agent 来：
- 实现音频管理器系统或审查现有系统
- 将音频事件连接到游戏触发器
- 实现自适应音乐系统（如已指定）
- 设置音频遮挡/混响区域
- 为音频事件触发器编写单元测试

4. **编译音频设计文档**，结合所有团队输出。

5. **保存到** `design/audio/audio-[feature].md`。

   注意：如果 `design/audio/` 不存在，写入文档的子 agent 应该创建它（写入文件时会自动创建目录）。

6. **输出摘要**，包括：音频事件数量、估算资产数量、
   实现任务，以及团队成员之间的任何开放问题。

裁决：**COMPLETE** — 音频设计文档已生成，团队管线已完成。

如果管线因依赖关系未解决而停止（如关键无障碍缺口或用户未解决的缺失 GDD）：

裁决：**BLOCKED** — [原因]

## 文件写入协议

所有文件写入（音频设计文档、SFX 规范、实现文件）都委托给
通过 Task 生成的子 agent。每个子 agent 执行"May I write to [path]?"
协议。此编排器不直接写入文件。

## 后续步骤

- 在实现开始前与 audio-director 审查音频设计文档。
- 设计批准后使用 `/dev-story` 实现音频管理器和事件系统。
- 创建音频资产后运行 `/asset-audit` 验证命名和格式合规性。

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
