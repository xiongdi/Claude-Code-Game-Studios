---
name: localize
description: "Full localization pipeline: scan for hardcoded strings, extract and manage string tables, validate translations, generate translator briefings, run cultural/sensitivity review, manage VO localization, test RTL/platform requirements, enforce string freeze, and report coverage."
argument-hint: "[scan|extract|validate|status|brief|cultural-review|vo-pipeline|rtl-check|freeze|qa]"
user-invocable: true
agent: localization-lead
allowed-tools: Read, Glob, Grep, Write, Bash, Task, AskUserQuestion
model: sonnet
---

# Localization Pipeline

本地化不仅仅是翻译——它是使游戏在每个语言和地区都感觉原生的完整过程。糟糕的本地化会破坏沉浸感、让玩家困惑并阻止平台认证。此技能涵盖从字符串提取到文化审查、VO 录制、RTL 布局测试和本地化 QA 签字的完整管道。

**模式：**
- `scan` — 查找硬编码字符串和本地化反模式（只读）
- `extract` — 提取字符串并生成可翻译的表
- `validate` — 检查翻译的完整性、占位符和长度
- `status` — 跨所有语言环境的内容矩阵
- `brief` — 为外部团队生成翻译上下文简报文档
- `cultural-review` — 标记文化敏感内容、符号、颜色、习语
- `vo-pipeline` — 管理语音本地化：脚本、录制规范、集成
- `rtl-check` — 验证 RTL 语言布局、镜像和字体支持
- `freeze` — 执行字符串冻结；在翻译开始前锁定源字符串
- `qa` — 在发布前运行完整的本地化 QA 循环

如果未提供子命令，输出用法并停止。裁决：**FAIL**——缺少必需的子命令。

---

## 第 2A 阶段：扫描模式

搜索 `src/` 中硬编码的用户面向字符串：

- UI 代码中未包装在本地化函数中的字符串字面量（`tr()`、`Tr()`、`NSLocalizedString`、`GetText` 等）
- 应该参数化的连接字符串
- 使用位置占位符（`%s`、`%d`）而不是命名占位符（`{playerName}`）的字符串
- 混合区域敏感数据（数字、日期、货币）而未使用区域感知格式化的格式字符串

搜索本地化反模式：

- 未使用区域感知函数的日期/时间格式化
- 没有区域感知的数字格式化（`1,000` 与 `1.000`）
- 嵌入图像或纹理中的字符串（标记 `assets/` 中的资产文件）
- 假设从左到右文本方向的字符串（位置布局、字符串组装顺序）
- 硬编码到字符串逻辑中的性别/复数假设（必须使用复数形式或性别标记）
- 硬编码标点符号（例如 `"You won!"` — 感叹风格因区域而异）

报告所有发现及文件路径和行号。此模式是只读的——不写入文件。

---

## 第 2B 阶段：提取模式

- 扫描所有源文件中的本地化字符串引用
- 与 `assets/data/strings/` 中的现有字符串表进行比较
- 为尚未键入的字符串生成新条目
- 按照约定建议键名：`[category].[subcategory].[description]`
  - 示例：`ui.hud.health_label`、`dialogue.npc.merchant.greeting`、`menu.main.play_button`
- 每个新条目必须包含一个 `context` 字段——翻译者注释，解释：
  - 它出现在哪里（哪个屏幕、哪个场景）
  - 最大字符长度
  - 任何占位符含义（`{playerName}` = 玩家选择的显示名称）
  - 如适用，性别/复数上下文

输出要添加到字符串表的新字符串的差异。

向用户展示差异。询问："我可以将这些新条目写入 `assets/data/strings/strings-en.json` 吗？"

如果同意，仅写入差异（新条目），不是完整替换。裁决：**COMPLETE**——字符串已提取并写入。

---

## 第 2C 阶段：验证模式

读取 `assets/data/strings/` 中的所有字符串表文件。对于每个语言环境，检查：

- **完整性**——键存在于源（en）中但此语言环境没有翻译
- **占位符不匹配**——源有 `{name}` 但翻译省略了它或添加了额外的
- **字符串长度违规**——翻译超过源 `context` 字段中记录的字符限制
- **复数形式计数**——语言环境需要 N 种复数形式；翻译提供的更少
- **孤立键**——翻译存在但 `src/` 中没有任何内容引用该键
- **过时翻译**——源字符串在翻译编写后更改（标记为重新翻译）
- **编码**——存在非 ASCII 字符且字体图集支持它们（如果不确定则标记）

按语言环境和严重性分组报告验证结果。此模式是只读的——不写入文件。

---

## 第 2D 阶段：状态模式

- 计算源表中可本地化的字符串总数
- 每个语言环境：计算已翻译、未翻译、过时（源自翻译后更改）
- 生成内容矩阵：

```markdown
## 本地化状态
生成日期：[日期]
字符串冻结：[Active / 尚未调用 / 已解除]

| 语言环境 | 总数 | 已翻译 | 缺失 | 过时 | 覆盖率 |
|--------|-------|-----------|---------|-------|----------|
| en（源） | [N] | [N] | 0 | 0 | 100% |
| [locale] | [N] | [N] | [N] | [N] | [X]% |

### 问题
- [N] 个硬编码字符串在源代码中找到（运行 /localize scan）
- [N] 个字符串超过字符限制
- [N] 个占位符不匹配
- [N] 个孤立键
- [N] 个字符串在冻结后添加（冻结违规）
```

此模式是只读的——不写入文件。

---

## 第 2E 阶段：简报模式

生成翻译上下文简报文档。此文档与字符串表导出一起发送给外部翻译团队或本地化供应商。

读取：
- `design/gdd/` — 提取游戏类型、tone、设定、角色名称
- `assets/data/strings/strings-en.json` — 源字符串表
- `design/narrative/` 中的任何现有 lore 或叙事文档

生成 `production/localization/translator-brief-[locale]-[date].md`：

```markdown
# 翻译简报 — [游戏名称] — [语言环境]

## 游戏概述
[2-3 段游戏、类型、tone 和受众的摘要]

## Tone 和 Voice
- **整体 tone**：[例如，"黑色幽默，不是闹剧——想想 Terry Pratchett，不是 Looney Tunes"]
- **玩家称呼**：[例如，"第二人称，非正式。永远不要正式的 'vous' —— 法语总是用 'tu'"]
- **脏话政策**：[例如，"温和 —— PG-13 等效。匹配源的强度，不要柔化或升级"]
- **幽默**：[例如，"存在文字游戏 —— 如果双关语无法翻译，发明一个等效的当地笑话；不要直译"]

## 角色词汇表
| 名称 | 角色 | 个性 | 备注 |
|------|------|-------------|-------|
| [名称] | [角色] | [个性] | [不要翻译 / 音译为 X] |

## 世界词汇表
| 术语 | 含义 | 备注 |
|------|---------|-------|
| [术语] | [它的意思] | [保留英文 / 翻译为 X] |

## 不翻译列表
以下内容必须逐字出现在所有语言环境中：
- [游戏名称]
- [与引擎内标签匹配的 UI 术语]
- [品牌或商标名称]

## 占位符参考
| 占位符 | 它代表什么 | 示例 |
|-------------|-------------------|---------|
| `{playerName}` | 玩家选择的显示名称 | "Shadowblade" |
| `{count}` | 整数数量 | "3" |

## 字符限制
具有严格限制的紧密 UI 字段在字符串表的 `context` 字段中标记。
在未说明限制的地方，以英文长度的 ±30% 为目标作为指导。

## 联系
问题直接联系：[用户/团队联系方式的占位符]
交付格式：JSON，与 strings-en.json 相同的模式
```

询问："我可以将此翻译简报写入 `production/localization/translator-brief-[locale]-[date].md` 吗？"

---

## 第 2F 阶段：文化审查模式

通过 Task 生成 `localization-lead`。要求他们针对目标语言环境审计以下内容的文化敏感性（从 `assets/data/strings/` 和 `assets/` 读取）：

### 要审查的内容领域

**符号和手势**
- 竖起大拇指、OK 手势、和平标志——含义因地区而异
- 艺术、UI 或音频中的宗教或精神符号
- 国旗、地图表示、争议领土

**颜色**
- 白色（在一些亚洲文化中哀悼）、绿色（在一些地区政治关联）、红色（幸运 vs 危险）
- 与文化关联冲突的警报/警告颜色

**数字**
- 4（日语/中文中死亡）、13、666——在 UI 中标记使用（房间号、物品数量、价格）

**幽默和习语**
- 在其他语言环境中翻译为冒犯的习语
- 在一些市场（特别是日本、德国、中东）不适当的厕所/身体幽默
- 在特定地区文化敏感的主题上的黑色幽默

**暴力和内容评级**
- 需要在 DE（德国）、AU（澳大利亚）、CN（中国）或 AE（阿联酋）更改评级的内容
- 血液颜色、血腥程度、药物引用——如需要，标记所有地区特定资产变体

**名称和表征**
- 在目标语言环境中冒犯、亵渎或带有负面含义的角色名称
- 对国籍、宗教或种族群体的刻板印象表征

以表格形式展示发现：

| 发现 | 受影响的语言环境 | 严重性 | 建议操作 |
|---------|--------------------|----------|--------------------|
| [描述] | [语言环境] | [BLOCKING / ADVISORY / NOTE] | [更改 / 标记审查 / 接受] |

BLOCKING = 在发布该语言环境之前必须修复。ADVISORY = 建议更改。NOTE = 仅供参考。

询问："我可以将此文化审查报告写入 `production/localization/cultural-review-[date].md` 吗？"

---

## 第 2G 阶段：VO 管道模式

管理语音本地化过程。从参数确定子任务：

- `vo-pipeline scan` — 识别所有需要 VO 录制的对话行
- `vo-pipeline script` — 生成含导演备注的录制脚本
- `vo-pipeline validate` — 检查所有录制的 VO 文件是否存在且正确命名
- `vo-pipeline integrate` — 验证 VO 文件在代码/资产中正确引用

### VO 管道：扫描

读取 `assets/data/strings/` 和 `design/narrative/`。识别：
- 所有对话行（匹配 `dialogue.*` 的键）含源文本
- 已录制的行（音频文件存在于 `assets/audio/vo/` 中）
- 尚未录制的行

输出录制清单：

```
## VO 录制清单 — [日期]

| 键 | 角色 | 源行 | 状态 |
|-----|-----------|-------------|--------|
| dialogue.npc.merchant.greeting | Merchant | "Welcome, traveller." | 已录制 |
| dialogue.npc.merchant.haggle | Merchant | "That's my final offer." | 需要录制 |
```

### VO 管道：脚本

为每个角色生成录制脚本文档，按场景分组。包括：

- 角色名称和简要个性备注
- 完整对话行，含不寻常专有名词的发音指南
- 每行的情感/方向备注（`[温暖、欢迎]`、`[恼怒、简短]`）
- 作为对话中回应的任何行（提供上下文："玩家刚刚说了 X"）

询问："我可以将 VO 录制脚本写入 `production/localization/vo-scripts-[locale]-[date].md` 吗？"

### VO 管道：验证

Glob `assets/audio/vo/[locale]/` 中的所有 `.wav`/`.ogg` 文件。与 VO 清单交叉引用。报告：
- 缺失文件（脚本中有行，无音频文件）
- 额外文件（音频文件存在，无匹配的字符串键）
- 命名约定违规

### VO 管道：集成

Grep `src/` 中的 VO 音频引用。验证每个引用的路径是否存在于 `assets/audio/vo/[locale]/` 中。报告损坏的引用。

---

## 第 2H 阶段：RTL 检查模式

从右到左的语言（阿拉伯语、希伯来语、波斯语、乌尔都语）需要超越
仅仅翻译文本的布局镜像。此模式验证实现。

读取 `.claude/docs/technical-preferences.md` 以确定引擎。然后检查：

**布局镜像**
- 引擎中是否启用了 RTL 布局？（Godot：`Control.layout_direction`，Unity：`RTL Support` 包，Unreal：文本方向标志）
- 所有 UI 容器是否设置为自动镜像，还是位置硬编码？
- 进度条、血条和方向指示器是否正确镜像？

**文本渲染**
- 是否加载了支持阿拉伯语/希伯来语字符集的字体？
- 阿拉伯语文字是否以正确的连字（连接脚本）渲染？
- 数字是否按要求显示为东阿拉伯数字？

**字符串组装**
- 是否有任何字符串连接假设从左到右的阅读顺序？
- 句子中的 `{placeholder}` 位置在句子结构反转时是否正确工作？

**资产审查**
- 是否有带有方向箭头或不对称设计的 UI 图标需要镜像变体？
- 是否有任何文本嵌入图像资产需要 RTL 版本？

要检查的 Grep 模式：
- 场景/预制件文件中引擎特定的 RTL 标志
- 任何 `HBoxContainer`、`LinearLayout`、`HorizontalBox` 节点——验证 layout_direction 设置
- 对话或 UI 代码附近使用 `+` 的字符串连接

报告发现。标记 BLOCKING 问题（没有修复内容不可读）vs ADVISORY（美容改进）。

询问："我可以将此 RTL 检查报告写入 `production/localization/rtl-check-[date].md` 吗？"

---

## 第 2I 阶段：冻结模式

字符串冻结锁定源（英文）字符串表，以便翻译可以在源不在翻译者下方更改的情况下进行。

### freeze 调用

检查 `production/localization/freeze-status.md` 中的当前冻结状态（如果存在）。

如果已冻结：
> "字符串冻结当前为 ACTIVE（调用日期 [date]）。自冻结以来已添加或修改了 [N] 个字符串。这些是冻结违规——它们需要重新翻译或批准的冻结解除。"

如果未冻结，展示冻结前清单：

```
冻结前清单
[ ] 所有计划的 UI 屏幕已实现
[ ] 所有对话行已最终确定（无进一步叙事修订计划）
[ ] 所有系统字符串（错误消息、教程文本）已完成
[ ] /localize scan 显示零个硬编码字符串
[ ] /localize validate 显示源（en）中无占位符不匹配
[ ] 营销字符串（商店描述、成就）已最终确定
```

使用 `AskUserQuestion`：
- 提示："以上所有项目是否确认？调用字符串冻结会锁定源表。"
- 选项：`[A] 是 — 现在调用字符串冻结` / `[B] 否 — 我还有字符串要添加`

如果 [A]：写入 `production/localization/freeze-status.md`：

```markdown
# 字符串冻结状态

**状态**：ACTIVE
**调用日期**：[date]
**调用者**：[user]
**冻结时的总字符串数**：[N]

## 冻结后更改
[冻结后添加或修改的任何字符串由 /localize extract 自动列在此处]
```

### freeze 解除

如果参数包含 `lift`：将 `freeze-status.md` 状态更新为 `LIFTED`，记录原因和日期。警告："解除冻结需要重新翻译所有修改的字符串。通知翻译团队。"

### freeze 检查（自动集成到 extract 中）

当 `extract` 模式找到新字符串或修改的字符串且 `freeze-status.md` 显示 Status: ACTIVE 时——将新键追加到 `## 冻结后更改` 并警告：
> "⚠️ 字符串冻结处于活动状态。已添加 [N] 个新/修改的字符串。这些是冻结违规。在继续之前通知你的本地化供应商。"

---

## 第 2J 阶段：QA 模式

本地化 QA 是一个专门的通道，在翻译交付后但在任何语言环境发布之前运行。这与 `/validate`（检查完整性）不同——这是一个基于试玩的质量检查。

通过 Task 生成 `localization-lead`，提供：
- 要 QA 的目标语言环境
- 游戏中所有屏幕/流程的列表（来自 `design/gdd/` 或 `/content-audit` 输出）
- 当前 `/localize validate` 报告
- 文化审查报告（如果存在）

要求 localization-lead 生成涵盖以下内容的 QA 计划：

1. **功能字符串检查**——每个字符串在游戏中显示时没有截断、占位符错误或编码损坏
2. **UI 溢出检查**——超出 UI 边界的翻译字符串（即使在字符限制内，一些语言也会扩展）
3. **上下文准确性**——10% 的字符串样本在游戏中审查翻译准确性和自然措辞
4. **文化审查项目**——验证文化审查中的所有 BLOCKING 项目已解决
5. **VO 同步检查**——如果存在 VO，验证翻译后的口型同步或字幕时间是否可接受
6. **平台认证要求**——检查平台特定的本地化要求（年龄评级文本、法律声明、ESRB/PEGI/CERO 文本）

输出每个语言环境的 QA 裁决：

```
## 本地化 QA 裁决 — [语言环境]

**状态**：PASS / PASS WITH CONDITIONS / FAIL
**审查者**：localization-lead
**日期**：[日期]

### 发现
| ID | 领域 | 描述 | 严重性 | 状态 |
|----|------|-------------|----------|--------|
| LOC-001 | UI 溢出 | "Settings" 按钮文本在 [屏幕] 上溢出 | BLOCKING | 未解决 |
| LOC-002 | 翻译 | [键] 翻译是直译——听起来不自然 | ADVISORY | 未解决 |

### 条件（如果 PASS WITH CONDITIONS）
- [条件 1 — 发布前必须解决]

### 签字确认
[ ] 所有 BLOCKING 发现已解决
[ ] Producer 批准发布 [语言环境]
```

询问："我可以将此本地化 QA 报告写入 `production/localization/loc-qa-[locale]-[date].md` 吗？"

**门集成**：Polish → Release 门要求每个发布的语言环境的 PASS 或 PASS WITH CONDITIONS 裁决。FAIL 仅阻止该语言环境的发布——如果他们的 QA 通过，其他语言环境可能仍然继续。

---

## 第 3 阶段：规则和后续步骤

### 规则
- 英文（en）始终是源语言环境
- 每个字符串表条目必须包含一个 `context` 字段，含翻译者注释、字符限制和占位符含义
- 永远不要直接修改翻译文件——生成差异供审查
- 字符限制必须按 UI 元素定义并在 validate 模式中执行
- 字符串冻结必须在发送给翻译者之前调用——永远不要翻译移动目标
- RTL 支持必须从一开始就设计——改造 RTL 布局是昂贵的
- 对于游戏将在商业上销售的任何语言环境，文化审查是必需的
- VO 脚本必须包含导演备注——原始对话行产生平淡的录制

### 推荐工作流

```
/localize scan            → 查找硬编码字符串
/localize extract         → 构建字符串表
/localize freeze          → 发送给翻译者前锁定源
/localize brief           → 生成翻译者简报文档
[发送给翻译者]
/localize validate        → 检查返回的翻译
/localize cultural-review → 标记文化敏感内容
/localize rtl-check       → 如果发布阿拉伯语 / 希伯来语 / 波斯语
/localize vo-pipeline     → 如果发布配音 VO
/localize qa              → 完整的本地化 QA 通道
```

当 `qa` 对所有发布语言环境返回 PASS 时，在运行 `/gate-check release` 时包含 QA 报告路径。
