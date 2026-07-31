# Skill Test Spec: /soak-test

## Skill Summary

`/soak-test` 生成一个结构化的 soak 测试协议——一个扩展运行时测试计划，旨在暴露内存泄漏、性能漂移和仅在持续游戏过程中才会出现的稳定性问题。该技能生成一个文档，指定测试持续时间、被测系统、监控检查点（例如，每 30 分钟内存采样）、通过/失败阈值以及提前终止条件。

该技能在持久化前会询问 "May I write to `production/qa/soak-[slug]-[date].md`?"。如果发现同一系统的先前 soak 测试，该技能会提出延长时间或添加新条件。不适用 director gate。当 soak 测试协议写入后裁决为 COMPLETE。

---

## Static Assertions (Structural)

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：COMPLETE
- [ ] 在写入协议前包含 "May I write" 协作协议语言
- [ ] 具有下一步交接（例如，`/regression-suite` 或 `/release-checklist`）

---

## Director Gate Checks

无。`/soak-test` 是一个 QA 计划工具。不适用 director gate。

---

## Test Cases

### Case 1: Happy Path — Online gameplay feature, 2-hour soak protocol

**Fixture:**
- 用户指定：system = "online multiplayer lobby"，duration = "2 hours"
- `technical-preferences.md` 已配置引擎

**Input:** `/soak-test online-lobby 2h`

**Expected behavior:**
1. Skill 为在线大厅系统生成 2 小时 soak 测试协议
2. 协议包括：每 30 分钟的监控检查点、要跟踪的指标（内存使用量、连接数、丢包率）、通过阈值、提前终止条件（崩溃或内存增长 >20%）
3. 包含网络专用检查（会话掉率、重连处理）
4. Skill 询问 "May I write to `production/qa/soak-online-lobby-2026-04-06.md`?"
5. 批准后写入文件；裁决为 COMPLETE

**Assertions:**
- [ ] 协议持续时间与请求的 2 小时匹配
- [ ] 监控检查点在合理间隔（例如，每 30 分钟）
- [ ] 包含网络专用检查（不仅是通用内存检查）
- [ ] "May I write" 使用正确的文件路径
- [ ] 裁决为 COMPLETE

---

### Case 2: No Target Defined — Prompts for system, duration, and conditions

**Fixture:**
- 未提供参数
- 会话状态中没有 soak 测试配置

**Input:** `/soak-test`

**Expected behavior:**
1. Skill 检测到未指定目标系统或持续时间
2. Skill 询问："What system or feature should be soak-tested?"
3. 用户回答系统后：Skill 询问："What duration? (e.g., 1h, 4h, 8h)"
4. 用户回答持续时间后：Skill 询问特定条件或使用默认值（正常游戏循环、默认玩家数量）
5. Skill 从收集的输入生成协议并询问 "May I write"

**Assertions:**
- [ ] 至少询问 2 个后续问题（系统 + 持续时间）
- [ ] 当用户未指定自定义条件时应用默认条件
- [ ] 在了解系统和持续时间之前不生成协议
- [ ] 文件写入后裁决为 COMPLETE

---

### Case 3: Previous Soak Test Exists — Offers to extend or add conditions

**Fixture:**
- `production/qa/soak-online-lobby-2026-03-15.md` 存在并包含 1 小时协议
- 用户希望延长到 4 小时并添加新的内存阈值条件

**Input:** `/soak-test online-lobby 4h`

**Expected behavior:**
1. Skill 找到 online-lobby 的现有 soak 测试
2. Skill 报告："Previous soak test found: soak-online-lobby-2026-03-15.md (1h)"
3. Skill 展示选项：创建新协议（4 小时独立），或将现有协议延长到 4 小时并添加新条件
4. 用户选择扩展；保留现有检查点，添加新检查点
5. Skill 询问 "May I write to `production/qa/soak-online-lobby-2026-04-06.md`?"
   （新文件，不覆盖旧文件）

**Assertions:**
- [ ] 呈现并引用现有 soak 测试
- [ ] 用户提供扩展 vs. 新建选项
- [ ] 创建新文件（不覆盖旧文件）
- [ ] 扩展协议包括新旧检查点
- [ ] 裁决为 COMPLETE

---

### Case 4: Mobile Target Platform — Memory-specific checkpoints added

**Fixture:**
- `technical-preferences.md` 指定目标平台：Mobile
- 用户请求对 "gameplay session" 进行 30 分钟的 soak 测试

**Input:** `/soak-test gameplay 30m`

**Expected behavior:**
1. Skill 读取 `technical-preferences.md` 并检测到移动目标平台
2. Soak 测试协议包括移动专用内存检查点：
   - 检查堆内存增长与设备基线的对比
   - 在检查点间隔检查纹理内存
   - 在 300MB 处添加警告阈值（移动上限）
3. 协议还包括热/电池消耗建议说明
4. Skill 询问 "May I write?" 并在批准后写入；裁决为 COMPLETE

**Assertions:**
- [ ] 从 technical-preferences.md 检测到移动平台
- [ ] 内存检查点包括适合移动的阈值（不是桌面）
- [ ] 协议中存在热/电池说明
- [ ] 裁决为 COMPLETE

---

### Case 5: Director Gate Check — No gate; soak-test is a planning utility

**Fixture:**
- 提供了有效的系统和持续时间

**Input:** `/soak-test combat 1h`

**Expected behavior:**
1. Skill 生成并写入 soak 测试协议
2. 不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] Skill 无需任何 gate 检查即可达到 COMPLETE

---

## Protocol Compliance

- [ ] 在生成协议前收集系统、持续时间和条件
- [ ] 包括定期间隔的监控检查点
- [ ] 包括通过/失败阈值和提前终止条件
- [ ] 根据目标平台调整检查点（移动 vs. 桌面）
- [ ] 在创建协议文件前询问 "May I write"
- [ ] 文件写入后裁决为 COMPLETE

---

## Coverage Notes

- 针对特定引擎子系统（渲染管线、物理模拟）的 soak 测试遵循相同的协议结构，未单独测试。
- 用户提供短于最短有用 soak 周期（例如，5 分钟）的持续时间的情况未测试；skill 会注明这对于有意义的结果来说太短了。
- Soak 测试协议的自动执行超出了此 skill 的范围——此 skill 生成计划，而不是运行器。
