# 技能测试规格: /ux-review

## 技能摘要

`/ux-review` 根据可访问性和交互标准验证现有 UX 规格或 HUD 设计文档。它检查必需章节（User Flows、Interaction States、Wireframe Description、Accessibility Notes）、交互状态定义的完整性（hover、focus、disabled、error）、可访问性合规性（键盘导航、颜色对比度说明、屏幕阅读器考虑），以及如果这些文档存在时与 art bible 或 design system 的一致性。

该 skill 是只读的——不产生文件写入。裁定：APPROVED（所有检查通过）、NEEDS REVISION（发现可修复问题）或 MAJOR REVISION NEEDED（结构性或可访问性失败）。不适用 director gate——`/ux-review` 本身就是 UX 规格的审查 gate。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：APPROVED、NEEDS REVISION、MAJOR REVISION NEEDED
- [ ] 不包含 "May I write" 语言（skill 是只读的）
- [ ] 具有下一步交接说明（例如，返回 `/ux-design` 进行修改，或继续实现）

---

## 导演门控检查

无。`/ux-review` 本身就是 UX 规格的审查 gate。在此 skill 内不调用其他 director gate。

---

## 测试用例

### Case 1: Happy Path——具有所有必需章节的完整 UX 规格，APPROVED

**Fixture:**
- `design/ux/hud.md` 存在，所有必需章节已填写：
  - User Flows：完整的玩家流程图
  - Interaction States：normal、hover、focus、disabled、error 全部定义
  - Wireframe Description：布局已描述
  - Accessibility Notes：键盘导航、对比度比率、屏幕阅读器说明

**Input:** `/ux-review hud`

**Expected behavior:**
1. Skill 读取 `design/ux/hud.md`
2. Skill 检查所有 4 个必需章节——全部存在且非空
3. Skill 检查交互状态——全部 5 种状态已定义
4. Skill 检查可访问性说明——键盘、对比度和屏幕阅读器已覆盖
5. Skill 输出：所有通过检查的清单
6. 裁定为 APPROVED

**Assertions:**
- [ ] 所有 4 个必需章节都被检查
- [ ] 所有 5 种交互状态被验证存在
- [ ] 裁定为 APPROVED
- [ ] 不写入任何文件

---

### Case 2: Missing Accessibility Section——NEEDS REVISION

**Fixture:**
- `design/ux/hud.md` 存在但 Accessibility Notes 章节为空
- 其他所有章节完全填写

**Input:** `/ux-review hud`

**Expected behavior:**
1. Skill 读取文件并检查所有章节
2. Accessibility Notes 章节为空——检查失败
3. Skill 输出："NEEDS REVISION — Accessibility Notes section is empty"
4. Skill 列出要添加的具体项目：键盘导航、颜色对比度比率、
   屏幕阅读器标签
5. 裁定为 NEEDS REVISION
6. 交接说明建议返回 `/ux-design hud` 填写该章节

**Assertions:**
- [ ] 返回 NEEDS REVISION 裁定（不是 APPROVED 或 MAJOR REVISION NEEDED）
- [ ] 列出具体缺失的内容项目
- [ ] 交接说明指向 `/ux-design hud` 进行修改
- [ ] 不写入任何文件

---

### Case 3: Interaction States Incomplete——NEEDS REVISION

**Fixture:**
- `design/ux/settings-menu.md` 存在
- Interaction States 章节仅定义：normal 和 hover
- 缺失：focus、disabled、error 状态

**Input:** `/ux-review settings-menu`

**Expected behavior:**
1. Skill 读取文件并检查交互状态
2. 仅定义了 5 种必需状态中的 2 种
3. Skill 报告："NEEDS REVISION — Interaction states incomplete: missing focus, disabled, error"
4. 裁定为 NEEDS REVISION，明确命名具体缺失的状态

**Assertions:**
- [ ] 返回 NEEDS REVISION 裁定
- [ ] 所有 3 种缺失状态在输出中明确命名
- [ ] Skill 对可修复的差距不返回 MAJOR REVISION NEEDED
- [ ] 交接说明建议返回 `/ux-design settings-menu`

---

### Case 4: File Not Found——带有修复指导的错误

**Fixture:**
- `design/ux/inventory-screen.md` 不存在

**Input:** `/ux-review inventory-screen`

**Expected behavior:**
1. Skill 尝试读取 `design/ux/inventory-screen.md`——文件未找到
2. Skill 输出："UX spec not found: design/ux/inventory-screen.md"
3. Skill 建议运行 `/ux-design inventory-screen` 先创建规格
4. 不执行审查；不发布裁定

**Assertions:**
- [ ] 错误消息以完整路径命名缺失的文件
- [ ] `/ux-design inventory-screen` 被建议为修复步骤
- [ ] 不产生审查清单
- [ ] 不发布裁定（错误状态，不是 APPROVED/NEEDS REVISION）

---

### Case 5: Director Gate Check——无 gate；ux-review 本身就是审查

**Fixture:**
- 有效的 UX 规格文件

**Input:** `/ux-review hud`

**Expected behavior:**
1. Skill 执行审查并发布裁定
2. 不派生其他 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 不调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 裁定为 APPROVED、NEEDS REVISION 或 MAJOR REVISION NEEDED——无 gate 裁定

---

## 协议合规

- [ ] 检查所有 4 个必需章节（User Flows、Interaction States、Wireframe、
     Accessibility Notes）
- [ ] 检查所有 5 种交互状态（normal、hover、focus、disabled、error）
- [ ] 检查可访问性覆盖（键盘导航、对比度、屏幕阅读器）
- [ ] 不写入任何文件
- [ ] 当裁定不是 APPROVED 时发布具体、可操作的反馈
- [ ] 以到 `/ux-design` 进行修改或实现的下一步交接说明结束

---

## 覆盖说明

- MAJOR REVISION NEEDED 在结构性章节完全缺失（不仅仅是空）或基本交互流程完全缺失时触发；
  不在此使用单独的 fixture 测试。
- Art bible / design system 一致性检查（颜色调色板对齐）作为能力提及但不单独进行 fixture 测试。
- 现有规格是为现已重命名的屏幕编写的情况不在此测试；
  skill 会按路径审查文件，无论名称如何。
