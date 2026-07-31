---
name: project-stage-detect
description: "Automatically analyze project state, detect stage, identify gaps, and recommend next steps based on existing artifacts. Use when user asks 'where are we in development', 'what stage are we in', 'full project audit'."
argument-hint: "[optional: role filter like 'programmer' or 'designer']"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write
model: haiku
# 只读诊断 skill — 无需委派给专家 agent
---

# 项目阶段检测

此 skill 扫描您的项目以确定其当前开发阶段、资源的完整性以及需要注意的缺口。它在以下情况下特别有用：
- 从现有项目开始时
- 加入代码库时
- 在里程碑之前检查缺失内容时
- 理解"我们在哪里？"

---

## 工作流

### 1. 扫描关键目录

分析项目结构和内容：

**设计文档** (`design/`):
- 统计 `design/gdd/*.md` 中的 GDD 文件数量
- 检查是否存在 game-concept.md、game-pillars.md、systems-index.md
- 如果 systems-index.md 存在，统计总系统数 vs 已设计的系统数
- 分析完整性（Overview、Detailed Design、Edge Cases 等）
- 统计 `design/narrative/` 中的叙事文档数量
- 统计 `design/levels/` 中的关卡设计数量

**源代码** (`src/`):
- 统计源文件数量（语言无关）
- 识别主要系统（包含 5+ 文件的目录）
- 检查是否存在 core/、gameplay/、ai/、networking/、ui/ 目录
- 估算代码行数（粗略规模）

**制作资源** (`production/`):
- 检查是否存在活跃的 sprint 计划
- 查找里程碑定义
- 查找路线图文档

**原型** (`prototypes/`):
- 统计原型目录数量
- 检查是否有 README（有文档 vs 无文档）
- 评估原型是已归档还是活跃的

**架构文档** (`docs/architecture/`):
- 统计 ADR（架构决策记录）数量
- 检查是否存在 overview/index 文档

**测试** (`tests/`):
- 统计测试文件数量
- 估算测试覆盖率（粗略启发式）

### 2. 分类项目阶段

基于扫描的资源确定阶段。首先检查 `production/stage.txt` —
如果存在，使用其值（来自 `/gate-check` 的显式覆盖）。否则，
使用以下启发式规则自动检测（从最先进阶段向后检查）：

| 阶段 | 指标 |
|-------|-----------|
| **Concept** | 无游戏概念文档，处于头脑风暴阶段 |
| **Systems Design** | 存在游戏概念，systems index 缺失或不完整 |
| **Technical Setup** | 存在 systems index，引擎未配置 |
| **Pre-Production** | 引擎已配置，`src/` 中少于 10 个源文件 |
| **Production** | `src/` 中有 10+ 个源文件，活跃开发中 |
| **Polish** | 仅显式设置（由 `/gate-check` Production → Polish gate 设置） |
| **Release** | 仅显式设置（由 `/gate-check` Polish → Release gate 设置） |

### 3. 协作式缺口识别

**不要**只列出缺失的文件。相反，**提出澄清性问题**：

- "我看到战斗代码（`src/gameplay/combat/`）但没有 `design/gdd/combat-system.md`。这是先做的原型，还是需要反向文档化？"
- "你有 15 个 ADR 但没有架构概览。我应该创建一个来帮助新贡献者吗？"
- "`production/` 中没有 sprint 计划。你在其他地方跟踪工作吗（Jira、Trello 等）？"
- "我找到了游戏概念但没有 systems index。你是否已将概念分解为各个系统，还是我们应该运行 `/map-systems`？"
- "Prototypes 目录有 3 个项目但没有 README。这些是实验，还是需要文档化？"

### 4: 生成阶段报告

使用模板：`.claude/docs/templates/project-stage-report.md`

**报告结构**：
```markdown
# 项目阶段分析

**日期**: [日期]
**阶段**: [Concept/Systems Design/Technical Setup/Pre-Production/Production/Polish/Release]
**阶段置信度**: [PASS — 明确检测 / CONCERNS — 信号模糊 / FAIL — 关键缺口阻塞进度]

## 完整性概览
- 设计: [X%] ([N] 个文档，[缺口])
- 代码: [X%] ([N] 个文件，[系统])
- 架构: [X%] ([N] 个 ADR，[缺口])
- 制作: [X%] ([状态])
- 测试: [X%] ([覆盖率估算])

## 已识别缺口
1. [缺口描述 + 澄清性问题]
2. [缺口描述 + 澄清性问题]

## 推荐后续步骤
[基于阶段和角色的优先级排序列表]
```

### 5: 按角色过滤的推荐（可选）

如果用户提供了角色参数（例如 `/project-stage-detect programmer`）：

**Programmer**:
- 关注架构文档、测试覆盖率、缺失的 ADR
- 代码与文档之间的缺口

**Designer**:
- 关注 GDD 完整性、缺失的设计部分
- 原型文档化

**Producer**:
- 关注 sprint 计划、里程碑跟踪、路线图
- 跨团队协调文档

**General**（无角色）：
- 所有缺口的整体视图
- 跨领域最高优先级项目

### 6: 写入前请求批准

**协作协议**：
```
我已分析您的项目。这是我的发现：

[展示摘要]

已识别缺口：
1. [缺口 1 + 问题]
2. [缺口 2 + 问题]

推荐后续步骤：
- [优先级 1]
- [优先级 2]
- [优先级 3]

我可以将完整的阶段分析写入 production/project-stage-report.md 吗？
```

在创建文件之前等待用户批准。

---

## 示例用法

```bash
# 通用项目分析
/project-stage-detect

# 面向 programmer 的分析
/project-stage-detect programmer

# 面向 designer 的分析
/project-stage-detect designer
```

---

## 后续操作

生成报告后，推荐相关的后续步骤：

- **有概念但没有 systems index？** → `/map-systems` 分解为系统
- **缺少设计文档？** → `/reverse-document design src/[系统]`
- **缺少架构文档？** → `/architecture-decision` 或 `/reverse-document architecture`
- **原型需要文档化？** → `/reverse-document concept prototypes/[名称]`
- **没有 sprint 计划？** → `/sprint-plan`
- **接近里程碑？** → `/milestone-review`

---

## 协作协议

此 skill 遵循协作设计原则：

1. **先提问**: 对缺口提问，不要假设
2. **展示选项**: "我应该创建 X，还是在其他地方跟踪？"
3. **用户决定**: 等待指示
4. **展示草稿**: 显示报告摘要
5. **获取批准**: "我可以写入 production/project-stage-report.md 吗？"

**永远不要**静默写入文件。**始终**先展示发现并在创建资源前询问。
