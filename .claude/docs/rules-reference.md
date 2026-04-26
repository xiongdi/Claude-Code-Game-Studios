# 路径特定规则

`.claude/rules/` 中的规则在编辑匹配路径中的文件时自动执行：

| 规则文件 | 路径模式 | 强制执行 |
| ---- | ---- | ---- |
| `gameplay-code.md` | `src/gameplay/**` | 数据驱动值、增量时间、无 UI 引用 |
| `engine-code.md` | `src/core/**` | 热路径零分配、线程安全、API 稳定性 |
| `ai-code.md` | `src/ai/**` | 性能预算、可调试性、数据驱动参数 |
| `network-code.md` | `src/networking/**` | 服务器权威、版本化消息、安全性 |
| `ui-code.md` | `src/ui/**` | 无游戏状态所有权、本地化就绪、无障碍 |
| `design-docs.md` | `design/gdd/**` | 必需 8 个章节、公式格式、边缘情况 |
| `narrative.md` | `design/narrative/**` | 传说一致性、角色 voice、 canon 级别 |
| `data-files.md` | `assets/data/**` | JSON 有效性、命名约定、schema 规则 |
| `test-standards.md` | `tests/**` | 测试命名、覆盖率要求、fixture 模式 |
| `prototype-code.md` | `prototypes/**` | 放宽标准、必需 README、记录假设 |
| `shader-code.md` | `assets/shaders/**` | 命名约定、性能目标、跨平台规则 |