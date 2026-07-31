---
name: balance-check
description: "Analyzes game balance data files, formulas, and configuration to identify outliers, broken progressions, degenerate strategies, and economy imbalances. Use after modifying any balance-related data or design. Use when user says 'balance report', 'check game balance', 'run a balance check'."
argument-hint: "[system-name|path-to-data-file]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion
model: sonnet
agent: economy-designer
---

## 阶段 1：识别平衡域

从 `$ARGUMENTS[0]` 确定平衡域：

- **Combat** → 武器/能力 DPS、击杀时间、伤害类型交互
- **Economy** → 资源 faucet/sink、获取率、物品定价
- **Progression** → XP/力量曲线、死区、力量峰值
- **Loot** → 稀有度分布、保底计时器、库存压力
- **给定文件路径** → 直接加载该文件并从内容推断域

如果没有参数，询问用户要检查哪个系统。

---

## 阶段 2：读取数据文件

从 `assets/data/` 和 `design/balance/` 读取识别到的域的相关文件。
记录读取的每个文件 —— 它们将出现在报告的 Data Sources 部分。

---

## 阶段 3：读取设计文档

从 `design/gdd/` 读取系统的 GDD 以了解预期的设计目标、
tuning knobs 和预期值范围。这是"正确"行为的基线。

---

## 阶段 4：执行分析

运行域特定的检查：

**战斗平衡：**
- 计算每个能力层级下所有武器/能力的 DPS
- 检查每个层级的击杀时间
- 识别任何支配所有其他选项的选项（严格更好）
- 检查防御选项是否会产生无法击杀的状态
- 验证伤害类型/抗性交互是否平衡

**经济平衡：**
- 映射所有资源 faucet/sink 及其流速
- 预测随时间推移的资源积累
- 检查无限资源循环
- 验证金币 sink 是否随金币生成扩展
- 检查是否有任何物品永远不值得购买

**进程平衡：**
- 绘制 XP 曲线和力量曲线
- 检查死区（没有有意义进程的时间过长）
- 检查力量峰值（能力的突然跳跃）
- 验证内容门控是否与玩家预期力量对齐
- 检查跳过/刷取策略是否破坏预期节奏

**战利品平衡：**
- 计算获取每个稀有度层级的预期时间
- 检查保底计时器数学
- 验证没有任何战利品在任何阶段严格无用
- 检查库存压力与获取率

---

## 阶段 5：输出分析

```
## Balance Check: [System Name]

### 分析的数据源
- [读取的文件列表]

### 健康摘要: [HEALTHY / CONCERNS / CRITICAL ISSUES]

### 检测到的异常值
| 项目/值 | 预期范围 | 实际值 | 问题 |
|-----------|---------------|--------|-------|

### 发现的退化策略
- [策略描述及其为何有问题]

### 进程分析
[显示进程曲线健康状况的图表描述或表格]

### 建议
| 优先级 | 问题 | 建议修复 | 影响 |
|----------|-------|--------------|--------|

### 需要关注的值
[带有建议调整理由的具体值]
```

---

## 阶段 6：修复与验证循环

展示报告后，使用 `AskUserQuestion`：
- 提示："平衡检查完成。你希望下一步做什么？"
- 选项：
  - `[A] 现在修复最高优先级问题 —— 带我过一遍`
  - `[B] 将报告保存到 design/balance/balance-check-[system]-[date].md`
  - `[C] 在这里停止 —— 我会手动审查发现`

如果 [A]：
- 询问首先解决哪个问题（按优先级行参考建议表）
- 引导用户更新 `assets/data/` 中的相关数据文件或 `design/balance/` 中的公式
- 每次修复后，提供重新运行相关平衡检查以验证没有引入新的异常值
- 如果修复更改了 GDD 中定义的 tuning knob 或被 ADR 引用，提醒用户：
  > "此值在设计文档中定义。在提交前对受影响的 GDD 运行 `/propagate-design-change [path]` 以查找下游影响。"

如果 [B]：
- 将报告写入 `design/balance/balance-check-[system]-[date].md`（如果需要创建目录）。对 [date] 使用 YYYY-MM-DD 格式的当前日期。
- 确认文件已写入，然后以："修复后重新运行 `/balance-check` 进行验证。"结束

如果 [C]：
- 总结未解决的问题并以："修复后重新运行 `/balance-check` 进行验证。"结束
