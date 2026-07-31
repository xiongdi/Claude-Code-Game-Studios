---
name: security-audit
description: "Audit the game for security vulnerabilities: save tampering, cheat vectors, network exploits, data exposure, and input validation gaps. Produces a prioritised security report with remediation guidance. Run before any public release or multiplayer launch."
argument-hint: "[full | network | save | input | quick]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write, Task
model: sonnet
agent: security-engineer
---

# 安全审计

安全对于任何已发布的游戏都不是可选的。即使是单人游戏也有
存档篡改向量。多人游戏有作弊面、数据泄露
风险和拒绝服务潜力。此 skill 系统性地审计代码库中最常见的游戏安全故障，并生成优先的修复计划。

**运行此 skill：**
- 在任何公开发布之前（Polish → Release gate 必需）
- 在启用任何在线/多人功能之前
- 在实现任何从磁盘或网络读取的系统之后
- 在报告与安全相关的 bug 时

**输出：** `production/security/security-audit-[日期].md`

---

## Phase 1: 解析参数和范围

**模式：**
- `full` — 所有类别（发布前推荐）
- `network` — 仅网络/多人
- `save` — 仅存档文件和序列化
- `input` — 仅输入验证和注入
- `quick` — 仅高严重性检查（最快，用于迭代）
- 无参数 — 运行 `full`

读取 `.claude/docs/technical-preferences.md` 以确定：
- 引擎和语言（影响要搜索的模式）
- 目标平台（影响适用的攻击面）
- 多人/网络是否在范围内

---

## Phase 2: Spawn Security Engineer

通过 Task spawn `security-engineer`。传递：
- 审计范围/模式
- 技术偏好中的引擎和语言
- 所有源目录的清单：`src/`、`assets/data/`、任何配置文件

security-engineer 跨 6 个类别运行审计（见 Phase 3）。在进行之前收集他们的完整发现。

---

## Phase 3: 审计类别

security-engineer 评估以下每个类别。跳过不适用于项目范围的类别。

### 类别 1: 存档文件和序列化安全
- 存档文件在加载前是否经过验证？（无盲目反序列化）
- 存档文件路径是否从用户输入构建？（路径遍历风险）
- 存档文件是否有校验和或签名？（篡改检测）
- 游戏是否信任来自存档文件的数值而不进行边界检查？
- 在存档加载附近是否有任何 eval() 或动态代码执行调用？

Grep 模式：`File.open`、`load`、`deserialize`、`JSON.parse`、`from_json`、`read_file` — 检查每个是否有验证。

### 类别 2: 网络和多人安全（如果是纯单人游戏则跳过）
- 游戏状态在服务器上是权威的吗，还是客户端决定结果？
- 传入的网络数据包是否经过大小、类型和值范围的验证？
- 玩家位置和状态更改是否在服务器端验证？
- 任何网络调用是否有速率限制？
- 身份验证令牌是否正确处理（永远不以明文发送）？
- 游戏是否在发布构建中暴露任何调试端点？

Grep 搜索：`recv`、`receive`、`PacketPeer`、`socket`、`NetworkedMultiplayerPeer`、`rpc`、`rpc_id` — 检查每个调用站点是否有验证。

### 类别 3: 输入验证
- 是否有任何玩家提供的字符串用于文件路径？（路径遍历）
- 是否有任何玩家提供的字符串未经净化就记录？（日志注入）
- 数值输入（例如，物品数量、角色属性）在使用前是否经过边界检查？
- 成就/统计值在写入任何后端之前是否经过检查？

Grep 搜索：`get_input`、`Input.get_`、`input_map`、用户面向的文本字段 — 检查验证。

### 类别 4: 数据暴露
- 任何 API 密钥、凭证或秘密是否硬编码在 `src/` 或 `assets/` 中？
- 调试符号或详细错误消息是否包含在发布构建中？
- 游戏是否将敏感玩家数据记录到磁盘或控制台？
- 是否有任何内部文件路径或系统信息暴露给玩家？

Grep 搜索：`api_key`、`secret`、`password`、`token`、`private_key`、`DEBUG`、`print(` 在面向发布的代码中。

### 类别 5: 作弊和防篡改向量
- 游戏关键值是否仅存储在内存中，而不是易于编辑的文件中？
- 任何关键游戏进度标志（例如，"已购买 DLC"）是否在服务器端验证？
- 是否有任何针对内存编辑工具（Cheat Engine 等）的多人游戏保护？
- 排行榜/分数提交在接受前是否经过验证？

注意：客户端反作弊在很大程度上是不可执行的。专注于任何竞争性或货币化内容的服务器端验证。

### 类别 6: 依赖和供应链
- 是否使用了任何第三方插件或库？列出它们。
- 任何插件在使用的版本中是否有已知的 CVE？
- 插件来源是否经过验证（官方市场、审查过的仓库）？

Glob 搜索：`addons/`、`plugins/`、`third_party/`、`vendor/` — 列出所有外部依赖。

---

## Phase 4: 分类发现

对于每个发现，分配：

**严重性：**
| 级别 | 定义 |
|-------|-----------|
| **CRITICAL** | 远程代码执行、数据泄露或破坏多人完整性的可利用作弊 |
| **HIGH** | 绕过进度的存档篡改、凭证泄露或服务器端权限绕过 |
| **MEDIUM** | 客户端作弊启用、信息泄露或影响有限的输入验证缺口 |
| **LOW** | 纵深防御改进 — 减少攻击面但不存在直接利用的加固 |

**状态：** Open / Accepted Risk / Out of Scope

---

## Phase 5: 生成报告

```markdown
# 安全审计报告

**日期**: [日期]
**范围**: [full | network | save | input | quick]
**引擎**: [引擎 + 版本]
**审计者**: security-engineer via /security-audit
**扫描的文件**: [N 个源文件，N 个配置文件]

---

## 执行摘要

| 严重性 | 数量 | 发布前必须修复 |
|----------|-------|------------------------|
| CRITICAL | [N] | 是 — 全部 |
| HIGH | [N] | 是 — 全部 |
| MEDIUM | [N] | 推荐 |
| LOW | [N] | 可选 |

**发布建议**: [可以发布 / 先修复关键问题 / 不要发布]

---

## CRITICAL 发现

### SEC-001: [标题]
**类别**: [存档 / 网络 / 输入 / 数据 / 作弊 / 依赖]
**文件**: `[路径]` 第 [N] 行
**描述**: [漏洞是什么]
**攻击场景**: [恶意用户如何利用它]
**修复**: [要应用的具体代码更改或模式]
**工作量**: [低 / 中 / 高]

[每个发现重复]

---

## HIGH 发现

[相同格式]

---

## MEDIUM 发现

[相同格式]

---

## LOW 发现

[相同格式]

---

## 已接受的风险

[团队明确接受并带有理由的任何发现]

---

## 依赖清单

| 插件 / 库 | 版本 | 来源 | 已知 CVE |
|-----------------|---------|--------|------------|
| [名称] | [版本] | [来源] | [无 / CVE-XXXX-NNNN] |

---

## 修复优先级顺序

1. [SEC-NNN] — [一行描述] — 估算工作量: [低/中/高]
2. ...

---

## 重新审计触发器

在修复任何 CRITICAL 或 HIGH 发现后运行 `/security-audit`。
Polish → Release gate 要求此报告无开放的 CRITICAL 或 HIGH 项。
```

---

## Phase 6: 写入报告

在对话中展示报告摘要（执行摘要 + 仅 CRITICAL/HIGH 发现）。

询问："我可以将完整的安全审计报告写入 `production/security/security-audit-[日期].md` 吗？"

仅在批准后写入。

---

## Phase 7: Gate 集成

此报告是 **Polish → Release gate** 的必需资源。

修复发现后，重新运行：`/security-audit quick` 以在运行 `/gate-check release` 之前确认 CRITICAL/HIGH 项已解决。

如果存在 CRITICAL 发现：
> "⛔ CRITICAL 安全发现必须在任何公开发布之前解决。在这些问题解决之前不要继续到 `/launch-checklist`。"

如果没有 CRITICAL/HIGH 发现：
> "✅ 无阻塞安全发现。报告已写入 `production/security/`。运行 `/gate-check release` 时包含此路径。"

---

## 协作协议

- **永远不要假设模式是安全的** — 标记它并让用户决定
- **已接受的风险是有效结果** — 一些 LOW 发现对独立团队是可接受的权衡；记录决策
- **多人游戏有更高标准** — 多人环境中的任何 HIGH 发现应被视为 CRITICAL
- **这不是渗透测试** — 此审计涵盖常见模式；在竞争或货币化多人发布之前，建议由人类安全专业人员进行真正的渗透测试
