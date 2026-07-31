---
name: ux-review
description: "Validates a UX spec, HUD design, or interaction pattern library for completeness, accessibility compliance, GDD alignment, and implementation readiness. Produces APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED verdict with specific gaps."
argument-hint: "[file-path or 'all' or 'hud' or 'patterns']"
user-invocable: true
allowed-tools: Read, Glob, Grep
model: sonnet
agent: ux-designer
---

## 概述

在 UX 设计文档进入实现流水线之前验证它们。
在 `/team-ui` 流水线中充当 UX 设计与视觉设计/实现之间的质量门控。

**运行此 skill 的时机：**
- 使用 `/ux-design` 完成 UX 规范后
- 在交接给 `ui-programmer` 或 `art-director` 之前
- 在 Pre-Production 到 Production 的 gate check 之前（这要求关键屏幕有经过审查的 UX 规范）
- UX 规范经过重大修订后

**结论级别：**
- **APPROVED** — 规范完整、一致且可立即实现
- **NEEDS REVISION** — 发现具体缺口；在交接前需要修复，但不是完全重新设计
- **MAJOR REVISION NEEDED** — 在范围、玩家需求或完整性方面存在根本性问题；需要大量返工

---

## 阶段 1：解析参数

- **具体文件路径**（例如 `/ux-review design/ux/inventory.md`）：验证该文档
- **`all`**：查找 `design/ux/` 中的所有文件并验证每个文件
- **`hud`**：专门验证 `design/ux/hud.md`
- **`patterns`**：专门验证 `design/ux/interaction-patterns.md`
- **无参数**：询问用户要验证哪个规范

对于 `all`，首先输出摘要表格（文件 | 结论 | 主要问题），然后为每个文件提供完整详情。

---

## 阶段 2：加载交叉引用上下文

在验证任何规范之前，加载：

1. **输入与平台配置**：读取 `.claude/docs/technical-preferences.md` 并提取 `## Input & Platform`。这是游戏支持哪些输入方式的权威来源 — 用它来驱动阶段 3A 中的输入方式覆盖检查，而不是规范自己的标题。如果未配置，回退到规范标题。
2. `design/accessibility-requirements.md` 中承诺的无障碍层级（如果存在）
3. `design/ux/interaction-patterns.md` 中的交互模式库（如果存在）
4. 规范标题中引用的 GDD（读取它们的 UI Requirements 部分）
5. `design/player-journey.md` 中的玩家旅程地图（如果存在），用于上下文到达验证

---

## 阶段 3A：UX 规范验证检查清单

针对基于 `ux-spec.md` 的文档运行所有检查。

### 完整性（必需部分）

- [ ] 文档标题存在，包含 Status、Author、Platform Target
- [ ] Purpose & Player Need — 有玩家视角的需求陈述（不是开发者视角）
- [ ] Player Context on Arrival — 描述玩家的状态和先前的活动
- [ ] Navigation Position — 显示屏幕在层次结构中的位置
- [ ] Entry & Exit Points — 记录了所有入口来源和出口目的地
- [ ] Layout Specification — 定义了区域，有组件清单表格
- [ ] States & Variants — 至少记录了 loading、empty/populated 和 error 状态
- [ ] Interaction Map — 覆盖所有目标输入方式（检查标题中的平台目标）
- [ ] Data Requirements — 每个显示的数据元素都有来源系统和所有者
- [ ] Events Fired — 每个玩家操作都有对应的事件或 null 解释
- [ ] Transitions & Animations — 至少指定了进入/退出过渡
- [ ] Accessibility Requirements — 存在屏幕级需求
- [ ] Localization Considerations — 文本元素的最大字符数
- [ ] Acceptance Criteria — 至少 5 个具体的可测试标准

### 质量检查

**玩家需求清晰度**
- [ ] 目的从玩家视角编写，不是系统/开发者视角
- [ ] 到达时的玩家目标明确（"The player arrives wanting to ___"）
- [ ] 到达时的玩家上下文具体（不仅仅是"他们打开了背包"）

**状态完整性**
- [ ] 记录了错误状态（不仅仅是 happy path）
- [ ] 记录了空状态（无数据场景）
- [ ] 如果屏幕获取异步数据，记录了加载状态
- [ ] 任何有计时器或自动关闭的状态都有持续时间记录

**输入方式覆盖**
- [ ] 如果平台包含 PC：完全指定了纯键盘导航
- [ ] 如果平台包含主机/手柄：记录了 d-pad 导航和 face button 映射
- [ ] 没有交互需要手柄上的鼠标式精确度
- [ ] 定义了焦点顺序（键盘的 Tab 顺序，手柄的 d-pad 顺序）

**数据架构**
- [ ] 没有数据元素将 "UI" 列为所有者（UI 不得拥有游戏状态）
- [ ] 所有实时数据都指定了更新频率（不仅仅是 "realtime" — 什么触发更新？）
- [ ] 所有数据元素都指定了 null 处理（数据不可用时显示什么？）

**无障碍**
- [ ] 匹配或超过了 `accessibility-requirements.md` 中的无障碍层级
- [ ] 如果是 Basic 层级：没有仅颜色信息指示器
- [ ] 如果是 Standard 层级+：记录了焦点顺序，指定了文本对比度比率
- [ ] 如果是 Comprehensive 层级+：关键状态变更的屏幕阅读器通知
- [ ] 色盲检查：任何颜色编码的元素都有非颜色替代方案

**GDD 一致性**
- [ ] 标题中引用的每个 GDD UI Requirement 在此规范中都有对应
- [ ] 没有 UI 元素在没有对应 GDD 需求的情况下显示或修改游戏状态
- [ ] 此规范没有遗漏任何 GDD UI Requirement（交叉检查引用的 GDD 部分）

**模式库一致性**
- [ ] 所有可交互组件都引用了模式库（或注明它们是新模式）
- [ ] 如果模式库中已存在，没有从头重新指定任何模式行为
- [ ] 此规范中发明的任何新模式都标记为添加到模式库

**本地化**
- [ ] 所有文本密集的元素都有字符限制警告
- [ ] 任何布局关键的文本已标记为容纳 40% 扩展

**验收标准质量**
- [ ] 标准足够具体，供未见过设计文档的 QA 测试人员使用
- [ ] 存在性能标准（屏幕在 Xms 内打开）
- [ ] 存在分辨率标准
- [ ] 没有标准需要阅读另一个文档才能评估

---

## 阶段 3B：HUD 验证检查清单

针对基于 `hud-design.md` 的文档运行所有检查。

### 完整性

- [ ] 定义了 HUD Philosophy
- [ ] 信息架构表格覆盖了 GDD 中具有 UI Requirements 的所有系统
- [ ] 为所有目标平台定义了带安全区域边距的 Layout Zones
- [ ] 每个 HUD 元素都有完整规范（区域、可见性触发器、数据源、优先级）
- [ ] HUD States by Gameplay Context 至少覆盖：探索、战斗、对话/过场、暂停
- [ ] 定义了 Visual Budget（最大同时元素数、最大屏幕百分比）
- [ ] Platform Adaptation 覆盖所有目标平台
- [ ] 玩家可调整元素的 Tuning Knobs 存在

### 质量检查

- [ ] 没有 HUD 元素覆盖中心游戏区域而没有隐藏它的可见性规则
- [ ] 任何 GDD 中存在的每个信息项目要么在 HUD 中，要么明确分类为 "hidden/demand"
- [ ] 所有颜色编码的 HUD 元素都有色盲变体
- [ ] Feedback & Notification 部分的 HUD 元素定义了队列/优先级行为
- [ ] Visual Budget 合规：总同时元素数在预算内

### GDD 一致性

- [ ] `design/gdd/systems-index.md` 中所有具有 UI 类别的系统在 HUD 中都有表示（或有理由的缺席）

---

## 阶段 3C：模式库验证检查清单

- [ ] 模式目录索引是最新的（与文档中的实际模式匹配）
- [ ] 所有标准控件模式都已指定：button variants、toggle、slider、dropdown、list、grid、modal、dialog、toast、tooltip、progress bar、input field、tab bar、scroll
- [ ] 当前 UX 规范所需的所有游戏特定模式都存在
- [ ] 每个模式都有：When to Use、When NOT to Use、完整状态规范、无障碍规范、实现说明
- [ ] 存在 Animation Standards 表格
- [ ] 存在 Sound Standards 表格
- [ ] 模式之间没有冲突行为（例如，"Back" 行为在所有导航模式中一致）

---

## 阶段 4：输出结论

```markdown
## UX Review: [Document Name]
**Date**: [date]
**Reviewer**: ux-review skill
**Document**: [file path]
**Platform Target**: [from header]
**Accessibility Tier**: [from header or accessibility-requirements.md]

### Completeness: [X/Y sections present]
- [x] Purpose & Player Need
- [ ] States & Variants — MISSING: error state not documented

### Quality Issues: [N found]
1. **[Issue title]** [BLOCKING / ADVISORY]
   - What's wrong: [specific description]
   - Where: [section name]
   - Fix: [specific action to take]

### GDD Alignment: [ALIGNED / GAPS FOUND]
- GDD [name] UI Requirements — [X/Y requirements covered]
- Missing: [list any uncovered GDD requirements]

### Accessibility: [COMPLIANT / GAPS / NON-COMPLIANT]
- Target tier: [tier]
- [list specific accessibility findings]

### Pattern Library: [CONSISTENT / INCONSISTENCIES FOUND]
- [findings]

### Verdict: APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED
**Blocking issues**: [N] — must be resolved before implementation
**Advisory issues**: [N] — recommended but not blocking

[For APPROVED]: This spec is ready for handoff to `/team-ui` Phase 2
(Visual Design).

[For NEEDS REVISION]: Address the [N] blocking issues above, then re-run
`/ux-review`.

[For MAJOR REVISION NEEDED]: The spec has fundamental gaps in [areas].
Recommend returning to `/ux-design` to rework [sections].
```

---

## 阶段 5：协作协议

此 skill 是只读的 — 它从不编辑或写入文件。它仅报告发现。

交付结论后：
- 对于 **APPROVED**：建议运行 `/team-ui` 开始实现协调
- 对于 **NEEDS REVISION**：提供修复具体缺口的帮助（"Would you like me to help draft the missing error state?"）— 但不要自动修复；等待用户指示
- 对于 **MAJOR REVISION NEEDED**：建议返回 `/ux-design` 并指定要返工的具体部分

永远不要阻止用户继续 — 结论是建议性的。记录风险，呈现发现，让用户决定是否在有顾虑的情况下继续。选择继续使用 NEEDS REVISION 规范的用户承担记录的风险。
