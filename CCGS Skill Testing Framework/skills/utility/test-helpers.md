# Skill 测试规格：/test-helpers

## Skill 摘要

`/test-helpers` 为项目的测试套件生成引擎专用的测试辅助工具。辅助工具包括工厂函数（用于创建具有已知状态的测试实体）、fixture 加载器、断言辅助工具以及外部依赖的 mock stub。生成的辅助工具遵循 `coding-standards.md` 中的命名和结构约定，并写入 `tests/helpers/`。

每个辅助文件都需要经过 "May I write" 询问。如果辅助文件已存在，该 skill 会提供扩展而不是替换。不适用 director gate。当辅助文件写入后裁决为 COMPLETE。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含裁决关键词：COMPLETE
- [ ] 在写入辅助工具前包含 "May I write" 协作协议语言
- [ ] 有下一步交接（例如，使用生成的辅助工具编写测试）

---

## Director 关卡检查

无。`/test-helpers` 是一个脚手架工具。不适用 director gate。

---

## 测试用例

### 用例 1：正常路径——为 Godot/GDScript 生成 Player 工厂辅助工具

**Fixture：**
- `technical-preferences.md` 配置引擎为 Godot 4，语言为 GDScript
- `tests/` 目录存在（已运行 test-setup）
- `design/gdd/player.md` 存在并定义了玩家属性
- `tests/helpers/` 中不存在现有辅助工具

**输入：** `/test-helpers player-factory`

**预期行为：**
1. Skill 读取引擎（Godot 4 / GDScript）和玩家 GDD 以获取属性上下文
2. Skill 生成确定性的 GDScript `PlayerFactory` 辅助工具：
   - `create_player(health: int = 100, speed: float = 200.0)` 函数
   - 返回预配置到已知状态的玩家节点
   - 使用依赖注入（无单例）
3. Skill 询问 "May I write to `tests/helpers/player_factory.gd`?"
4. 批准后写入文件；裁决为 COMPLETE

**断言：**
- [ ] 生成的辅助工具是 GDScript（不是 C# 或 Blueprint）
- [ ] 工厂函数参数使用与 GDD 值匹配的默认值
- [ ] 辅助工具使用依赖注入（无 Autoload/单例引用）
- [ ] 文件名遵循 GDScript 的 snake_case 约定
- [ ] 裁决为 COMPLETE

---

### 用例 2：不存在测试设置——重定向到 /test-setup

**Fixture：**
- `tests/` 目录不存在

**输入：** `/test-helpers player-factory`

**预期行为：**
1. Skill 检查 `tests/` 目录——未找到
2. Skill 报告："Test directory not found — test framework must be set up first"
3. Skill 建议在生成辅助工具前先运行 `/test-setup`
4. 不创建辅助文件

**断言：**
- [ ] 错误消息识别缺失的 tests/ 目录
- [ ] 建议 `/test-setup` 作为先决步骤
- [ ] 不调用写入工具
- [ ] 裁决不是 COMPLETE（阻塞状态）

---

### 用例 3：辅助工具已存在——提供扩展而不是替换

**Fixture：**
- `tests/helpers/player_factory.gd` 已存在，包含 `create_player()` 函数
- 用户请求向工厂添加新的 `create_enemy()` 函数

**输入：** `/test-helpers enemy-factory`

**预期行为：**
1. Skill 找到现有的 `player_factory.gd` 并检查是否是要扩展的正确文件（或者是否应该创建单独的 `enemy_factory.gd`）
2. Skill 展示选项：将 `create_enemy()` 添加到现有工厂或创建 `tests/helpers/enemy_factory.gd`
3. 用户选择扩展；skill 起草 `create_enemy()` 函数
4. Skill 询问 "May I extend `tests/helpers/player_factory.gd`?"
5. 批准后添加函数；裁决为 COMPLETE

**断言：**
- [ ] 检测并呈现现有辅助工具
- [ ] 用户获得扩展 vs. 新文件的选择
- [ ] 使用 "May I extend" 语言（不是用于替换的 "May I write"）
- [ ] 扩展文件中保留现有的 `create_player()`
- [ ] 裁决为 COMPLETE

---

### 用例 4：系统无 GDD——在辅助工具中注明缺失的设计上下文

**Fixture：**
- `technical-preferences.md` 配置为 Godot 4 / GDScript
- `tests/` 存在
- 用户请求为 "inventory system" 创建辅助工具，但不存在 `design/gdd/inventory.md`

**输入：** `/test-helpers inventory-factory`

**预期行为：**
1. Skill 查找 `design/gdd/inventory.md`——未找到
2. Skill 注明："No GDD found for inventory — generating helper with placeholder defaults"
3. Skill 生成包含通用占位符值的 `inventory_factory.gd`
   （item_count = 0, max_capacity = 20）并附带注释："# TODO: align defaults
   with inventory GDD when written"
4. Skill 询问 "May I write to `tests/helpers/inventory_factory.gd`?"
5. 文件已写入；裁决为 COMPLETE 并附带建议说明

**断言：**
- [ ] Skill 在没有 GDD 的情况下继续（不阻塞）
- [ ] 生成的辅助工具具有带 TODO 注释的占位符默认值
- [ ] 输出中注明缺失的 GDD（建议性警告）
- [ ] 裁决为 COMPLETE

---

### 用例 5：Director 关卡检查——无 gate；test-helpers 是脚手架实用工具

**Fixture：**
- 引擎已配置，tests/ 存在

**输入：** `/test-helpers player-factory`

**预期行为：**
1. Skill 生成并写入辅助文件
2. 不派生 director agent
3. 输出中不出现 gate ID

**断言：**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 无需任何 gate 检查即可达到 COMPLETE

---

## 协议合规性

- [ ] 在生成任何辅助工具前读取引擎（辅助工具是引擎专用的）
- [ ] 在可用时读取 GDD 获取默认值
- [ ] 注明缺失的 GDD 上下文而不是阻塞
- [ ] 检测现有辅助文件并提供扩展而不是替换
- [ ] 在任何文件操作前询问 "May I write"（或 "May I extend"）
- [ ] 辅助工具写入后裁决为 COMPLETE

---

## 覆盖说明

- Mock/stub 辅助工具生成（用于保存系统或音频总线等依赖项）遵循与工厂辅助工具相同的模式，未单独测试。
- Unity C# 辅助工具生成（使用 NSubstitute 或自定义 mock）遵循与用例 1 相同的逻辑，输出与语言匹配。
- 请求的辅助工具类型无法识别的情况未测试；skill 会要求用户澄清辅助工具类型。
