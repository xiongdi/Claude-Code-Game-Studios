# Skill 测试规格：/team-audio

## Skill 摘要

协调音频团队完成一个四步流程：audio direction（audio-director）→ sound design + accessibility review 并行（sound-designer + accessibility-specialist）→ technical implementation + engine validation 并行（technical-artist + 主要引擎专家）→ code integration（gameplay-programmer）。在派生 agent 之前读取相关 GDD、sound bible（如存在）和现有音频资源列表。将所有输出编译为一份音频设计文档，保存到 `design/gdd/audio-[feature].md`。在每个步骤转换时使用 `AskUserQuestion`。当音频设计文档生成后裁决为 COMPLETE。当未配置引擎时优雅地跳过引擎专家派生。

---

## 静态断言（结构性）

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个步骤/阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 包含"File Write Protocol"章节
- [ ] 文件写入委托给 sub-agent——编排器不直接写入文件
- [ ] Sub-agent 在任何写入前强制执行"可以写入 [path]？"协议
- [ ] 末尾有下一步交接（引用 `/dev-story`、`/asset-audit`）
- [ ] Error Recovery Protocol 章节存在
- [ ] 在步骤转换时使用 `AskUserQuestion` 以在继续前获取用户批准
- [ ] Step 2 明确并行派生 sound-designer 和 accessibility-specialist
- [ ] Step 3 明确并行派生 technical-artist 和引擎专家（当引擎配置时）
- [ ] Skill 在上下文收集期间如存在则读取 `design/gdd/sound-bible.md`
- [ ] 输出文档保存到 `design/gdd/audio-[feature].md`

---

## 测试用例

### 用例 1：正常路径——所有步骤完成，音频设计文档保存

**Fixture：**
- 目标功能的 GDD 在 `design/gdd/combat.md` 存在
- Sound bible 在 `design/gdd/sound-bible.md` 存在
- 现有音频资源在 `assets/audio/` 中列出
- 引擎在 `.claude/docs/technical-preferences.md` 中配置
- 计划的音频事件列表中不存在无障碍缺口

**输入：** `/team-audio combat`

**预期行为：**
1. 上下文收集：编排器在派生任何 agent 前读取 `design/gdd/combat.md`、`design/gdd/sound-bible.md` 和 `assets/audio/` 资源列表
2. Step 1：派生 audio-director；定义声音身份、情感基调、自适应音乐方向、混音目标和战斗的自适应音频规则
3. `AskUserQuestion` 展示音频方向；用户在 Step 2 开始前批准
4. Step 2：并行派生 sound-designer 和 accessibility-specialist；sound-designer 产出 SFX 规格、带触发条件的音频事件列表和混音组；accessibility-specialist 识别关键游戏音频事件并指定视觉回退和字幕要求
5. `AskUserQuestion` 展示 SFX 规格和无障碍要求；用户在 Step 3 开始前批准
6. Step 3：并行派生 technical-artist 和主要引擎专家；technical-artist 设计总线结构、中间件集成、内存预算和流策略；引擎专家验证集成方法对所配置引擎是惯用的
7. `AskUserQuestion` 展示技术计划；用户在 Step 4 开始前批准
8. Step 4：派生 gameplay-programmer；将音频事件连接到游戏触发器、实现自适应音乐、设置遮挡区域、为音频事件触发器编写单元测试
9. 编排器将所有输出编译为单一音频设计文档
10. Subagent 在写入前询问"可以写入音频设计文档到 `design/gdd/audio-combat.md` 吗？"
11. 摘要输出列出：音频事件数量、预估资源数量、实现任务和任何开放问题
12. 判定：COMPLETE

**断言：**
- [ ] Sound bible 在上下文收集期间（Step 1 前）存在时被读取
- [ ] audio-director 在 sound-designer 或 accessibility-specialist 之前派生
- [ ] `AskUserQuestion` 在 Step 1 输出后和 Step 2 启动前出现
- [ ] sound-designer 和 accessibility-specialist Task 调用在 Step 2 同时发出
- [ ] technical-artist 和引擎专家 Task 调用在 Step 3 同时发出
- [ ] gameplay-programmer 在 Step 3 `AskUserQuestion` 批准前不启动
- [ ] 音频设计文档写入 `design/gdd/audio-combat.md`（不是其他路径）
- [ ] 摘要包含音频事件数量和预估资源数量
- [ ] 编排器不直接写入任何文件
- [ ] 文档交付后判定为 COMPLETE

---

### 用例 2：无障碍缺口——关键游戏音频事件无视觉回退

**Fixture：**
- 目标功能的 GDD 存在
- Step 1 和 Step 2 进行中
- sound-designer 的音频事件列表包括"EnemyNearbyAlert"——一个空间音频提示，警告玩家有敌人从屏幕外接近
- accessibility-specialist 审查事件列表，发现"EnemyNearbyAlert"没有视觉回退（无屏幕指示器、无字幕、无手柄震动指定）

**输入：** `/team-audio stealth`（Step 2 场景）

**预期行为：**
1. Step 1-2 进行；accessibility-specialist 和 sound-designer 并行派生
2. accessibility-specialist 返回审查结果，带有 BLOCKING 问题："`EnemyNearbyAlert` 是关键游戏音频事件（警告玩家屏幕外威胁），无视觉回退——听障玩家无法检测此威胁。这是 BLOCKING 无障碍缺口。"
3. 编排器在呈现 `AskUserQuestion` 前立即在对话中浮现问题
4. `AskUserQuestion` 将无障碍问题展示为 BLOCKING 问题，选项：
   - 为 EnemyNearbyAlert 添加视觉指示器（例如，HUD 上的方向箭头）并继续
   - 添加手柄触觉反馈作为回退并继续
   - 在此停止并在进入 Step 3 前解决所有无障碍缺口
5. Step 3（technical-artist + 引擎专家）在用户解决或明确接受缺口前不启动
6. 如果未解决，无障碍缺口包含在最终音频设计文档的"Open Accessibility Issues"下

**断言：**
- [ ] 无障碍缺口在报告中标记为 BLOCKING（不是建议）
- [ ] 说明具体事件名称（"EnemyNearbyAlert"）和缺口性质
- [ ] `AskUserQuestion` 在 Step 3 启动前浮现缺口
- [ ] 至少提供一个解决选项（添加视觉回退、添加触觉回退）
- [ ] Step 3 在缺口未解决且无用户明确授权时不启动
- [ ] 如果缺口未解决被带入，它作为开放问题记录在音频设计文档中

---

### 用例 3：无参数——使用指引或设计文档推断

**Fixture：**
- 任何项目状态

**输入：** `/team-audio`（无参数）

**预期行为：**
1. Skill 检测到未提供参数
2. 输出使用指引：例如，"Usage: `/team-audio [feature or area]` —— 指定要设计音频的功能或区域（例如，`combat`、`main menu`、`forest biome`、`boss encounter`）"
3. Skill 退出，不派生任何 agent

**断言：**
- [ ] Skill 在未提供参数时不派生任何 agent
- [ ] 使用消息包含带参数示例的正确调用格式
- [ ] Skill 不从现有设计文档推断功能而不经用户指示
- [ ] 不使用 `AskUserQuestion`——输出为直接指引

---

### 用例 4：缺失 Sound Bible——Skill 注明缺口并在没有它的情况下继续

**Fixture：**
- 目标功能的 GDD 在 `design/gdd/main-menu.md` 存在
- `design/gdd/sound-bible.md` 不存在
- 引擎已配置；其他上下文文件存在

**输入：** `/team-audio main menu`

**预期行为：**
1. 上下文收集：编排器读取 `design/gdd/main-menu.md` 并检查 `design/gdd/sound-bible.md`
2. Sound bible 未找到；编排器在对话中注明缺口："Note: `design/gdd/sound-bible.md` 未找到——音频方向将在无项目范围声音身份参考的情况下继续。如果这是持续项目，建议创建 sound bible。"
3. 管线正常通过全部四个步骤，不以 sound bible 作为输入
4. Step 1 中的 audio-director 被告知不存在 sound bible，必须仅从功能 GDD 建立声音身份
5. 缺失的 sound bible 在最终摘要中被提及为推荐的下一步

**断言：**
- [ ] 编排器在上下文收集期间（Step 1 前）检查 sound bible
- [ ] 缺失的 sound bible 在对话中明确注明——不静默忽略
- [ ] 管线不因缺失 sound bible 而停止
- [ ] audio-director 在其提示上下文中被告知不存在 sound bible
- [ ] 摘要或 Next Steps 章节建议创建 sound bible
- [ ] 如果所有其他步骤成功，判定仍为 COMPLETE

---

### 用例 5：引擎未配置——引擎专家步骤被优雅跳过

**Fixture：**
- 引擎在 `.claude/docs/technical-preferences.md` 中未配置（显示 `[TO BE CONFIGURED]`）
- 目标功能的 GDD 存在
- Sound bible 可能存在也可能不存在

**输入：** `/team-audio boss encounter`

**预期行为：**
1. 上下文收集：编排器读取 `.claude/docs/technical-preferences.md` 并检测到未配置引擎
2. Step 1-2 正常进行（audio-director、sound-designer、accessibility-specialist）
3. Step 3：technical-artist 正常派生；引擎专家派生被跳过
4. 编排器在对话中注明："Engine specialist not spawned — no engine configured in technical-preferences.md. Engine integration validation will be deferred until an engine is selected."
5. Step 4：gameplay-programmer 继续，附带说明引擎特定音频集成模式无法被验证的注释
6. 引擎专家缺口包含在音频设计文档的"Deferred Validation"下
7. 判定：COMPLETE（跳过是优雅的，不是阻塞项）

**断言：**
- [ ] 引擎专家在未配置引擎时不被派生
- [ ] Skill 不错误退出，因为缺失引擎配置
- [ ] 跳过在对话中明确注明——不静默省略
- [ ] technical-artist 在 Step 3 仍被派生（跳过仅适用于引擎专家）
- [ ] gameplay-programmer 在 Step 4 继续，附带延期验证的注释
- [ ] 延期引擎验证记录在音频设计文档中
- [ ] 判定为 COMPLETE（引擎未配置是已知的优雅情况）

---

## 协议合规性

- [ ] 上下文收集（GDD、sound bible、资源列表）在任何 agent 派生前运行
- [ ] `AskUserQuestion` 在每个步骤输出后和下一步启动前使用
- [ ] 并行派生：Step 2（sound-designer + accessibility-specialist）和 Step 3（technical-artist + 引擎专家）在等待结果前发出所有 Task 调用
- [ ] 编排器不直接写入任何文件——所有写入委托给 sub-agent
- [ ] 每个 sub-agent 在任何写入前强制执行"可以写入 [path]？"协议
- [ ] 任何 agent 的 BLOCKED 状态立即浮现——不静默跳过
- 当一些 agent 完成而另一些阻塞时，始终生成部分报告
- [ ] 音频设计文档路径遵循模式 `design/gdd/audio-[feature].md`
- [ ] 判定严格为 COMPLETE 或 BLOCKED——不使用其他判定值
- [ ] Next Steps 交接引用 `/dev-story` 和 `/asset-audit`

---

## 覆盖说明

- Error Recovery Protocol 中的"用更窄范围重试"和"跳过此 agent"解决路径未单独测试——它们遵循用例 2 和 5 中验证的相同 `AskUserQuestion` + 部分报告模式。
- Step 4（gameplay-programmer）正常路径行为由用例 1 隐式验证。
  此步骤的故障模式遵循标准 Error Recovery Protocol。
- Accessibility-specialist 的字幕和字幕要求（超出视觉回退）由用例 1 隐式验证。用例 2 关注更严重的情况，即关键游戏事件完全没有回退。
- 引擎专家验证逻辑（惯用集成、版本特定变化）仅针对配置和未配置状态测试。
  引擎专家输出的具体内容在此行为规格范围之外。
