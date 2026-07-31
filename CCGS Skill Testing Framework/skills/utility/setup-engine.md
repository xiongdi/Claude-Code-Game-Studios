# Skill Test Spec: /setup-engine

## Skill Summary

`/setup-engine` 通过填充 `technical-preferences.md` 来配置项目的引擎、语言、渲染后端、物理引擎、专家 agent 分配和命名约定。它接受可选的引擎参数（例如，`/setup-engine godot`）以跳过引擎选择步骤。对于 `technical-preferences.md` 的每个部分，该技能会展示一个草稿并在更新前询问 "May I write to `technical-preferences.md`?"。

该技能还根据选择的引擎填充专家路由表（文件扩展名 → agent 映射）。它没有 director gate——配置是一个技术工具任务。当文件完全写入后裁决始终为 COMPLETE。

---

## Static Assertions (Structural)

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：COMPLETE
- [ ] 在更新 technical-preferences.md 前包含 "May I write" 协作协议语言
- [ ] 具有下一步交接（例如，根据流程不同为 `/brainstorm` 或 `/start`）

---

## Director Gate Checks

无。`/setup-engine` 是一个技术配置技能。不适用 director gate。

---

## Test Cases

### Case 1: Godot 4 + GDScript — Full engine configuration

**Fixture:**
- `technical-preferences.md` 仅包含占位符
- 提供了引擎参数：`godot`

**Input:** `/setup-engine godot`

**Expected behavior:**
1. Skill 跳过引擎选择步骤（提供了参数）
2. Skill 展示 Godot 的语言选项：GDScript 或 C#
3. 用户选择 GDScript
4. Skill 起草所有引擎部分：引擎/语言/渲染/物理字段、
   命名约定（GDScript 使用 snake_case）、专家分配
   （godot-specialist、gdscript-specialist、godot-shader-specialist 等）
5. Skill 填充路由表：`.gd` → gdscript-specialist、`.gdshader` →
   godot-shader-specialist、`.tscn` → godot-specialist
6. Skill 询问 "May I write to `technical-preferences.md`?"
7. 批准后写入文件；裁决为 COMPLETE

**Assertions:**
- [ ] Engine 字段设置为 Godot 4（不是占位符）
- [ ] Language 字段设置为 GDScript
- [ ] 命名约定适合 GDScript（snake_case）
- [ ] 路由表包含 `.gd`、`.gdshader` 和 `.tscn` 条目
- [ ] 专家已分配（不是占位符）
- [ ] 在写入前询问 "May I write"
- [ ] 裁决为 COMPLETE

---

### Case 2: Unity + C# — Unity-specific configuration

**Fixture:**
- `technical-preferences.md` 仅包含占位符
- 提供了引擎参数：`unity`

**Input:** `/setup-engine unity`

**Expected behavior:**
1. Skill 设置引擎为 Unity，语言为 C#
2. 命名约定适合 C#（类使用 PascalCase，字段使用 camelCase）
3. 专家分配引用 unity-specialist、csharp-specialist
4. 路由表：`.cs` → csharp-specialist、`.asmdef` → unity-specialist、
   `.unity`（场景）→ unity-specialist
5. Skill 询问 "May I write to `technical-preferences.md`?" 并在批准后写入

**Assertions:**
- [ ] Engine 字段设置为 Unity（不是 Godot 或 Unreal）
- [ ] Language 字段设置为 C#
- [ ] 命名约定反映 C# 约定
- [ ] 路由表包含 `.cs` 和 `.unity` 条目
- [ ] 裁决为 COMPLETE

---

### Case 3: Unreal + Blueprint — Unreal-specific configuration

**Fixture:**
- `technical-preferences.md` 仅包含占位符
- 提供了引擎参数：`unreal`

**Input:** `/setup-engine unreal`

**Expected behavior:**
1. Skill 设置引擎为 Unreal Engine 5，主要语言为 Blueprint（Visual Scripting）
2. 专家分配引用 unreal-specialist、blueprint-specialist
3. 路由表：`.uasset` → blueprint-specialist 或 unreal-specialist、
   `.umap` → unreal-specialist
4. 性能预算预设为 Unreal 默认值（例如，更高的绘制调用预算）
5. Skill 询问 "May I write" 并在批准后写入；裁决为 COMPLETE

**Assertions:**
- [ ] Engine 字段设置为 Unreal Engine 5
- [ ] 路由表包含 `.uasset` 和 `.umap` 条目
- [ ] 分配了 Blueprint 专家
- [ ] 裁决为 COMPLETE

---

### Case 4: Engine Already Configured — Offers to reconfigure specific sections

**Fixture:**
- `technical-preferences.md` 的引擎设置为 Godot 4，所有字段已填充
- 未提供引擎参数

**Input:** `/setup-engine`

**Expected behavior:**
1. Skill 读取 `technical-preferences.md` 并检测到完全配置的引擎（Godot 4）
2. Skill 报告："Engine already configured as Godot 4 + GDScript"
3. Skill 展示选项：重新配置全部、仅重新配置特定部分
   （Engine/Language、Naming Conventions、Specialists、Performance Budgets）
4. 用户选择 "Reconfigure Performance Budgets only"
5. 仅更新性能预算部分；所有其他字段不变
6. Skill 询问 "May I write to `technical-preferences.md`?" 并在批准后写入

**Assertions:**
- [ ] 当仅请求部分更新时，Skill 不会覆盖所有字段
- [ ] 用户提供特定部分的重新配置
- [ ] 写入的文件中仅修改选定的部分
- [ ] 裁决为 COMPLETE

---

### Case 5: Director Gate Check — No gate; setup-engine is a utility skill

**Fixture:**
- 没有配置引擎的新项目

**Input:** `/setup-engine godot`

**Expected behavior:**
1. Skill 完成完整的引擎配置
2. 任何时候都不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 无需任何 gate 检查即可达到 COMPLETE

---

## Protocol Compliance

- [ ] 在请求写入前展示配置草稿
- [ ] 在写入前询问 "May I write to `technical-preferences.md`?"
- [ ] 提供时尊重引擎参数（跳过选择步骤）
- [ ] 检测现有配置并提供部分重新配置
- [ ] 为所选引擎的所有关键文件类型填充路由表
- [ ] 文件写入后裁决为 COMPLETE

---

## Coverage Notes

- Godot 4 + C#（而不是 GDScript）遵循与 Case 1 相同的流程，但使用不同的命名约定和 godot-csharp-specialist 分配。此变体未单独测试。
- 引擎版本特定的指导（例如，来自 VERSION.md 的 Godot 4.6 知识差距警告）由技能呈现，但在此不进行断言测试。
- 每个引擎的性能预算默认值被注明为引擎特定的，但确切的默认值不进行断言测试。
