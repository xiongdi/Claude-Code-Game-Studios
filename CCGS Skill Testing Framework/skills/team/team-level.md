# 技能测试规格: /team-level

## 技能摘要

协调完整的关卡设计团队完成单个关卡或区域。协调 narrative-director、world-builder、level-designer、systems-designer、art-director、accessibility-specialist 和 qa-tester 五个顺序步骤（含一个并行阶段 Step 4）。将所有团队输出编译为一份关卡设计文档，保存到 `design/levels/[level-name].md`。在每个步骤转换时使用 `AskUserQuestion`。将所有文件写入委托给 sub-agent。生成包含 COMPLETE / BLOCKED 裁决的摘要报告，并交接给 `/design-review`、`/dev-story`、`/qa-plan`。

---

## 静态断言（结构性）

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段/步骤标题（Step 1 到 Step 5 全部存在）
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含"可以写入吗"或"File Write Protocol"语言——写入委托给 sub-agent，编排器不直接写入文件
- [ ] 末尾有下一步交接（引用 `/design-review`、`/dev-story`、`/qa-plan`）
- [ ] Error Recovery Protocol 章节存在，包含全部四个恢复步骤
- [ ] 在步骤转换时使用 `AskUserQuestion` 以在继续前获取用户批准
- [ ] Step 4 明确标记为并行（art-director 和 accessibility-specialist 同时运行）
- [ ] 上下文收集读取：`design/gdd/game-concept.md`、`design/gdd/game-pillars.md`、`design/levels/`、`design/narrative/` 和相关 world-building 文档
- [ ] Team Composition 列出全部七个角色（narrative-director、world-builder、level-designer、systems-designer、art-director、accessibility-specialist、qa-tester）
- [ ] accessibility-specialist 输出包含严重性评级（BLOCKING / RECOMMENDED / NICE TO HAVE）
- [ ] 最终关卡设计文档保存到 `design/levels/[level-name].md`

---

## 测试用例

### 用例 1：正常路径——所有团队成员产出输出，文档编译并保存

**Fixture：**
- `design/gdd/game-concept.md` 存在且已填充
- `design/gdd/game-pillars.md` 存在
- `design/levels/` 目录存在（可能包含其他关卡文档）
- `design/narrative/` 目录存在，有相关叙事文档

**输入：** `/team-level forest dungeon`

**预期行为：**
1. 上下文收集——编排器读取 game-concept.md、game-pillars.md、`design/levels/` 中的现有关卡文档、`design/narrative/` 中的叙事文档，以及森林区域的 world-building 文档
2. Step 1——派生 narrative-director：定义叙事目的、关键角色、对话触发器、情感弧线；派生 world-builder：提供传说背景、环境叙事机会、世界规则；`AskUserQuestion` 在 Step 2 前确认 Step 1 输出
3. Step 2——派生 level-designer：设计空间布局（关键路径、可选路径、秘密）、节奏曲线、遭遇战、谜题、入口/出口点及与相邻区域的连接；`AskUserQuestion` 在 Step 3 前确认布局
4. Step 3——派生 systems-designer：指定敌人组成、战利品表、难度平衡、区域特定机制、资源分布；`AskUserQuestion` 在 Step 4 前确认系统
5. Step 4——并行派生 art-director 和 accessibility-specialist；art-director：视觉主题、调色板、光照、资源列表、VFX 需求；accessibility-specialist：导航清晰度、色盲安全、认知负荷检查——每个问题评级为 BLOCKING / RECOMMENDED / NICE TO HAVE；`AskUserQuestion` 在 Step 5 前展示两个输出
6. Step 5——派生 qa-tester：关键路径测试用例、边界/边缘情况（序列破坏、软锁定）、试玩清单、验收标准
7. 编排器将所有团队输出编译为关卡设计文档格式；sub-agent 询问"可以写入 `design/levels/forest-dungeon.md` 吗？"；文件保存
8. 摘要报告：区域概览、遭遇战数量、预估资源列表、叙事节拍、跨团队依赖、判定：COMPLETE
9. 列出下一步：`/design-review design/levels/forest-dungeon.md`、`/dev-story`、`/qa-plan`

**断言：**
- [ ] 上下文收集期间在派生任何 agent 前读取全部五个来源
- [ ] Step 1 中同时派生 narrative-director 和 world-builder（可顺序或并行——两者都必须在 Step 2 前完成）
- [ ] 每个步骤门控都调用 `AskUserQuestion`（最少：Step 1、Step 2、Step 3、Step 4 后）
- [ ] Step 4 agent（art-director、accessibility-specialist）同时启动
- [ ] 所有文件写入委托给 sub-agent——编排器不直接写入
- [ ] 关卡文档保存到 `design/levels/forest-dungeon.md`（从参数 slug 化）
- [ ] 最终摘要报告中有判定 COMPLETE
- [ ] 下一步包括 `/design-review`、`/dev-story`、`/qa-plan`
- [ ] 摘要报告包括：区域概览、遭遇战数量、预估资源列表、叙事节拍

---

### 用例 2：阻塞的 Agent（world-builder）——产出部分报告并注明缺口

**Fixture：**
- `design/gdd/game-concept.md` 存在
- 森林区域的 world-building 文档不存在
- world-builder agent 返回 BLOCKED："未找到森林区域的 world-building 文档——无法提供传说背景"

**输入：** `/team-level forest dungeon`

**预期行为：**
1. 上下文收集完成；注明缺失的 world-building 文档
2. Step 1——narrative-director 成功完成；world-builder 被派生并返回 BLOCKED
3. 触发 Error Recovery Protocol："world-builder: BLOCKED——无森林区域 world-building 文档"
4. 呈现 `AskUserQuestion`，选项：
   - (a) 跳过 world-builder 并在关卡文档中注明传说缺口
   - (b) 用更窄范围重试（world-builder 仅从 game-concept.md 推断）
   - (c) 在此停止并先创建 world-building 文档
5. 如果用户选择 (a)：管线继续使用仅 narrative-director 上下文进行 Step 2-5；关卡文档编译时带有明确标记的缺口部分："World-building context: NOT PROVIDED — see open dependency"
6. 生成最终报告：记录部分输出，world-builder 部分标记为 BLOCKED，整体判定：BLOCKED

**断言：**
- [ ] BLOCKED 浮现消息在 world-builder 失败后立即出现——在 Step 2 未经用户输入开始前
- [ ] `AskUserQuestion` 至少提供三个选项（跳过/重试/停止）
- [ ] 生成部分报告——narrative-director 已完成的工作不丢弃
- [ ] 关卡文档（如编译）包含对缺失 world-building 上下文的明确缺口标注
- [ ] 整体判定为 BLOCKED（不是 COMPLETE），当 world-builder 仍未解决时
- [ ] Skill 不静默编造传说内容来填补缺口

---

### 用例 3：无参数——显示使用指引

**Fixture：**
- 任何项目状态

**输入：** `/team-level`（无参数）

**预期行为：**
1. Skill 检测到未提供参数
2. 输出使用消息解释所需参数（关卡名称或要设计的区域）
3. 提供示例调用：`/team-level tutorial`、`/team-level forest dungeon`、`/team-level final boss arena`
4. Skill 退出，不读取任何项目文件或派生任何 subagent

**断言：**
- [ ] Skill 在未提供参数时不派生任何 subagent
- [ ] 使用消息包含 frontmatter 中的 argument-hint 格式
- [ ] 显示至少一个有效调用的示例
- [ ] 失败前不读取任何 GDD 或关卡文件
- [ ] 不显示判定（管线从未启动）

---

### 用例 4：无障碍审查门控——签字前浮现阻塞性问题

**Fixture：**
- Step 1-3 成功完成
- `design/accessibility-requirements.md` 提交层级：Enhanced
- accessibility-specialist（Step 4，并行）标记一个 BLOCKING 问题：穿越森林地牢的关键路径要求玩家仅通过颜色区分两种环境危害（毒池 vs. 浅水）——没有形状、图标或音频提示区分它们

**输入：** `/team-level forest dungeon`

**预期行为：**
1. Step 1-3 完成；Step 4 并行阶段开始
2. accessibility-specialist 返回：BLOCKING 问题——"关键路径危害区分仅依赖颜色（毒池 vs. 浅水）。根据 Enhanced 无障碍层级，需要形状、图标或音频提示。"
3. art-director 返回 Step 4 输出（完成）
4. Skill 通过 `AskUserQuestion` 展示两个 Step 4 结果——突出显示 BLOCKING 问题
5. `AskUserQuestion` 提供：
   - (a) 在 Step 5 前返回 level-designer + art-director 重新设计危害的视觉/音频语言
   - (b) 记录为已知无障碍缺口并继续 Step 5，问题已记录
6. Skill 不静默继续通过 BLOCKING 问题
7. 如果用户选择 (a)：派生 level-designer 和 art-director 修订；重新运行 Step 4 无障碍检查
8. 无论用户选择如何，最终报告都包含 BLOCKING 问题及其解决状态

**断言：**
- [ ] BLOCKING 无障碍问题不被视为建议——它被浮现为阻塞项
- [ ] `AskUserQuestion` 展示具体问题文本（不只是"发现无障碍问题"）
- [ ] Step 5（qa-tester）在用户确认 BLOCKING 问题前不开始
- [ ] 提供修订路径：level-designer + art-director 可在继续前被送回
- [ ] 最终报告包含无障碍问题及其解决状态
- [ ] art-director 已完成的输出在 accessibility-specialist 阻塞时不丢弃

---

### 用例 5：循环关卡引用——标记相邻区域依赖

**Fixture：**
- Step 1-3 进行中
- level-designer（Step 2）产出的布局指定入口/出口点连接到"水晶洞穴"（一个相邻区域）
- `design/levels/crystal-caves.md` 不存在——水晶洞穴区域尚未设计

**输入：** `/team-level forest dungeon`

**预期行为：**
1. Step 2——level-designer 产出布局，包括："西出口连接到水晶洞穴入口点 A"
2. 编排器（或 level-designer subagent）检查 `design/levels/` 中的 `crystal-caves.md`；文件未找到
3. 浮现依赖缺口："关卡引用水晶洞穴作为相邻区域，但 `design/levels/crystal-caves.md` 不存在"
4. 呈现 `AskUserQuestion`，选项：
   - (a) 继续使用占位符引用——在关卡文档中将依赖注明为 UNRESOLVED
   - (b) 暂停并先运行 `/team-level crystal caves` 建立该区域
5. Skill 不编造水晶洞穴内容来满足引用
6. 如果用户选择 (a)：关卡文档编译时西出口标记为"→ crystal-caves (UNRESOLVED — area not yet designed)"；在摘要报告的开放依赖部分标记
7. 最终报告包含开放跨关卡依赖部分

**断言：**
- [ ] Skill 通过检查 `design/levels/` 检测缺失的相邻区域——不假设它将在之后创建
- [ ] Skill 不编造水晶洞穴内容（传说、布局、连接）来解决引用
- [ ] `AskUserQuestion` 提供"先设计水晶洞穴"选项，引用 `/team-level`
- [ ] 如果用户继续使用占位符，关卡文档明确将西出口标记为 UNRESOLVED
- [ ] 摘要报告包含开放跨关卡依赖部分，列出未解决的引用
- [ ] 循环或前向引用不会导致 skill 循环或崩溃

---

## 协议合规性

- [ ] `AskUserQuestion` 用于每个步骤转换——用户在管线推进前批准
- [ ] 所有文件写入通过 Task 委托给 sub-agent——编排器不直接调用 Write 或 Edit
- [ ] 遵循 Error Recovery Protocol：浮现 → 评估 → 提供选项 → 部分报告
- [ ] Step 4 agent（art-director、accessibility-specialist）按 skill 规格并行启动
- [ ] 即使 agent 被 BLOCKED 也始终生成部分报告
- [ ] Accessibility BLOCKING 问题在签字前浮现并需要用户明确确认
- [ ] 判定为 COMPLETE / BLOCKED 之一
- [ ] 末尾有下一步：`/design-review`、`/dev-story`、`/qa-plan`

---

## 覆盖说明

- Step 1 中的 narrative-director 和 world-builder 可顺序或并行——skill 规格
  派生两者但不强制同时启动；Step 1 的并行覆盖需要
  明确的时序断言 fixture。
- 阻塞 world-builder 用例（用例 2）中的"用更窄范围重试"选项——
  重试行为本身未深入测试；其完整路径类似于用例 2 和
  其他 team-* 规格中覆盖的阻塞 agent 模式。
- systems-designer（Step 3）阻塞场景未单独测试；适用相同的 Error Recovery
  Protocol，模式由用例 2 验证。
- Step 4 并行排序（art-director 在 accessibility-specialist 之前或之后完成）
  不影响结果——无论顺序如何，两者都必须在 Step 5 前返回。
- 关卡文档 slug 约定（参数 → 文件名）由用例 1 隐式测试
  （`forest dungeon` → `forest-dungeon.md`）；多词 slug 化边缘情况（特殊
  字符、非常长的名称）未覆盖。
