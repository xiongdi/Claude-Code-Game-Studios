# 示例会话：完整 Story 生命周期 — /story-readiness → 实施 → /story-done

**背景：** 独立开发者，生产 sprint 中期。移动系统 GDD 和架构已完成。一个用于移动核心实现的 story 已由 `/create-stories` 生成。

**起始状态：**
- `design/gdd/movement-system.md` — 完整，8个章节
- `docs/architecture/movement-adr-001.md` — 状态：已批准
- `production/sprints/sprint-03.md` — backlog 中列出 STORY-MOV-001
- `production/stories/STORY-MOV-001.md` — 存在但尚未验证
- 尚无实施代码

**会话时长：** 约50分钟
**轮次：** 13
**结果：** Story 验证就绪，已实施，用 `/story-done` 验证完成

---

## 阶段1：Story 就绪检查

### 第1轮：用户运行就绪检查

**用户：**
> /story-readiness production/stories/STORY-MOV-001.md

### 第2轮：Agent 验证 Story

**Agent（通过 /story-readiness）：**
> 读取 `production/stories/STORY-MOV-001.md`...
>
> **STORY-MOV-001: 实现 CharacterBody2D 移动与地形修饰符**
>
> 运行4项验证检查：
>
> **1. 设计完整性**
> - 引用的 GDD：`design/gdd/movement-system.md` ✅
> - GDD 章节覆盖：详细规则 ✅、公式 ✅、边缘情况 ✅
> - 嵌入的 TR-ID：TR-MOV-001 到 TR-MOV-008 ✅
>
> **2. 架构完整性**
> - 引用的 ADR：`docs/architecture/movement-adr-001.md` ✅
> - ADR 状态：**已批准** ✅ *（如果是 Proposed 则会被 BLOCKED）*
> - Story 中的控制清单版本：`2026-03-10` — 当前清单是 `2026-03-10` ✅
>
> **3. 范围清晰度**
> - 验收标准：9项，全部可测量 ✅
> - 范围外项目列出：游泳、攀爬 ✅
> - 发现歧义：**1**
>   - ⚠️ Story 说"翻滚方向跟随最后输入方向"但 GDD 说"向移动方向翻滚"——当玩家停止并立即翻滚时冲突。实施前澄清。
>
> **4. 完成定义**
> - 需要测试：公式函数的单元测试 ✅
> - 集成信号：`on_stamina_event` 必须正确触发 ✅
>
> ---
> **判定：需要工作**
>
> 实施开始前必须解决一个歧义。你想现在解决它以便我将 story 标记为就绪吗？

### 第3轮：用户解决歧义

**用户：**
> 翻滚使用最后移动方向。如果玩家站着不动，翻滚使用面向方向代替。

### 第4轮：Agent 更新 Story

**Agent：**
> 更新 story 的详细规则部分：
>
> > 翻滚方向：最后移动方向。如果翻滚输入时速度为零，翻滚使用当前面向方向。
>
> 可以将此写入 `production/stories/STORY-MOV-001.md` 并将状态设置为 `ready-for-dev` 吗？

**用户：**
> 是

**Agent 写入更新。Story 状态：`ready-for-dev`。**

> **判定：就绪**
> Story 已准备好实施。在你准备好构建时领取。

---

## 阶段2：实施

### 第5轮：用户开始实施

**用户：**
> 让我们实施 STORY-MOV-001。我准备好开始编码了。

### 第6轮：Agent 读取 Story 和 ADR

**Agent（gameplay-programmer）：**
> 读取 `production/stories/STORY-MOV-001.md`、`design/gdd/movement-system.md` 和 `docs/architecture/movement-adr-001.md`...
>
> **架构（来自 ADR-001）：**
> - `MovementComponent.gd` — 附加到玩家场景，不是单例
> - 地形修饰符查找通过 `TerrainLayer.get_modifier(tile_position)` 辅助函数
> - 耐力事件通过信号（不直接调用 StaminaSystem）
>
> **我提议的文件结构：**
> ```
> src/gameplay/movement/
>   movement_component.gd       # CharacterBody2D 逻辑，状态机
>   terrain_modifier_table.gd   # 常量表：terrain_type -> {speed_mod, drain_mod}
> tests/unit/
>   test_movement_formulas.gd   # 速度/消耗计算的单元测试
> ```
>
> 可以用这个结构继续吗？

**用户：**
> 是

### 第7轮：Agent 实施并显示关键代码

**Agent：**
> 实施 `movement_component.gd`。在你写入文件之前的关键代码片段供你审查：
>
> ```gdscript
> class_name MovementComponent
> extends Node
>
> signal stamina_event(type: String, amount: float)
>
> @export var base_walk_speed: float = 120.0
> @export var run_multiplier: float = 1.7
> @export var stamina_drain_walk: float = 2.0
> @export var stamina_drain_run: float = 6.0
> @export var stamina_cost_roll: float = 15.0
>
> var _is_carrying_heavy: bool = false
>
> func _physics_process(delta: float) -> void:
>     var terrain_mod := _get_terrain_modifier()
>     var effective_speed := _calculate_speed(terrain_mod)
>     # ...
> ```
>
> 所有值都是 `@export` — 无硬编码。这是你的方向吗？

**用户：**
> 看起来好。写入它。

### 第8-9轮：Agent 写入文件和测试

**Agent 写入：**
- `src/gameplay/movement/movement_component.gd`
- `src/gameplay/movement/terrain_modifier_table.gd`
- `tests/unit/test_movement_formulas.gd`

**Agent：**
> 文件已写入。运行可用的 lint 检查...
>
> 无错误。测试文件创建，包含6个单元测试覆盖：
> - 行走速度公式（3种地形类型）
> - 奔跑速度公式
> - 耐力消耗计算
> - 翻滚耐力成本（有/无零速度边缘情况）
>
> 需要手动测试：无敌帧、坡度阻止、地形集成（需要运行场景）。

---

## 阶段3：Story 完成

### 第10轮：用户手动测试并运行 /story-done

**用户：**
> 我已在场景中测试——一切正常。/story-done production/stories/STORY-MOV-001.md

### 第11轮：Agent 运行完成审查

**Agent（通过 /story-done）：**
> 读取 story、GDD、ADR...
>
> **阶段1：Story 已定位** ✅ `production/stories/STORY-MOV-001.md`
>
> **阶段2：验收标准验证**
>
> | # | 标准 | 验证 | 状态 |
> |---|-----------|--------------|--------|
> | 1 | 普通地面上行走速度正确 | 自动：公式单元测试通过 | ✅ 自动 |
> | 2 | 奔跑速度正确 | 自动：公式单元测试通过 | ✅ 自动 |
> | 3 | 耐力0时奔跑取消 | 手动：用户确认 | ✅ 手动 |
> | 4 | 翻滚无敌帧工作 | 手动：用户确认 | ✅ 手动 |
> | 5 | 地形修饰符正确应用 | 手动：用户确认 | ✅ 手动 |
> | 6 | 重物携带禁用奔跑 | 自动：尚未可测试（库存未集成） | ⏳ 推迟 |
> | 7 | 耐力信号正确触发 | 自动：集成测试待定 | ⏳ 推迟 |
> | 8 | 无硬编码值 | 自动：grep 魔法数字 → 未找到 | ✅ 自动 |
> | 9 | 坡度>45°阻止移动 | 手动：用户确认 | ✅ 手动 |
>
> **阶段3：GDD/ADR 偏差**
>
> 扫描实施与 TR-ID...
>
> - TR-MOV-001 到 TR-MOV-007：✅ 按指定实施
> - TR-MOV-008（重物携带信号）：⚠️ 咨询性 — 推迟到库存集成 story。不是偏差，只是现在无法验证。
>
> **阶段4：清单陈旧检查**
>
> Story 控制清单版本 `2026-03-10` 匹配当前清单 `2026-03-10`。✅
>
> **阶段5：代码审查提示**
>
> 建议在标记完成前运行 `/code-review src/gameplay/movement/`。不阻塞。
>
> ---
> **完成报告：带备注完成**
>
> Story 完成。两个标准推迟到库存集成——记录为推迟，不是阻塞。
>
> 可以将 story 状态更新为 `Status: Complete` 并添加完成备注，以及更新 `production/sprint-status.yaml` 吗？

**用户：**
> 是

### 第12轮：Agent 关闭 Story

**Agent 写入：**
- Story `状态：完成` + 完成备注（注意推迟的标准）
- `production/sprint-status.yaml` — STORY-MOV-001 设置为 `done`

**Agent：**
> STORY-MOV-001 已关闭。
>
> **推迟的标准已记录**（将在库存集成 story 中自动浮现）：
> - 重物携带禁用奔跑（需要库存信号）
> - 耐力信号集成测试（需要耐力系统）
>
> **Sprint 中下一个就绪 story：**
> - STORY-MOV-002：耐力系统 — 这是合乎逻辑的下一个 story（下游依赖现在解锁）
> - STORY-HUD-001：移动 HUD 指示器 — 也解锁
>
> 在你领取下一个之前运行 `/story-readiness`。

---

## 此示例演示什么

- **就绪门捕获真实问题**：没有 `/story-readiness`，翻滚方向歧义会导致后期实施决策
- **ADR 状态是硬门**：如果 `movement-adr-001.md` 是 `Proposed` 而不是 `Accepted`，story 将被 `BLOCKED`，实施不会开始
- **推迟的标准**：不是每个验收标准都可在 story 关闭时验证 — `/story-done` 追踪推迟项目而非阻塞完成
- **TR-ID 引用**：story 存储 `TR-MOV-001` ID，而非引用 GDD 文本 — 当 GDD 措辞 later 清理时避免错误偏差标记
- **sprint-status.yaml**：Sprint 状态的单一真实来源，由 `/story-done` 更新并由 `/sprint-status` 读取
- **清单版本检查**：确保 story 的实施指导没有偏离当前架构决策
