# 技能测试规格: /test-setup

## 技能摘要

`/test-setup` 根据配置的引擎为项目搭建测试框架。它创建 `coding-standards.md` 中定义的 `tests/` 目录结构（unit/、integration/、performance/、playtest/）并为检测到的引擎生成适当的测试运行器配置：Godot 的 GdUnit4 配置、Unity 的 Unity Test Runner asmdef 或 Unreal Engine 的无头运行器。

创建的每个文件或目录都需要通过 "May I write" 请求。如果测试框架已存在，该 skill 验证配置而不是重新初始化。不适用 director gate。当脚手架就位时裁定为 COMPLETE。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁定关键词：COMPLETE
- [ ] 在创建文件前包含 "May I write" 协作协议语言
- [ ] 具有下一步交接说明（例如，`/test-helpers` 生成辅助工具）

---

## 导演门控检查

无。`/test-setup` 是搭建实用工具。不适用 director gate。

---

## 测试用例

### Case 1: Happy Path——Godot 项目，搭建 GdUnit4 测试结构

**Fixture:**
- `technical-preferences.md` 将引擎设置为 Godot 4，语言为 GDScript
- `tests/` 目录尚不存在

**Input:** `/test-setup`

**Expected behavior:**
1. Skill 从 `technical-preferences.md` 读取引擎 → Godot 4 + GDScript
2. Skill 起草测试目录结构：tests/unit/、tests/integration/、
   tests/performance/、tests/playtest/ 和 GdUnit4 运行器配置文件
3. Skill 询问 "May I write the tests/ directory structure?"
4. 批准后创建目录和 GdUnit4 运行器脚本
5. Skill 确认运行器脚本与 coding-standards.md 中的 CI 命令匹配：
   `godot --headless --script tests/gdunit4_runner.gd`
6. 裁定为 COMPLETE

**Assertions:**
- [ ] 创建所有 4 个子目录（unit/、integration/、performance/、playtest/）
- [ ] 生成 GdUnit4 运行器配置
- [ ] 运行器脚本路径与 coding-standards.md CI 命令匹配
- [ ] 在创建任何文件前询问 "May I write"
- [ ] 裁定为 COMPLETE

---

### Case 2: Unity Project——使用 asmdef 搭建 Unity Test Runner

**Fixture:**
- `technical-preferences.md` 将引擎设置为 Unity，语言为 C#
- `tests/` 目录不存在

**Input:** `/test-setup`

**Expected behavior:**
1. Skill 读取引擎 → Unity + C#
2. Skill 创建具有 Unity 约定的 `Tests/` 目录（大写）
3. Skill 生成 `Tests/Tests.asmdef` 和 `Tests/Editor/EditorTests.asmdef`
4. 配置 EditMode 和 PlayMode 测试运行器模式
5. Skill 询问 "May I write the Tests/ directory structure?"
6. 裁定为 COMPLETE

**Assertions:**
- [ ] 创建 Unity 特定的 `Tests/` 结构（不是 Godot 结构）
- [ ] 生成 `.asmdef` 文件
- [ ] EditMode 和 PlayMode 运行器配置存在
- [ ] 裁定为 COMPLETE

---

### Case 3: Test Framework Already Exists——验证配置，不重新初始化

**Fixture:**
- `tests/unit/`、`tests/integration/` 存在
- GdUnit4 运行器脚本存在（Godot 项目）

**Input:** `/test-setup`

**Expected behavior:**
1. Skill 检测现有 tests/ 结构
2. Skill 报告："Test framework already exists — verifying configuration"
3. Skill 检查：运行器脚本路径、目录完整性、CI 命令对齐
4. 如果所有检查通过：报告 "Configuration verified — no changes needed"
5. 如果检查失败（例如，缺少 tests/performance/）：报告具体差距并
   询问 "May I add the missing directories?"

**Assertions:**
- [ ] Skill 在框架存在时不重新初始化
- [ ] 对现有结构执行验证检查
- [ ] 仅缺失部分触发 "May I write" 请求
- [ ] 无论一切正常还是修复了差距，裁定都为 COMPLETE

---

### Case 4: No Engine Configured——重定向到 /setup-engine

**Fixture:**
- `technical-preferences.md` 仅包含占位符（引擎未设置）

**Input:** `/test-setup`

**Expected behavior:**
1. Skill 读取 `technical-preferences.md` 并找到引擎占位符
2. Skill 报告："Engine not configured — cannot scaffold engine-specific test framework"
3. Skill 建议先运行 `/setup-engine`
4. 不创建目录或文件

**Assertions:**
- [ ] 错误消息明确说明引擎未配置
- [ ] `/setup-engine` 被建议为下一步
- [ ] 不调用写入工具
- [ ] 裁定不是 COMPLETE（阻塞状态）

---

### Case 5: Director Gate Check——无 gate；test-setup 是搭建实用工具

**Fixture:**
- 引擎已配置，tests/ 不存在

**Input:** `/test-setup`

**Expected behavior:**
1. Skill 搭建并写入所有测试框架文件
2. 不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 不调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] Skill 无需任何 gate 检查即达到 COMPLETE

---

## 协议合规

- [ ] 在生成任何脚手架前从 `technical-preferences.md` 读取引擎
- [ ] 生成引擎适当的测试运行器配置（不是通用的）
- [ ] 创建 coding-standards.md 中的所有 4 个子目录
- [ ] 在创建文件前询问 "May I write"
- [ ] 检测现有框架并提供验证（不重新初始化）
- [ ] 当脚手架就位时裁定为 COMPLETE

---

## 覆盖说明

- Unreal Engine 测试搭建（带 `-nullrhi` 的无头运行器）遵循与 Case 1 和 2 相同的模式，
  不单独进行 fixture 测试。
- CI 集成文件生成（例如，`.github/workflows/test.yml`）被引用但不在此进行断言测试——
  它可能是单独的 skill 关注点。
- tests/ 存在但来自不同引擎的情况（例如，现在是 Godot 项目中的 Unity 测试）不在此测试；
  skill 会检测不匹配并提供协调。
