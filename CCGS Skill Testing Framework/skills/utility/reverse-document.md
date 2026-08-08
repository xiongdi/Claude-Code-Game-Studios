# 技能测试规格: /reverse-document

## 技能摘要

`/reverse-document` 从现有源代码生成设计或架构文档。它读取指定的源文件，从类结构、方法名、常量和注释中推断设计意图，并生成 GDD 骨架（针对游戏系统）或架构概述（针对技术系统）。输出是尽力推断——魔法数字和未记录的逻辑可能导致 PARTIAL 裁决。

该技能在创建文档前会询问 "May I write to [inferred path]?"。不适用 director gate。裁决：COMPLETE（清晰推断）、PARTIAL（某些字段模糊，需要人工审查）。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：COMPLETE、PARTIAL
- [ ] 在写入文档前包含 "May I write" 协作协议语言
- [ ] 具有下一步交接（例如，`/design-review` 以验证生成的文档）

---

## 导演门控检查

无。`/reverse-document` 是一个文档工具。不适用 director gate。

---

## 测试用例

### Case 1: Well-Structured Source — Accurate design doc skeleton produced

**Fixture:**
- `src/gameplay/health_system.gd` 存在并包含：
  - `@export var max_health: int = 100`
  - 带 clamping 逻辑的 `func take_damage(amount: int)`
  - `signal health_changed(new_value: int)`
  - 所有公共方法都有文档字符串

**Input:** `/reverse-document src/gameplay/health_system.gd`

**Expected behavior:**
1. Skill 读取源文件并识别生命值系统
2. Skill 推断设计意图：最大生命值、take_damage 行为、生命值信号
3. Skill 生成生命值系统的 GDD 骨架，包含 8 个必需部分：
   Overview、Player Fantasy、Detailed Rules、Formulas、Edge Cases、Dependencies、
   Tuning Knobs、Acceptance Criteria
4. Formulas 部分包含推断的 clamping 公式
5. Tuning Knobs 注明 `max_health = 100` 为可配置值
6. Skill 询问 "May I write to `design/gdd/health-system.md`?"
7. 文件已写入；裁决为 COMPLETE

**Assertions:**
- [ ] 输出中存在所有 8 个必需的 GDD 部分
- [ ] `max_health = 100` 作为 Tuning Knob 出现
- [ ] clamping 公式在 Formulas 部分中被捕获
- [ ] "May I write" 使用推断的路径
- [ ] 裁决为 COMPLETE

---

### Case 2: Ambiguous Source — Magic Numbers, PARTIAL Verdict

**Fixture:**
- `src/gameplay/enemy_ai.gd` 存在并包含：
  - 内联魔法数字：`if distance < 150:`, `speed = 3.5`
  - 无注释或文档字符串
  - 不自我解释的复杂状态机逻辑

**Input:** `/reverse-document src/gameplay/enemy_ai.gd`

**Expected behavior:**
1. Skill 读取文件并检测到没有上下文的魔法数字
2. Skill 生成带有注释的 GDD 骨架："AMBIGUOUS VALUE: 150 (unknown units —
   is this pixels, world units, or tiles?)"
3. Skill 将 Formulas 和 Tuning Knobs 部分标记为需要人工审查
4. Skill 询问 "May I write to `design/gdd/enemy-ai.md`?" 并附带 PARTIAL 建议
5. 文件写入并带有 PARTIAL 标记；裁决为 PARTIAL

**Assertions:**
- [ ] 魔法数字出现 AMBIGUOUS VALUE 注释
- [ ] 需要人工审查的部分被明确标记
- [ ] 裁决为 PARTIAL（不是 COMPLETE）
- [ ] 文件仍被写入——PARTIAL 不是阻塞失败

---

### Case 3: Multiple Interdependent Files — Cross-System Overview Produced

**Fixture:**
- 用户提供 2 个源文件：`combat_system.gd` 和 `damage_resolver.gd`
- 文件相互引用（combat 调用 damage_resolver）

**Input:** `/reverse-document src/gameplay/combat_system.gd src/gameplay/damage_resolver.gd`

**Expected behavior:**
1. Skill 读取两个文件并检测到依赖关系
2. Skill 生成跨系统架构概述（不是单独的 GDD）
3. 概述描述：Combat System → Damage Resolver 交互、共享接口、两者之间的数据流
4. Skill 询问 "May I write to `docs/architecture/combat-damage-overview.md`?"
5. 批准后写入概述；裁决为 COMPLETE（或如果模糊则为 PARTIAL）

**Assertions:**
- [ ] 两个文件被一起分析（不是作为两个单独的文档）
- [ ] 输出中记录了跨系统依赖
- [ ] 输出文件写入 `docs/architecture/`（不是 `design/gdd/`）
- [ ] 裁决为 COMPLETE 或 PARTIAL

---

### Case 4: Source File Not Found — Error

**Fixture:**
- `src/gameplay/inventory_system.gd` 不存在

**Input:** `/reverse-document src/gameplay/inventory_system.gd`

**Expected behavior:**
1. Skill 尝试读取指定的文件——未找到
2. Skill 输出："Source file not found: src/gameplay/inventory_system.gd"
3. Skill 建议检查路径或运行 `/map-systems` 以识别正确的源文件
4. 不创建文档

**Assertions:**
- [ ] 错误消息使用完整路径命名缺失的文件
- [ ] 提供了替代建议（检查路径或 `/map-systems`）
- [ ] 不调用写入工具
- [ ] 不发布裁决（错误状态）

---

### Case 5: Director Gate Check — No gate; reverse-document is a utility

**Fixture:**
- 存在结构良好的源文件

**Input:** `/reverse-document src/gameplay/health_system.gd`

**Expected behavior:**
1. Skill 生成并写入设计文档
2. 不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 裁决为 COMPLETE 或 PARTIAL——不涉及 gate 裁决

---

## 协议合规

- [ ] 在生成任何内容前读取源文件
- [ ] 当目标是游戏系统时生成所有 8 个必需的 GDD 部分
- [ ] 用 AMBIGUOUS VALUE 标记注释模糊值
- [ ] 对多个文件生成跨系统概述（不是单独的 GDD）
- [ ] 在创建任何输出文件前询问 "May I write"
- [ ] 裁决为 COMPLETE（清晰推断）或 PARTIAL（模糊字段）

---

## 覆盖说明

- 架构概述格式（针对技术/基础设施系统）与 GDD 格式不同；推断的输出类型由源文件的性质决定（游戏逻辑 → GDD；引擎/基础设施代码 → 架构文档）。
- 源文件可读但仅包含无有意义逻辑的自动生成样板代码的情况未测试；skill 可能会生成带有 PARTIAL 裁决的近乎空的骨架。
- C# 和 Blueprint 源文件遵循与 GDScript 相同的推断模式；语言特定的差异在技能正文中处理。
