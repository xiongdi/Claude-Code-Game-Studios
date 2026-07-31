# Skill 测试规格：/start

## Skill 摘要

`/start` 是新项目的首次 onboarding skill。它引导用户
完成项目命名、游戏引擎选择和初始目录结构设置。
它创建存根配置文件（CLAUDE.md、technical-preferences.md），
然后将路由传递给 `/setup-engine`，把选定的引擎作为参数。
每个创建的文件或目录都需要通过"可以写入吗"询问，遵循协作协议。

该 skill 检测项目是否已配置以及是否存在部分设置，
提供恢复或重新启动的选项。它没有 director 关卡——
它是一个在任何 agent 层级存在之前运行的实用设置 skill。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：COMPLETE、BLOCKED
- [ ] 为每个配置文件包含"可以写入吗"协作协议语言
- [ ] 末尾有下一步交接（路由到 `/setup-engine`）

---

## Director 关卡检查

无。`/start` 是实用设置 skill。在该 skill 运行的时刻
尚不存在任何 director agent。

---

## 测试用例

### 用例 1：正常路径——全新仓库，无引擎，完整 onboarding 流程

**Fixture：**
- 空仓库：无 CLAUDE.md 覆盖，无 `production/stage.txt`，无
  `technical-preferences.md` 内容（仅有占位符）
- 无现有设计文档或源代码

**输入：** `/start`

**预期行为：**
1. Skill 检测到无现有配置并开始全新 onboarding
2. Skill 询问项目名称
3. Skill 提供 3 个引擎选项：Godot 4、Unity、Unreal Engine 5
4. 用户选择引擎
5. Skill 询问"可以写入初始目录结构吗？"
6. Skill 创建 `directory-structure.md` 中定义的所有目录
7. Skill 询问"可以写入 CLAUDE.md 存根吗？"并在批准后写入
8. Skill 路由到 `/setup-engine [chosen-engine]` 完成技术配置

**断言：**
- [ ] 在写入任何文件前捕获项目名称
- [ ] 恰好提供 3 个引擎选项
- [ ] 对每个配置文件单独询问"可以写入吗"
- [ ] 未经用户明确批准不写入任何文件
- [ ] 末尾交接给 `/setup-engine`，附带选定的引擎参数
- [ ] 所有文件写入并发出交接后，判定为 COMPLETE

---

### 用例 2：已配置——检测现有配置，提供跳过或重新配置选项

**Fixture：**
- `technical-preferences.md` 的引擎已设置（不是占位符）
- `production/stage.txt` 存在，内容为 `Concept`

**输入：** `/start`

**预期行为：**
1. Skill 读取 `technical-preferences.md` 并检测到已配置的引擎
2. Skill 报告："This project is already configured with [engine]"
3. Skill 提供选项：跳过（退出）、重新配置引擎、或重新配置特定部分
4. 如果用户选择跳过：skill 干净退出并总结当前配置
5. 如果用户选择重新配置：skill 进入引擎选择步骤

**断言：**
- [ ] Skill 未经用户选择重新配置不会覆盖现有配置
- [ ] 在状态消息中向用户显示检测到的引擎名称
- [ ] 至少提供 2 个选项（跳过或重新配置）
- [ ] 无论用户跳过还是重新配置，判定均为 COMPLETE

---

### 用例 3：引擎选择——用户选择 Godot 4，路由到 /setup-engine godot

**Fixture：**
- 全新仓库——无现有配置

**输入：** `/start`

**预期行为：**
1. Skill 提供引擎选项，用户选择 Godot 4
2. Skill 在批准后写入初始存根（目录结构、CLAUDE.md）
3. Skill 明确将下一步路由到 `/setup-engine godot`
4. 交接消息清楚地命名引擎和下一个 skill 调用

**断言：**
- [ ] 交接命令是 `/setup-engine godot`（不是通用的 `/setup-engine`）
- [ ] 在所有初始存根写入后发出交接，而不是之前
- [ ] 引擎选择在写入开始前向用户回显

---

### 用例 4：中断的设置——检测到部分配置，提供恢复或重新启动

**Fixture：**
- 目录结构存在（已创建），但 `technical-preferences.md` 仍然是
  全部占位符（引擎从未选择——设置被中断）
- 无 `production/stage.txt`

**输入：** `/start`

**预期行为：**
1. Skill 检测到部分状态：目录存在但引擎未配置
2. Skill 报告："A partial setup was detected——directories exist but engine is not configured"
3. Skill 提供：从引擎选择恢复，或从头重新启动
4. 如果恢复：skill 跳过目录创建，进入引擎选择
5. 如果重新启动：skill 在继续前询问"可以覆盖现有结构吗？"

**断言：**
- [ ] 正确识别部分状态（目录存在，引擎缺失）
- [ ] 向用户提供恢复 vs. 重新启动选择——不强制走单一路径
- [ ] 恢复路径跳过重新创建目录（不对结构进行冗余的"可以写入吗"询问）
- [ ] 重新启动路径在触及任何文件前请求覆盖许可

---

### 用例 5：Director 关卡检查——无关卡；start 是实用设置 skill

**Fixture：**
- 任何 fixture

**输入：** `/start`

**预期行为：**
1. Skill 完成完整 onboarding 流程
2. 在任何时候都不派生 director agent
3. 输出中不出现 gate ID（CD-*、TD-*、AD-*、PR-*）

**断言：**
- [ ] 在 skill 执行期间未调用 director 关卡
- [ ] 不出现 gate 跳过消息（gate 是缺失的，不是被抑制的）
- [ ] Skill 在没有任何 gate 判定的情况下达到 COMPLETE

---

## 协议合规性

- [ ] 在写入任何文件前询问项目名称
- [ ] 将引擎选项呈现为结构化选择（不是自由文本）
- [ ] 对目录结构和 CLAUDE.md 存根分别询问"可以写入吗"
- [ ] 以交接给 `/setup-engine` 并附带引擎名称作为参数结束
- [ ] 在输出末尾明确陈述判定（COMPLETE 或 BLOCKED）

---

## 覆盖说明

- 用户拒绝所有引擎选项并提供自定义引擎名称的情况未测试——
  skill 仅为三个受支持引擎设计。
- Git 初始化（如有）不在此处测试；这是 skill 边界之外的
  基础设施问题。
- Solo vs. lean 模式行为不适用——此 skill 没有关卡，
  模式选择无关紧要。
