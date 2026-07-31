# Skill Test Spec: /launch-checklist

## Skill Summary

`/launch-checklist` 生成并评估一个完整的发布准备清单，涵盖：法律合规（EULA、隐私政策、ESRB/PEGI 评级）、平台认证状态、商店页面完整性（截图、描述、元数据）、构建验证（版本标签、可重现构建）、分析和崩溃报告配置以及首次运行体验验证。

该技能在 "May I write" 询问后生成写入 `production/launch/launch-checklist-[date].md` 的清单报告。如果存在先前的发布清单，它会将新结果与旧的进行比较，以突出新解决和新阻塞的项目。不适用 director gate——`/team-release` 编排完整的发布管道。裁决：LAUNCH READY、LAUNCH BLOCKED 或 CONCERNS。

---

## Static Assertions (Structural)

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：LAUNCH READY、LAUNCH BLOCKED、CONCERNS
- [ ] 在写入清单前包含 "May I write" 协作协议语言
- [ ] 具有下一步交接（例如，`/team-release` 或 `/day-one-patch`）

---

## Director Gate Checks

无。`/launch-checklist` 是一个准备状态审计工具。完整的发布管道由 `/team-release` 管理。

---

## Test Cases

### Case 1: Happy Path — All Checklist Items Verified, LAUNCH READY

**Fixture:**
- 法律文档存在：`production/legal/` 中有 EULA、隐私政策
- 平台认证：在生产笔记中标记为已提交和已批准
- 商店页面资源：`production/store/` 中有截图、描述、元数据
- 构建：存在版本标签 `v1.0.0`，已确认可重现构建
- 崩溃报告：在 `technical-preferences.md` 中配置

**Input:** `/launch-checklist`

**Expected behavior:**
1. Skill 检查所有清单类别
2. 所有项目通过其验证检查
3. Skill 生成所有项目标记为 PASS 的清单报告
4. Skill 询问 "May I write to `production/launch/launch-checklist-2026-04-06.md`?"
5. 批准后写入报告；裁决为 LAUNCH READY

**Assertions:**
- [ ] 检查所有清单类别（法律、平台、商店、构建、分析、UX）
- [ ] 所有项目在报告中出现并带有 PASS 标记
- [ ] 裁决为 LAUNCH READY
- [ ] "May I write" 使用正确的日期文件名

---

### Case 2: Platform Certification Not Submitted — LAUNCH BLOCKED

**Fixture:**
- 所有其他清单项目通过
- 平台认证部分："not submitted"（未找到提交记录）

**Input:** `/launch-checklist`

**Expected behavior:**
1. Skill 检查所有项目
2. 平台认证检查失败：无提交记录
3. Skill 报告："LAUNCH BLOCKED — Platform certification not submitted"
4. 命名缺少认证的具体平台
5. 裁决为 LAUNCH BLOCKED

**Assertions:**
- [ ] 裁决为 LAUNCH BLOCKED（不是 CONCERNS）
- [ ] 平台认证被识别为阻塞项目
- [ ] 指定了缺失的平台名称
- [ ] 报告中仍显示所有其他通过的项目

---

### Case 3: Manual Check Required — CONCERNS Verdict

**Fixture:**
- 所有关键清单项目通过
- 首次运行体验项目："MANUAL CHECK NEEDED — human must play the first 5
  minutes and verify tutorial completion flow"
- 商店截图项目："MANUAL CHECK NEEDED — art team must verify screenshot
  quality matches current build"

**Input:** `/launch-checklist`

**Expected behavior:**
1. Skill 检查所有项目
2. 2 个项目被标记为需要人工验证
3. Skill 报告："CONCERNS — 2 items require manual verification before launch"
4. 两个项目都列出并附带手动验证说明
5. 裁决为 CONCERNS（不是 LAUNCH BLOCKED，因为这些是建议性的）

**Assertions:**
- [ ] 裁决为 CONCERNS（不是 LAUNCH READY 或 LAUNCH BLOCKED）
- [ ] 两个手动检查项目都列出并附带验证说明
- [ ] Skill 不会因 MANUAL CHECK 项目自动阻塞

---

### Case 4: Previous Checklist Exists — Delta Comparison

**Fixture:**
- `production/launch/launch-checklist-2026-03-25.md` 存在并包含先前结果：
  - 2 个项目为 BLOCKED（平台认证、崩溃报告）
  - 1 个项目有 MANUAL CHECK
- 新清单：平台认证现在为 PASS，崩溃报告现在为 PASS，
  手动检查仍开放；1 个项目新标记（EULA 最后更新日期）

**Input:** `/launch-checklist`

**Expected behavior:**
1. Skill 找到先前的清单并加载以进行比较
2. Skill 生成新清单并进行比较：
   - 新解决："Platform cert — was BLOCKED, now PASS"
   - 新解决："Crash reporting — was BLOCKED, now PASS"
   - 仍开放：手动检查（未改变）
   - 新问题：EULA 最后更新日期（不在先前清单中）
3. 报告中突出显示差异
4. 裁决为 CONCERNS（手动检查 + 新 EULA 问题）

**Assertions:**
- [ ] 差异部分显示新解决的项目
- [ ] 差异部分显示新问题（不在先前清单中）
- [ ] 先前清单中仍开放的项目被注明为持续存在
- [ ] 裁决反映当前状态（不是先前状态）

---

### Case 5: Director Gate Check — No gate; launch-checklist is an audit utility

**Fixture:**
- 所有清单依赖项存在

**Input:** `/launch-checklist`

**Expected behavior:**
1. Skill 运行完整清单并写入报告
2. 不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 裁决为 LAUNCH READY、LAUNCH BLOCKED 或 CONCERNS——无 gate 裁决

---

## Protocol Compliance

- [ ] 检查所有必需类别（法律、平台、商店、构建、分析、UX）
- [ ] 对硬失败（未完成的认证、缺失的法律文档）使用 LAUNCH BLOCKED
- [ ] 对需要手动验证的建议性项目使用 CONCERNS
- [ ] 存在先前清单时与之比较
- [ ] 在创建清单报告前询问 "May I write"
- [ ] 裁决为 LAUNCH READY、LAUNCH BLOCKED 或 CONCERNS

---

## Coverage Notes

- 特定区域合规（GDPR 数据处理、COPPA 针对 13 岁以下受众）会被检查，但具体需求未在测试断言中枚举。
- 商店页面完整性检查（截图、描述）依赖于 `production/store/` 中文件的存在；它无法验证视觉质量。
- 构建可重现性检查验证版本标签和构建配置的存在，但不执行构建过程。
