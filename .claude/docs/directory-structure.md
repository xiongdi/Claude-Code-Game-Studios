# 目录结构

```text
/
├── CLAUDE.md                    # 主配置
├── .claude/                     # Agent 定义、技能、Hooks、规则、文档
├── src/                         # 游戏源代码（core、gameplay、ai、networking、ui、tools）
├── assets/                      # 游戏资源（美术、音频、vfx、shaders、数据）
├── design/                      # 游戏设计文档（gdd、叙事、关卡、平衡）
├── docs/                        # 技术文档（架构、API、复盘）
│   └── engine-reference/        # 策划的引擎 API 快照（版本固定）
├── tests/                       # 测试套件（单元、集成、性能、试玩）
├── tools/                       # 构建和管线工具（CI、构建、资源管线）
├── prototypes/                  # 一次性原型（与 src/ 隔离）
└── production/                  # 制作管理（sprints、里程碑、发布）
    ├── session-state/           # 临时会话状态（active.md — gitignored）
    └── session-logs/            # 会话审计追踪（gitignored）
```