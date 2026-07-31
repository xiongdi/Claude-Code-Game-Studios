# Skill Test Spec: /hotfix

## Skill Summary

`/hotfix` 管理紧急修复工作流：它从 main 创建 hotfix 分支，对识别的文件应用有针对性的修复，运行 `/smoke-check` 验证修复不会引入回退，并提示用户确认合并回 main。每次代码更改都需要 "May I write to [filepath]?" 询问。Git 操作（分支创建、合并）作为 Bash 命令呈现给用户确认后再执行。

该技能对时间敏感——director 审查是可选的事后步骤，不是阻塞 gate。裁决：HOTFIX COMPLETE（修复已应用，smoke check 通过，已合并）或 HOTFIX BLOCKED（修复引入回退或用户拒绝）。

---

## Static Assertions (Structural)

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：HOTFIX COMPLETE、HOTFIX BLOCKED
- [ ] 对代码更改包含 "May I write" 语言
- [ ] 具有下一步交接（例如，`/bug-report` 以记录问题，或版本升级）

---

## Director Gate Checks

无。Hotfix 对时间要求严格。Director 审查可能作为事后步骤单独进行。此技能中不调用任何 gate。

---

## Test Cases

### Case 1: Happy Path — Critical crash bug fixed, smoke check passes

**Fixture:**
- `main` 分支是干净的
- 在 `src/gameplay/arena.gd` 中识别出 Bug（进入 boss 竞技场时崩溃）
- 用户提供复现步骤

**Input:** `/hotfix`（用户描述崩溃和受影响的文件）

**Expected behavior:**
1. Skill 提议创建 hotfix 分支：`hotfix/boss-arena-crash`
2. 用户确认；显示分支创建的 Bash 命令并确认
3. Skill 识别 `arena.gd` 中的修复位置并起草更改
4. Skill 询问 "May I write to `src/gameplay/arena.gd`?" 并在批准后应用修复
5. Skill 运行 `/smoke-check`——PASS
6. Skill 展示合并命令并询问用户确认合并到 `main`
7. 用户确认；执行合并；裁决为 HOTFIX COMPLETE

**Assertions:**
- [ ] 在任何代码更改之前创建 hotfix 分支
- [ ] 在修改任何源文件之前询问 "May I write"
- [ ] 修复应用后运行 `/smoke-check`
- [ ] 合并需要明确的用户确认（不是自动的）
- [ ] 成功合并后裁决为 HOTFIX COMPLETE

---

### Case 2: Smoke Check Fails — HOTFIX BLOCKED

**Fixture:**
- 修复已应用于 `src/gameplay/arena.gd`
- `/smoke-check` 返回 FAIL："Player health clamping regression detected"

**Input:** `/hotfix`

**Expected behavior:**
1. Skill 应用修复并运行 `/smoke-check`
2. Smoke check 返回 FAIL 并识别具体回退
3. Skill 报告："HOTFIX BLOCKED — smoke check failed: [regression detail]"
4. Skill 展示选项：尝试修订修复、更改，或合并已知回退（用户确认风险）
5. Smoke check 失败时不发生自动合并

**Assertions:**
- [ ] 裁决为 HOTFIX BLOCKED
- [ ] Smoke check 失败逐字显示给用户
- [ ] Smoke check 失败时不自动执行合并
- [ ] 用户获得明确的后续操作选项

---

### Case 3: Fix to Already-Released Build — Version tag noted, patch bump prompted

**Fixture:**
- 最新的 git 标签是 `v1.2.0`
- Hotfix 针对 v1.2.0 发布中的 bug

**Input:** `/hotfix`

**Expected behavior:**
1. Skill 检测到当前 HEAD 是标记的发布（v1.2.0）
2. Skill 注明："Hotfix targeting tagged release v1.2.0"
3. Smoke check 通过后，skill 提示："Should version be bumped to v1.2.1?"
4. 如果用户确认版本升级：skill 询问 "May I write to VERSION or equivalent?"
5. 版本更新和合并后：裁决为 HOTFIX COMPLETE 并注明版本

**Assertions:**
- [ ] 检测版本标签上下文并向用户呈现
- [ ] 合并后建议补丁版本升级（不要求）
- [ ] 版本升级需要自己的 "May I write" 确认
- [ ] 裁决为 HOTFIX COMPLETE

---

### Case 4: No Repro Steps — Skill Asks Before Applying Fix

**Fixture:**
- 用户使用模糊描述调用 `/hotfix`："something is broken on level 3"
- 未提供复现步骤

**Input:** `/hotfix`（模糊描述）

**Expected behavior:**
1. Skill 检测到信息不足以识别修复位置
2. Skill 询问："Please provide reproduction steps and the affected file or system"
3. Skill 在提供复现步骤之前不会创建分支或修改任何文件
4. 用户提供复现步骤后：正常的 hotfix 流程开始

**Assertions:**
- [ ] 没有复现步骤时不创建分支
- [ ] 没有明确识别的修复位置时不进行代码更改
- [ ] 复现步骤请求是具体的（不是通用的 "please provide more info"）
- [ ] 用户提供复现步骤后正常 hotfix 流程恢复

---

### Case 5: Director Gate Check — No gate; hotfixes are time-critical

**Fixture:**
- 已识别出具有复现步骤的关键 bug

**Input:** `/hotfix`

**Expected behavior:**
1. Skill 完成 hotfix 工作流
2. 执行期间不派生 director agent
3. 输出中不出现 gate ID
4. 事后 director 审查（如果需要）是手动后续操作，不在此调用

**Assertions:**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 裁决为 HOTFIX COMPLETE 或 HOTFIX BLOCKED——无 gate 裁决

---

## Protocol Compliance

- [ ] 在进行任何代码更改之前创建 hotfix 分支
- [ ] 在修改任何源文件之前询问 "May I write"
- [ ] 应用修复后运行 `/smoke-check`
- [ ] 合并前需要明确的用户确认
- [ ] Smoke check 失败时 HOTFIX BLOCKED——不自动合并
- [ ] 裁决为 HOTFIX COMPLETE 或 HOTFIX BLOCKED

---

## Coverage Notes

- 一次修复需要修改多个文件的情况遵循相同的每个文件 "May I write" 模式，未单独测试。
- Hotfix 后的步骤（创建 bug 报告、更新 changelog）在交接中建议，但不作为此技能执行的一部分进行测试。
- 合并期间的冲突解决（如果 main 已分叉）未测试；skill 会呈现冲突并要求用户手动解决。
