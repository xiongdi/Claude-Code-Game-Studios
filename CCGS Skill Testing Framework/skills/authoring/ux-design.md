# Skill Test Spec: /ux-design

## Skill Summary

`/ux-design` 是一个引导式逐章节 UX 规格编写 skill。它为指定屏幕或 HUD 元素生成用户流程图表（文本描述）、交互状态定义、线框图描述和可访问性说明。该 skill 遵循 skeleton-first 模式：立即创建包含所有章节标题的文件，然后通过讨论填写每个章节，并在用户批准后将每个章节写入磁盘。

该 skill 没有内联 director gate——`/ux-review` 是单独的审查步骤。每个章节都需要 "May I write section [N] to [filepath]?" 请求。如果指定屏幕已存在 UX 规格，该 skill 提供改造单个章节的选项而非替换。当所有章节写入后裁定为 COMPLETE。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：COMPLETE
- [ ] 包含逐章节的 "May I write" 语言
- [ ] 具有下一步交接说明（例如，`/ux-review` 验证完成的规格）

---

## Director Gate Checks

无。`/ux-design` 没有内联 director gate。`/ux-review` 是在此 skill 完成后调用的单独审查 skill。

---

## Test Cases

### Case 1: Happy Path——新 HUD 规格，所有章节编写并写入

**Fixture:**
- `design/ux/` 中无现有 HUD UX 规格
- 引擎和渲染偏好已配置

**Input:** `/ux-design hud`

**Expected behavior:**
1. Skill 创建骨架文件 `design/ux/hud.md`，包含所有章节标题
2. Skill 讨论并起草每个章节：User Flows、Interaction States
   （normal/hover/focus/disabled）、Wireframe Description、Accessibility Notes
3. 每个章节起草并用户确认后，skill 询问 "May I write
   section [N] to `design/ux/hud.md`?"
4. 每个章节在批准后按顺序写入
5. 所有章节写入后，裁定为 COMPLETE
6. Skill 建议下一步运行 `/ux-review`

**Assertions:**
- [ ] 骨架文件首先创建（章节正文为空）
- [ ] 逐章节询问 "May I write section [N]"（非仅在末尾一次）
- [ ] 所有必需章节存在：User Flows、Interaction States、
     Wireframe Description、Accessibility Notes
- [ ] 末尾有到 `/ux-review` 的交接说明
- [ ] 裁定为 COMPLETE

---

### Case 2: Existing UX Spec——改造：用户选择要更新的章节

**Fixture:**
- `design/ux/hud.md` 已存在，所有章节已填写
- 用户只想更新 Accessibility Notes 章节

**Input:** `/ux-design hud`

**Expected behavior:**
1. Skill 读取现有 `design/ux/hud.md` 并检测所有章节已填写
2. Skill 报告："UX spec already exists for HUD — offering to retrofit"
3. Skill 列出所有章节并询问要更新哪个
4. 用户选择 Accessibility Notes
5. Skill 起草更新的可访问性内容并询问 "May I write section
   Accessibility Notes to `design/ux/hud.md`?"
6. 仅该章节更新；其他章节保留；裁定为 COMPLETE

**Assertions:**
- [ ] 检测现有规格并提供改造选项
- [ ] 用户选择要更新哪个章节
- [ ] 仅所选章节更新——其他章节不变
- [ ] 对更新的章节询问 "May I write"
- [ ] 裁定为 COMPLETE

---

### Case 3: Dependency Gap——规格引用了无设计文档的系统

**Fixture:**
- 用户正在为库存屏幕编写 UX 规格
- `design/gdd/inventory.md` 不存在

**Input:** `/ux-design inventory-screen`

**Expected behavior:**
1. Skill 开始编写库存屏幕 UX 规格
2. 在 User Flows 章节期间，skill 尝试引用库存系统规则
3. Skill 检测："No GDD found for inventory system — UX spec has a DEPENDENCY GAP"
4. 依赖差距在规格中标记（内联注明："DEPENDENCY GAP: inventory GDD"）
5. Skill 继续编写，为缺失规则使用占位符说明
6. 裁定为 COMPLETE，附带关于依赖差距的建议说明

**Assertions:**
- [ ] DEPENDENCY GAP 标签出现在缺失系统文档的规格中
- [ ] Skill 不因缺失 GDD 而阻塞——继续使用占位符
- [ ] 依赖差距也在 skill 输出中注明（不仅在文件中）
- [ ] 交接说明建议同时运行 `/ux-review` 和编写缺失的 GDD

---

### Case 4: No Argument Provided——用法错误

**Fixture:**
- 调用 skill 时未提供参数

**Input:** `/ux-design`

**Expected behavior:**
1. Skill 检测未提供屏幕名称或参数
2. Skill 输出用法错误："Screen name required. Usage: `/ux-design [screen-name]`"
3. Skill 提供示例：`/ux-design hud`、`/ux-design main-menu`、`/ux-design inventory`
4. 不创建文件；不询问 "May I write"

**Assertions:**
- [ ] 用法错误清晰陈述
- [ ] 提供调用示例
- [ ] 不创建文件
- [ ] Skill 不尝试在没有参数的情况下继续

---

### Case 5: Director Gate Check——无 gate；ux-review 是单独的审查 skill

**Fixture:**
- 提供了参数的新屏幕规格

**Input:** `/ux-design settings-menu`

**Expected behavior:**
1. Skill 编写设置菜单 UX 规格的所有章节
2. 不派生 director agent
3. 编写过程中输出中不出现 gate ID

**Assertions:**
- [ ] 在 ux-design 期间不调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 裁定为 COMPLETE，无需任何 gate 检查

---

## Protocol Compliance

- [ ] 在讨论内容前创建包含所有章节标题的骨架文件
- [ ] 一次讨论和起草一个章节
- [ ] 每个章节批准后询问 "May I write section [N]"
- [ ] 检测现有规格并提供改造路径
- [ ] 以到 `/ux-review` 的交接说明结束
- [ ] 当所有章节写入后裁定为 COMPLETE

---

## Coverage Notes

- 交互状态枚举（normal/hover/focus/disabled/error）是每个规格的核心要求；
  `/ux-review` skill 检查完整性。
- 线框图描述是纯文本（无图像）；图像引用可由设计师事后手动添加。
- 响应式布局问题（不同屏幕尺寸）被注明为可选内容，不在此进行断言测试。
