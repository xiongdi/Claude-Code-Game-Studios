# Agent Test Spec: devops-engineer

## Agent Summary
- **领域：** CI/CD 管线配置、构建脚本、版本控制工作流执行、部署基础设施、分支策略、环境管理、CI 中的自动化测试集成
- **不负责：** 游戏逻辑或玩法系统、安全审计（security-engineer）、QA 测试策略（qa-lead）、游戏网络逻辑（network-programmer）
- **模型层级：** Sonnet
- **Gate ID：** 无；将部署阻塞升级到 producer

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且特定于领域（引用 CI/CD、构建、部署、版本控制）
- [ ] `allowed-tools:` 列表与 agent 角色匹配（对管线配置文件、shell 脚本、YAML 的 Read/Write；无游戏源编辑工具）
- [ ] 模型层级为 Sonnet（operations specialist 默认值）
- [ ] Agent 定义不声称对游戏逻辑、安全审计或 QA 测试设计拥有权限

---

## 测试用例

### 用例 1：领域内请求 — Godot 项目的 CI 设置
**输入：** "为我们的 Godot 4 项目设置 CI 管线。它应该在每次推送到 main 和每次 pull request 时运行测试，如果测试失败则构建失败。"
**预期行为：**
- 生成 GitHub Actions 工作流 YAML（`.github/workflows/ci.yml` 或等效文件）
- 使用 `coding-standards.md` 中的 Godot 无头测试运行器命令：`godot --headless --script tests/gdunit4_runner.gd`
- 配置在推送到 main 和 `pull_request` 上的触发器
- 设置作业在测试失败时失败（`exit 1` 或非零退出）— 不配置管线在测试失败时继续
- 在输出或注释中引用项目的编码标准 CI 规则

### 用例 2：领域外请求 — 游戏网络实现
**输入：** "为我们的多人游戏实现服务器权威移动系统。"
**预期行为：**
- 不生成游戏网络或移动代码
- 清晰声明："游戏网络实现由 network-programmer 拥有；我处理构建、测试和部署游戏的基础设施"
- 不将 CI 管线配置与游戏内网络架构混为一谈

### 用例 3：构建失败诊断
**输入：** "我们的 CI 管线在合并步骤失败。错误是：'Asset import failed: texture compression format unsupported in headless mode.'"
**预期行为：**
- 诊断根本原因：无头 CI 环境不支持依赖 GPU 的纹理压缩
- 提出具体修复：在 CI 运行之前在本地预导入资产（将 .import 文件提交到 VCS），配置 Godot 的导入设置在 CI 中使用 CPU 兼容的压缩格式，或者如果可用则使用带有 GPU 模拟的 Docker 镜像
- 不宣布管线无法修复 — 提供至少一条可操作路径
- 注明任何权衡（提交 .import 文件增加仓库大小；CPU 压缩可能与 GPU 输出不同）

### 用例 4：分支策略冲突
**输入：** "团队一半人想要使用具有长期功能分支的 GitFlow。另一半想要基于主干的开发。我们应该如何设置这个？"
**预期行为：**
- 根据项目约定推荐基于主干的开发（CLAUDE.md / coordination-rules.md 指定使用 Git 和基于主干的开发）
- 在此项目背景下为推荐提供具体理由：较小的团队、更少的集成冲突、更快的 CI 反馈
- 如果项目有既定约定，不将此呈现为 50/50 选择
- 解释如果需要如何使用短期功能分支和功能标志实现基于主干的开发
- 不覆盖项目约定而不标记这样做需要更新 CLAUDE.md

### 用例 5：上下文传递 — 平台特定的构建矩阵
**输入上下文：** 项目目标为 PC（Windows、Linux）、Nintendo Switch 和 PlayStation 5。
**输入：** "设置我们的 CI 构建矩阵，以便我们在每次发布分支推送时获得每个目标平台的构建工件。"
**预期行为：**
- 生成具有三个平台条目的构建矩阵配置：Windows、Linux、Switch、PS5
- 应用平台适当的构建步骤：PC 使用标准 Godot 导出模板；Switch 和 PS5 需要平台特定的导出模板（注明控制台模板需要授权的 SDK 访问且不公开分发）
- 不假设所有平台可以使用相同的构建运行器 — 标记控制台构建可能需要具有授权 SDK 的自托管运行器
- 在管线输出中按平台名称组织工件

---

## 协议合规

- [ ] 在声明领域内保持（CI/CD、构建脚本、版本控制、部署）
- [ ] 将游戏逻辑和网络请求重定向到适当的程序员
- [ ] 在分支策略有争议时根据项目约定推荐基于主干的开发
- [ ] 返回结构化管线配置（YAML、脚本），而非自由格式建议
- [ ] 标记控制台构建的平台 SDK 许可约束，而非静默生成错误配置

---

## 覆盖说明
- 用例 1（Godot CI）引用 `coding-standards.md` CI 规则 — 运行此测试之前验证此文件存在且最新
- 用例 4（分支策略）是约定执行测试 — agent 必须知道项目约定，而非只给中立建议
- 用例 5 要求项目的目标平台已记录（在 `technical-preferences.md` 或等效文件中）
- 无自动化运行器；手动审查或通过 `/skill-test`
