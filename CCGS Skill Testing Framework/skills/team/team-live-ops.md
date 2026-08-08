# 技能测试规格: /team-live-ops

## 技能摘要

协调 live-ops 团队完成一个 7 阶段规划流程，产出赛季或活动计划。协调 live-ops-designer、economy-designer、analytics-engineer、community-manager、narrative-director 和 writer。第 3 和第 4 阶段（经济设计和分析）同时运行。以一个需要用户审批的整合赛季计划结束，然后交接给制作。

---

## 静态断言（结构性）

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 在 File Write Protocol 章节中包含"可以写入吗"语言（委托给 sub-agent）
- [ ] 有 File Write Protocol 章节，说明编排器不直接写入文件
- [ ] 末尾有下一步交接，引用 `/design-review`、`/sprint-plan` 和 `/team-release`
- [ ] 在阶段转换时使用 `AskUserQuestion` 以在继续前获取用户批准
- [ ] 明确说明第 3 和第 4 阶段可以同时运行（并行派生）
- [ ] 存在错误恢复章节（或通过 BLOCKED 处理隐含）
- [ ] 输出文档章节指定路径在 `design/live-ops/seasons/` 下

---

## 测试用例

### 用例 1：正常路径——全部 7 个阶段完成，产出赛季计划

**Fixture：**
- `design/live-ops/economy-rules.md` 存在，有当前经济配置
- `design/live-ops/ethics-policy.md` 存在，有项目伦理政策
- 游戏概念文档在其标准路径存在
- 正在计划的新赛季名称无现有赛季文档

**输入：** `/team-live-ops "Season 2: The Frozen Wastes"`

**预期行为：**
1. 第 1 阶段：通过 Task 派生 `live-ops-designer`；接收赛季简报，包含范围、内容列表和留存机制；展示给用户
2. AskUserQuestion：用户在第 2 阶段开始前批准第 1 阶段输出
3. 第 2 阶段：通过 Task 派生 `narrative-director`；读取第 1 阶段赛季简报；产出叙事框架文档（主题、故事钩子、传说连接）；展示给用户
4. 第 3 和第 4 阶段（并行）：在等待任一结果前，通过两个 Task 调用同时派生 `economy-designer` 和 `analytics-engineer`；economy-designer 读取 `design/live-ops/economy-rules.md`
5. 第 5 阶段：并行派生 `narrative-director` 和 `writer` 产出游戏内叙事文本和面向玩家的文案；两者都读取第 2 阶段叙事框架文档
6. 第 6 阶段：通过 Task 派生 `community-manager`；读取赛季简报、经济设计和叙事框架；产出包含草稿文案的沟通日历
7. 第 7 阶段：收集所有阶段输出；展示整合赛季计划摘要，包含经济健康检查、分析准备、伦理审查和开放问题
8. AskUserQuestion：用户批准完整赛季计划
9. Sub-agent 在写入前询问"可以写入 `design/live-ops/seasons/S2_The_Frozen_Wastes.md`？"、`...analytics.md` 和 `...comms.md`
10. 判定：COMPLETE——赛季计划产出并交接给制作

**断言：**
- [ ] 全部 7 个阶段按顺序执行；第 3 和第 4 阶段作为并行 Task 调用发出
- [ ] 第 7 阶段整合摘要包含全部六个部分（赛季简报、叙事框架、经济设计、分析计划、内容清单、沟通日历）
- [ ] 第 7 阶段中的伦理审查部分明确引用 `design/live-ops/ethics-policy.md`
- [ ] 三个输出文档写入 `design/live-ops/seasons/`，命名约定正确
- [ ] 文件写入委托给 sub-agent——编排器不直接写入
- [ ] 最终输出中出现判定：COMPLETE
- [ ] 下一步引用 `/design-review`、`/sprint-plan` 和 `/team-release`

---

### 用例 2：发现伦理违规——奖励元素违反伦理政策

**Fixture：**
- 所有标准 live-ops fixture 存在（economy-rules.md、ethics-policy.md）
- `design/live-ops/ethics-policy.md` 明确禁止针对 18 岁以下玩家的 loot box
- economy-designer（第 3 阶段）提出"Mystery Chest"机制，有随机高级奖励且无保底计时器

**输入：** `/team-live-ops "Season 3: Shadow Tournament"`

**预期行为：**
1. 第 1-4 阶段正常进行；economy-designer 提出 Mystery Chest 机制
2. 第 7 阶段：编排器对照伦理政策审查第 3 阶段输出；识别 Mystery Chest 违反伦理政策中"无透明随机高级奖励"规则
3. 第 7 阶段摘要的伦理审查部分明确标记违规："ETHICS FLAG: Mystery Chest 机制在第 3 阶段经济设计中违反 [policy rule]。在解决前批准被阻塞。"
4. 在提供赛季计划批准前呈现 `AskUserQuestion`，附带解决选项
5. Skill 在伦理违规解决或用户明确放弃前不发出 COMPLETE 判定或写入输出文档

**断言：**
- [ ] 第 7 阶段伦理审查部分明确命名违规元素及其违反的政策规则
- [ ] Skill 在存在伦理违规时不自动批准赛季计划
- [ ] 使用 AskUserQuestion 浮现违规并提供解决选项（修订经济设计、带记录理由的覆盖、取消）
- [ ] 输出文档在违规未解决时不写入
- [ ] 如果用户选择修订：skill 重新派生 economy-designer 产出修正设计，然后返回第 7 阶段审查
- [ ] 判定：COMPLETE 仅在伦理标记清除后发出

---

### 用例 3：无参数——显示使用指引

**Fixture：**
- 任何项目状态

**输入：** `/team-live-ops`（无参数）

**预期行为：**
1. 第 1 阶段：检测到无参数
2. 输出："Usage: `/team-live-ops [season name or event description]` —— 提供要计划的赛季或实时活动的名称或描述。"
3. Skill 立即退出，不派生任何 subagent

**断言：**
- [ ] Skill 不猜测赛季名称或编造范围
- [ ] 错误消息包含带 argument-hint 的正确使用格式
- [ ] 在参数检查失败前不发出 Task 调用
- [ ] 不读取或写入任何文件

---

### 用例 4：并行阶段验证——第 3 和第 4 阶段同时运行

**Fixture：**
- 所有标准 live-ops fixture 存在
- 第 1 阶段（赛季简报）和第 2 阶段（叙事框架）已批准
- 第 3 阶段（economy-designer）和第 4 阶段（analytics-engineer）的输入相互独立

**输入：** `/team-live-ops "Season 1: The First Thaw"`（在第 3/4 阶段转换时观察）

**预期行为：**
1. 用户批准第 2 阶段后，编排器在等待任一结果前发出两个 Task 调用（economy-designer 和 analytics-engineer）
2. 两个 agent 都接收赛季简报作为上下文；analytics-engineer 不等待 economy-designer 输出就开始
3. Economy-designer 输出和 analytics-engineer 输出在第 5 阶段开始前一起收集
4. 如果两个并行 agent 中有一个阻塞，另一个继续；报告部分结果

**断言：**
- [ ] 第 3 和第 4 阶段的两个 Task 调用在等待任一结果前发出——它们不是顺序的
- [ ] Analytics-engineer 提示不包含 economy-designer 输出作为必需输入（输入是独立的）
- [ ] 如果 economy-designer 阻塞但 analytics-engineer 成功，分析输出被保留，阻塞通过 AskUserQuestion 浮现
- [ ] 第 5 阶段在第 3 和第 4 阶段结果都收集前不开始
- [ ] Skill 文档明确说明"第 3 和第 4 阶段可以同时运行"

---

### 用例 5：缺失伦理政策——`design/live-ops/ethics-policy.md` 不存在

**Fixture：**
- `design/live-ops/economy-rules.md` 存在
- `design/live-ops/ethics-policy.md` 不存在
- 所有其他 fixture 存在

**输入：** `/team-live-ops "Season 4: Desert Heat"`

**预期行为：**
1. 第 1-4 阶段进行；economy-designer 和 analytics-engineer 被给予伦理政策路径但它缺失
2. 第 7 阶段：编排器尝试运行伦理审查；检测到 `design/live-ops/ethics-policy.md` 缺失
3. 第 7 阶段摘要包含缺口标记："ETHICS REVIEW SKIPPED: `design/live-ops/ethics-policy.md` 未找到。经济设计未对照伦理政策审查。建议在生产开始前创建一个。"
4. Skill 仍完成赛季计划并达到 COMPLETE 判定，但缺口在输出和赛季设计文档中突出标记
5. 下一步包括创建伦理政策文档的建议

**断言：**
- [ ] Skill 在伦理政策文件缺失时不报错
- [ ] Skill 在文件缺失时不编造伦理政策规则
- [ ] 第 7 阶段摘要明确注明伦理审查被跳过及其原因
- [ ] 判定：COMPLETE 在文件缺失时仍可达成
- [ ] 缺口标记出现在赛季设计输出文档中（不只是对话中）
- [ ] 下一步建议创建 `design/live-ops/ethics-policy.md`

---

## 协议合规性

- [ ] `AskUserQuestion` 用于每个阶段转换——用户在下一阶段开始前批准
- [ ] 第 3 和第 4 阶段始终并行派生，不顺序进行
- [ ] File Write Protocol：编排器从不直接调用 Write/Edit——所有写入委托给 sub-agent
- [ ] 每个输出文档由相关 sub-agent 发出自己的"可以写入 [path]？"请求
- [ ] 第 7 阶段中的伦理审查始终明确引用伦理政策文件路径
- [ ] 错误恢复：任何 BLOCKED agent 立即浮现，附带 AskUserQuestion 选项（跳过/重试/停止）
- [ ] 如果任何阶段阻塞，生成部分报告——工作从不丢弃
- [ ] 判定：COMPLETE 仅在用户批准整合赛季计划后；如果存在未解决的伦理违规则为 BLOCKED
- [ ] 下一步始终包括 `/design-review`、`/sprint-plan` 和 `/team-release`

---

## 覆盖说明

- 第 5 阶段并行派生（narrative-director + writer）遵循与第 3/4 阶段相同的模式，但在此未单独测试——它使用用例 4 中验证的相同并行 Task 协议。
- "economy-rules.md 缺失"边缘情况未单独测试——它将作为 economy-designer 的 BLOCKED 结果浮现，并遵循用例 4 中隐式测试的标准错误恢复路径。
- 完整内容编写管线（第 5 阶段输出验证）由用例 1 正常路径整合摘要检查隐式验证。
- Community manager 沟通日历格式（发布前、发布日、赛季中、最后一周）由用例 1 隐式验证；不需要单独的边缘情况。
