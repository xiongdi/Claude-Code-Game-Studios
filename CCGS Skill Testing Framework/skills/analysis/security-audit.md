# Skill Test Spec: /security-audit

## Skill Summary

`/security-audit` 审计游戏的安全风险，包括存档数据完整性、网络通信、反作弊暴露和数据隐私。它读取 `src/` 中的源文件查找安全模式，并检查敏感数据是否正确处理。不触发任何 director gate。Skill 不写入文件（仅结果报告）。判定结果：SECURE、CONCERNS 或 VULNERABILITIES FOUND。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：SECURE、CONCERNS、VULNERABILITIES FOUND
- [ ] 不要求 "May I write" 语言（只读；仅结果报告）
- [ ] 具有下一步交接说明（如何处理结果）

---

## Director Gate Checks

无。安全审计是只读建议性技能；不触发任何 gate。

---

## Test Cases

### Case 1: Happy Path — 存档数据加密，无硬编码凭证

**Fixture:**
- `src/core/save_system.gd` 使用 `Crypto` 类在写入前加密存档数据
- 任何 `src/` 文件中无硬编码 API 密钥、密码或凭证
- 客户端可见输出版本号或内部构建 ID

**Input:** `/security-audit`

**Expected behavior:**
1. Skill 扫描 `src/` 查找安全模式：加密使用、硬编码凭证、暴露的内部信息
2. 所有检查通过：存档数据加密、未找到凭证、无暴露的内部信息
3. 结果报告显示所有检查 PASS
4. 判定为 SECURE

**Assertions:**
- [ ] Skill 检查存档数据处理的加密使用
- [ ] Skill 扫描硬编码凭证（API 密钥、密码、令牌）
- [ ] Skill 检查暴露给玩家的版本/构建号
- [ ] 所有检查显示在结果报告中
- [ ] 所有检查通过时判定为 SECURE

---

### Case 2: Vulnerabilities Found — 未加密的存档数据和暴露的版本

**Fixture:**
- `src/core/save_system.gd` 以纯 JSON 写入存档数据（无加密）
- `src/ui/debug_overlay.gd` 包含：`label.text = "Build: " + ProjectSettings.get("application/config/version")`
  （向玩家暴露内部构建版本）

**Input:** `/security-audit`

**Expected behavior:**
1. Skill 扫描 `src/` — 发现 `save_system.gd` 中未加密的存档写入
2. Skill 发现 `debug_overlay.gd` 中暴露的版本字符串
3. 两个结果都标记为 VULNERABILITIES
4. 判定为 VULNERABILITIES FOUND
5. Skill 为每个漏洞提供修复建议

**Assertions:**
- [ ] 未加密存档数据标记为漏洞，附带文件和近似行号
- [ ] 暴露的版本字符串标记为漏洞
- [ ] 每个漏洞给出修复建议
- [ ] 检测到任何漏洞时判定为 VULNERABILITIES FOUND
- [ ] 不写入或修改任何文件

---

### Case 3: Online Features Without Authentication — CONCERNS

**Fixture:**
- `src/networking/lobby.gd` 存在，包含函数：`join_lobby()`、`send_chat()`
- `send_chat()` 前未找到身份验证检查 — 玩家无需验证即可调用
- 游戏具有在线多人功能（从文件存在推断）

**Input:** `/security-audit`

**Expected behavior:**
1. Skill 扫描 `src/networking/` — 检测到在线功能代码
2. Skill 检查网络调用前的身份验证守卫 — 在 `send_chat()` 上未找到
3. 标记："Online feature without authentication check — CONCERNS"
4. 判定为 CONCERNS（非 VULNERABILITIES FOUND，因为这是缺失的控制，非漏洞利用）

**Assertions:**
- [ ] Skill 通过扫描网络源文件检测在线功能
- [ ] 网络操作前缺失的身份验证检查被标记
- [ ] 缺失身份验证守卫判定为 CONCERNS（建议性严重级别）
- [ ] 输出建议在网络调用前添加身份验证

---

### Case 4: Edge Case — 没有源文件可分析

**Fixture:**
- `src/` 目录不存在或完全为空

**Input:** `/security-audit`

**Expected behavior:**
1. Skill 尝试扫描 `src/` — 未找到文件
2. Skill 输出错误："No source files found in `src/` — nothing to audit"
3. 不生成结果报告
4. 不输出判定

**Assertions:**
- [ ] `src/` 为空或不存在时 Skill 不崩溃
- [ ] 输出清楚说明未找到源文件
- [ ] 不输出判定（无内容可评估）
- [ ] Skill 建议验证 `src/` 目录路径

---

### Case 5: Gate Compliance — 无 gate；security-engineer 单独调用

**Fixture:**
- 源文件存在；检测到 1 个 CONCERNS 级别结果（发布构建中启用调试日志）
- `review-mode.txt` 内容为 `full`

**Input:** `/security-audit`

**Expected behavior:**
1. Skill 扫描源；在发布路径中找到活动调试日志
2. 无论审查模式如何，不触发任何 director gate
3. 判定为 CONCERNS
4. 输出注明："For formal security review, consider engaging a security-engineer agent"
5. 结果以只读报告呈现；不写入文件

**Assertions:**
- [ ] 任何审查模式下都不触发 director gate
- [ ] 建议（非强制）咨询 security-engineer
- [ ] 不写入任何文件
- [ ] 建议性级别安全结果判定为 CONCERNS

---

## Protocol Compliance

- [ ] 审计前读取 `src/` 中的源文件
- [ ] 检查存档数据加密、硬编码凭证、暴露的内部信息、身份验证守卫
- [ ] 为每个结果提供修复建议
- [ ] 不写入任何文件（只读技能）
- [ ] 不触发任何 director gate
- [ ] 判定为以下之一：SECURE、CONCERNS、VULNERABILITIES FOUND

---

## Coverage Notes

- 反作弊分析（客户端值验证、服务器权威）此处未显式测试；根据严重级别遵循 CONCERNS 或 VULNERABILITIES 模式。
- 数据隐私合规（GDPR、COPPA）超出此 spec 范围；这些需要法律审查，超出代码扫描范围。
