# 设计目录

在此目录中创作或编辑文件时，请遵循以下标准。

## GDD 文件（`design/gdd/`）

每个 GDD 必须按此顺序包含全部 **8 个必需章节**：
1. Overview（概览）— 一段式摘要
2. Player Fantasy（玩家幻想）— 预期感受和体验
3. Detailed Rules（详细规则）— 明确的机制
4. Formulas（公式）— 所有数学变量定义
5. Edge Cases（边缘情况）— 处理的异常情况
6. Dependencies（依赖）— 列出其他系统
7. Tuning Knobs（调优旋钮）— 识别可配置值
8. Acceptance Criteria（验收标准）— 可测试的成功条件

**文件命名：** `[system-slug].md`（例如 `movement-system.md`、`combat-system.md`）

**系统索引：** `design/gdd/systems-index.md` — 添加新 GDD 时更新。

**设计顺序：** 基础 → 核心 → 功能 → 表现 → 打磨

**验证：** 创作任何 GDD 后运行 `/design-review [path]`。
完成一组相关 GDD 后运行 `/review-all-gdds`。

## 快速规格（`design/quick-specs/`）

用于调优更改、小型机制或平衡调整的轻量级规格。
使用 `/quick-design` 创作。

## UX 规格（`design/ux/`）

- 每屏幕规格：`design/ux/[screen-name].md`
- HUD 设计：`design/ux/hud.md`
- 交互模式库：`design/ux/interaction-patterns.md`
- 无障碍要求：`design/ux/accessibility-requirements.md`

使用 `/ux-design` 创作。在提交到 `/team-ui` 前用 `/ux-review` 验证。