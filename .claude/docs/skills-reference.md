# 可用技能（Slash Commands）

73 个按阶段组织的 Slash 命令。在 Claude Code 中输入 `/` 访问其中任何一个。

## 入门与导航

| 命令 | 用途 |
|---------|---------|
| `/start` | 首次入门 — 询问你处于哪个阶段，然后引导你到正确的工作流程 |
| `/help` | 上下文感知的"我下一步该做什么？" — 读取当前阶段并显示所需的下一个步骤 |
| `/project-stage-detect` | 完整项目审计 — 检测阶段、识别存在差距、推荐下一步 |
| `/setup-engine` | 配置引擎 + 版本、检测知识差距、填充版本感知参考文档 |
| `/adopt` | Brownfield 格式审计 — 检查现有 GDD/ADR/Story 的内部结构，生成迁移计划 |

## 游戏设计

| 命令 | 用途 |
|---------|---------|
| `/brainstorm` | 使用专业工作室方法引导构思（MDA、SDT、Bartle、动词优先） |
| `/map-systems` | 将游戏概念分解为系统，映射依赖，优先化设计顺序 |
| `/design-system` | 引导逐章节为单个游戏系统创作 GDD |
| `/quick-design` | 轻量级设计规格用于小更改 — 调优、调整、小添加 |
| `/review-all-gdds` | 跨 GDD 一致性和游戏设计整体性审查 |
| `/propagate-design-change` | 当 GDD 修订时，找到受影响的 ADR 并生成影响报告 |

## 美术与资源

| 命令 | 用途 |
|---------|---------|
| `/art-bible` | 引导式逐节 Art Bible 创作 — 在资产生产开始前创建视觉身份规格 |
| `/asset-spec` | 从 GDD、关卡文档或角色档案生成每资产视觉规格和 AI 生成提示 |
| `/asset-audit` | 审计资源的命名约定、文件大小预算和管线合规 |

## UX 与界面设计

| 命令 | 用途 |
|---------|---------|
| `/ux-design` | 引导逐章节创作 UX 规格（屏幕/流程、HUD 或模式库） |
| `/ux-review` | 验证 UX 规格是否符合 GDD 一致性、无障碍和模式合规 |

## 架构

| 命令 | 用途 |
|---------|---------|
| `/create-architecture` | 引导创作主架构文档 |
| `/architecture-decision` | 创建一个 ADR |
| `/architecture-review` | 验证所有 ADR 的完整性、依赖排序和 GDD 覆盖率 |
| `/create-control-manifest` | 从 Accepted ADR 生成扁平化程序员规则表 |

## Story 与 Sprint

| 命令 | 用途 |
|---------|---------|
| `/create-epics` | 将 GDD + ADR 翻译为 Epic（每个架构模块一个） |
| `/create-stories` | 将单个 Epic 分解为可实现的 Story 文件 |
| `/dev-story` | 读取 Story 并实现 — 路由到正确的程序员 Agent |
| `/sprint-plan` | 生成或更新 Sprint 计划；初始化 sprint-status.yaml |
| `/sprint-status` | 快速 30 行 Sprint 快照（读取 sprint-status.yaml） |
| `/story-readiness` | 验证 Story 在领取前是否可实现（READY/NEEDS WORK/BLOCKED） |
| `/story-done` | 实现后的 8 阶段完成审查；更新 Story 文件，显示下一个 Story |
| `/estimate` | 带复杂度、依赖和风险分解的结构化工作量估算 |

## 评审与分析

| 命令 | 用途 |
|---------|---------|
| `/design-review` | 审查游戏设计文档的完整性和一致性 |
| `/code-review` | 文件或更改集的架构代码审查 |
| `/balance-check` | 分析游戏平衡数据、公式和配置 — 标记异常值 |
| `/content-audit` | GDD 指定的内容计数与已实现内容的审计 |
| `/scope-check` | 分析功能或 Sprint 范围与原始计划的对比，标记范围蔓延 |
| `/perf-profile` | 带瓶颈识别的结构化性能分析 |
| `/tech-debt` | 扫描、追踪、优先化和报告技术债务 |
| `/gate-check` | 验证开发阶段之间前进的就绪状态（PASS/CONCERNS/FAIL） |
| `/consistency-check` | 扫描所有 GDD 与实体注册表，检测跨文档不一致（相互矛盾的属性、名称、规则） |
| `/security-audit` | 审计游戏的安全漏洞：存档篡改、作弊向量、网络利用、数据暴露和输入验证缺口 |

## QA 与测试

| 命令 | 用途 |
|---------|---------|
| `/qa-plan` | 为 Sprint 或功能生成 QA 测试计划 |
| `/smoke-check` | 在 QA 交接前运行关键路径冒烟测试门 |
| `/soak-test` | 为长时间游戏会话生成浸泡测试协议 |
| `/regression-suite` | 将覆盖率映射到 GDD 关键路径，识别缺少回归测试的已修复 Bug |
| `/test-setup` | 为项目引擎搭建测试框架和 CI/CD 管线 |
| `/test-helpers` | 为测试套件生成引擎特定测试辅助库 |
| `/test-evidence-review` | 测试文件和手动证据的质量审查 |
| `/test-flakiness` | 从 CI 运行历史检测 flaky 测试，标记隔离或修复 |
| `/skill-test` | 验证技能文件是否符合结构和行为正确性 |
| `/skill-improve` | 使用 test-fix-retest 循环改进技能 — 诊断、建议修复、重写、验证 |

## 制作

| 命令 | 用途 |
|---------|---------|
| `/milestone-review` | 审查里程碑进度并生成状态报告 |
| `/retrospective` | 运行结构化 Sprint 或里程碑回顾 |
| `/bug-report` | 创建结构化 Bug 报告 |
| `/bug-triage` | 读取所有开放 Bug，重新评估优先级 vs 严重性，分配所有者和标签 |
| `/reverse-document` | 从现有实现生成设计或架构文档 |
| `/playtest-report` | 生成结构化试玩报告或分析现有试玩笔记 |

## 发布

| 命令 | 用途 |
|---------|---------|
| `/release-checklist` | 为当前构建生成和验证预发布检查清单 |
| `/launch-checklist` | 跨所有部门的完整发布就绪验证 |
| `/changelog` | 从 git 提交和 Sprint 数据自动生成变更日志 |
| `/patch-notes` | 从 git 历史和内部数据生成玩家面向的补丁说明 |
| `/hotfix` | 带审计追踪的紧急修复，绕过正常 Sprint 流程 |
| `/day-one-patch` | 为金主后发现但发布前或发布时已知的 issues 准备聚焦的 day-one 补丁 |

## 创意与内容

| 命令 | 用途 |
|---------|---------|
| `/prototype` | 概念原型 — brainstorm 后立即构建一次性构建验证核心想法（阶段 1） |
| `/vertical-slice` | Pre-Production 验证 — 投入 Production 前构建生产级端到端构建（阶段 4） |
| `/onboard` | 为新贡献者或 Agent 生成上下文入门文档 |
| `/localize` | 本地化工作流：字符串提取、验证、翻译就绪 |

## 团队编排

在单个功能区域协调多个 Agent：

| 命令 | 协调 |
|---------|-------------|
| `/team-combat` | game-designer + gameplay-programmer + ai-programmer + technical-artist + sound-designer + qa-tester |
| `/team-narrative` | narrative-director + writer + world-builder + level-designer |
| `/team-ui` | ux-designer + ui-programmer + art-director + accessibility-specialist |
| `/team-release` | release-manager + qa-lead + devops-engineer + producer |
| `/team-polish` | performance-analyst + technical-artist + sound-designer + qa-tester |
| `/team-audio` | audio-director + sound-designer + technical-artist + gameplay-programmer |
| `/team-level` | level-designer + narrative-director + world-builder + art-director + systems-designer + qa-tester |
| `/team-live-ops` | live-ops-designer + economy-designer + community-manager + analytics-engineer |
| `/team-qa` | qa-lead + qa-tester + gameplay-programmer + producer |