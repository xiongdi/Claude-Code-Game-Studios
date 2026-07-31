# 编码标准

- 所有游戏代码必须在公共 API 上包含文档注释
- 每个系统必须在 `docs/architecture/` 中有对应的架构决策记录
- 游戏玩法值必须是数据驱动的（外部配置），禁止硬编码
- 所有公共方法必须可单元测试（依赖注入优于单例）
- 提交必须引用相关的设计文档或任务 ID
- **提交信息**：使用 Conventional Commits 格式 — `feat:`、`fix:`、`chore:`、`docs:`、`test:`、`refactor:`。在正文中引用 story 或 task ID（例如 `Story: EPIC-001-S02`）。
- **验证驱动开发**：添加游戏系统时先写测试。
  对于 UI 更改，用截图验证。在标记工作完成前比较预期输出与实际输出。
  每个实现都应该有办法证明它有效。

# 设计文档标准

- 所有设计文档使用 Markdown
- 每个机制在 `design/gdd/` 中有专属文档
- 文档必须包含这 8 个必需章节：
  1. **Overview（概览）** -- 一段式摘要
  2. **Player Fantasy（玩家幻想）** -- 预期感受和体验
  3. **Detailed Rules（详细规则）** -- 明确的机制
  4. **Formulas（公式）** -- 所有数学变量定义
  5. **Edge Cases（边缘情况）** -- 处理的异常情况
  6. **Dependencies（依赖）** -- 列出其他系统
  7. **Tuning Knobs（调优旋钮）** -- 识别可配置值
  8. **Acceptance Criteria（验收标准）** -- 可测试的成功条件
- 平衡值必须链接到其来源公式或原理

# 测试标准

## 按 Story 类型的测试证据

所有 Story 在标记为 Done 前必须有适当的测试证据：

| Story 类型 | 必需证据 | 位置 | 门控级别 |
|---|---|---|---|
| **Logic（逻辑）**（公式、AI、状态机） | 自动化单元测试 — 必须通过 | `tests/unit/[system]/` | BLOCKING |
| **Integration（集成）**（多系统） | 集成测试或记录的试玩 | `tests/integration/[system]/` | BLOCKING |
| **Visual/Feel（视觉/手感）**（动画、VFX、手感） | 截图 + 主管签字 | `production/qa/evidence/` | ADVISORY |
| **UI**（菜单、HUD、屏幕） | 手动演练文档或交互测试 | `production/qa/evidence/` | ADVISORY |
| **Config/Data（配置/数据）**（平衡调优） | 冒烟检查通过 | `production/qa/smoke-[date].md` | ADVISORY |

## 自动化测试规则

- **命名**：文件用 `[system]_[feature]_test.[ext]`；函数用 `test_[scenario]_[expected]`
- **确定性**：测试每次运行必须产生相同结果 — 无随机种子，无时间依赖断言
- **隔离**：每个测试设置和拆除自己的状态；测试不得依赖执行顺序
- **无硬编码数据**：测试 fixture 使用常量文件或工厂函数，不用内联魔法数字
  （例外：边界值测试中精确数字本身就是重点）
- **独立性**：单元测试不调用外部 API、数据库或文件 I/O — 使用依赖注入

## 不自动化的内容

- 视觉保真度（shader 输出、VFX 外观、动画曲线）
- "手感"质量（输入响应、感知重量、时机）
- 平台特定渲染（在目标硬件上测试，而非无头）
- 完整游戏会话（由试玩覆盖，不由自动化覆盖）

## CI/CD 规则

- 自动化测试套件在每次推送到 main 和每个 PR 时运行
- 测试失败则不能合并 — 测试是 CI 中的阻塞门
- 永远不要禁用或跳过失败的测试来让 CI 通过 — 修复根本问题
- 引擎特定 CI 命令：
  - **Godot**: `godot --headless --script tests/gdunit4_runner.gd`
  - **Unity**: `game-ci/unity-test-runner@v4` (GitHub Actions)
  - **Unreal**: 带 `-nullrhi` 标志的无头运行器