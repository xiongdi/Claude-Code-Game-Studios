# 文档目录

在此目录中创作或编辑文件时，请遵循以下标准。

## 架构决策记录（`docs/architecture/`）

使用 ADR 模板：`.claude/docs/templates/architecture-decision-record.md`

**必需章节：** Title、Status、Context、Decision、Consequences、
ADR Dependencies、Engine Compatibility、GDD Requirements Addressed

**状态生命周期：** `Proposed` → `Accepted` → `Superseded`
- 永远不要跳过 `Accepted` — 引用 `Proposed` ADR 的 Story 会被自动阻止
- 使用 `/architecture-decision` 通过引导流程创建 ADR

**TR 注册表：** `docs/architecture/tr-registry.yaml`
- 稳定的需求 ID（例如 `TR-MOV-001`），将 GDD 需求链接到 Story
- 永远不要重新编号现有 ID — 只追加新的
- 由 `/architecture-review` 第 8 阶段更新

**控制清单：** `docs/architecture/control-manifest.md`
- 扁平化程序员规则表：每层 Required / Forbidden / Guardrails
- 头部包含日期戳 `Manifest Version:`
- Story 嵌入此版本；`/story-done` 检查是否过时

**验证：** 完成一组 ADR 后运行 `/architecture-review`。

## 引擎参考（`docs/engine-reference/`）

版本固定的引擎 API 快照。**使用任何引擎 API 前始终检查此处** —
LLM 的训练数据早于固定的引擎版本。

当前引擎：见 `docs/engine-reference/godot/VERSION.md`