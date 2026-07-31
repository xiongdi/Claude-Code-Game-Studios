---
name: skill-improve
description: "Improve a skill using a test-fix-retest loop. Runs static checks, proposes targeted fixes, rewrites the skill, re-tests, and keeps or reverts based on score change."
argument-hint: "[skill-name]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Bash
model: sonnet
---

# Skill Improve

在单个 skill 上运行改进循环：
测试 → 修复 → 重新测试 → 保留或还原。

---

## Phase 1: 解析参数

从第一个参数读取 skill 名称。如果缺失，输出用法并停止：

```
用法: /skill-improve [skill-name]
示例: /skill-improve tech-debt
```

验证 `.claude/skills/[名称]/SKILL.md` 是否存在。如果不存在，停止并显示：
"未找到 Skill '[名称]'。"

---

## Phase 2: 基线测试

运行 `/skill-test static [名称]` 并记录基线分数：
- FAIL 数量
- WARN 数量
- 哪些具体检查失败（检查 1-7）

向用户展示：
```
静态基线:   [N] 个失败，[M] 个警告
失败项: 检查 4（无 ask-before-write），检查 5（无 handoff）
```

如果基线为 0 个 FAIL 和 0 个 WARN，记录并继续到 Phase 2b。

### Phase 2b: 类别基线

在 `CCGS Skill Testing Framework/catalog.yaml` 中查找 skill 的 `category:` 字段。

如果未找到 `category:` 字段，显示：
"类别：尚未分配 — 跳过类别检查。"
并跳到 Phase 3。

如果找到类别，运行 `/skill-test category [名称]` 并记录类别基线：
- FAIL 数量
- WARN 数量
- 哪些具体类别评分标准失败

向用户展示：
```
类别基线: [N] 个失败，[M] 个警告（[类别] 评分标准）
```

如果静态和类别基线都为 0 个 FAIL 和 0 个 WARN，停止：
"此 skill 已通过所有静态和类别检查。无需改进。"

---

## Phase 3: 诊断

读取 `.claude/skills/[名称]/SKILL.md` 的完整 skill 文件。

对于每个失败或警告的**静态**检查，识别确切的缺口：

- **检查 1 失败** → 缺少哪个 frontmatter 字段
- **检查 2 失败** → 找到多少个 phase vs 最低要求
- **检查 3 失败** → skill 主体中无判定关键词
- **检查 4 失败** → allowed-tools 中有 Write 或 Edit 但无 ask-before-write 语言
- **检查 5 警告** → 末尾无跟进或下一步部分
- **检查 6 警告** → 设置了 `context: fork` 但找到的 phase 少于 5 个
- **检查 7 警告** → argument-hint 为空或与记录的模式不匹配

对于每个失败或警告的**类别**检查（如果在 Phase 2b 中分配了类别），
识别 skill 文本中的确切缺口。例如：
- 如果 G2 失败（gate 模式，未 spawn 完整的 director）：skill 主体从未引用所有 4 个
  PHASE-GATE director 提示
- 如果 A2 失败（编写，无逐节 May-I-write）：skill 在末尾问一次，而不是
  在每个部分写入前问
- 如果 T3 失败（团队，未暴露 BLOCKED）：skill 在被阻塞的 agent 上不停止依赖工作

在进行任何更改之前，向用户展示完整的组合诊断。

---

## Phase 4: 提议修复

为每个失败和警告编写有针对性的修复。将提议的更改清楚地标记为 before/after 块。仅更改失败的内容 — 不要重写正在通过的部分。

询问："我可以将此改进版本写入 `.claude/skills/[名称]/SKILL.md` 吗？"

如果用户不同意，在此停止。

---

## Phase 5: 写入并重新测试

记录 skill 文件的当前内容（以便需要时还原）。

将改进后的 skill 写入 `.claude/skills/[名称]/SKILL.md`。

重新运行 `/skill-test static [名称]` 并记录新的静态分数。
如果分配了类别，还要重新运行 `/skill-test category [名称]` 并记录新的类别分数。

展示比较：
```
静态:   之前 [N] 个失败，[M] 个警告  →  之后 [N'] 个失败，[M'] 个警告
类别: 之前 [N] 个失败，[M] 个警告  →  之后 [N'] 个失败，[M'] 个警告（如适用）
组合变化: 改善 / 无变化 / 更差
```

---

## Phase 6: 判定

计算组合失败总数：静态 FAIL + 类别 FAIL + 静态 WARN + 类别 WARN。

**如果组合分数改善（组合失败计数低于基线）：**
报告："分数已改善。更改已保留。"
展示每个维度修复内容的摘要。

**如果组合分数相同或更差：**
报告："组合分数未改善。"
展示更改了什么以及为什么可能没有帮助。
询问："我可以使用 git checkout 还原 `.claude/skills/[名称]/SKILL.md` 吗？"
如果同意：运行 `git checkout -- .claude/skills/[名称]/SKILL.md`

---

## Phase 7: 后续步骤

- 运行 `/skill-test static all` 查找下一个有失败的 skill。
- 运行 `/skill-improve [下一个名称]` 继续在另一个 skill 上循环。
- 运行 `/skill-test audit` 查看整体覆盖进度。
