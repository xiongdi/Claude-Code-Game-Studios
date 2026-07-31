# Skill Test Spec: /asset-audit

## Skill Summary

`/asset-audit` 审计 `assets/` 目录的命名规范合规性、缺失元数据和格式/大小问题。它根据 `technical-preferences.md` 中定义的规范和预算读取资源文件。不触发任何 director gate。Skill 未经用户批准不写入。判定结果：COMPLIANT、WARNINGS 或 NON-COMPLIANT。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：COMPLIANT、WARNINGS、NON-COMPLIANT
- [ ] 不要求 "May I write" 语言（只读；可选报告需批准）
- [ ] 具有下一步交接说明（审计结果后如何处理）

---

## Director Gate Checks

无。资源审计是只读分析技能；不触发任何 gate。

---

## Test Cases

### Case 1: Happy Path — 所有资源遵循命名规范

**Fixture:**
- `technical-preferences.md` 指定命名规范：`snake_case`，如 `enemy_grunt_idle.png`
- `assets/art/characters/` 包含：`enemy_grunt_idle.png`、`enemy_sniper_run.png`
- `assets/audio/sfx/` 包含：`sfx_jump_land.ogg`、`sfx_item_pickup.ogg`
- 所有文件都在大小预算内（纹理 ≤2MB，音频 ≤500KB）

**Input:** `/asset-audit`

**Expected behavior:**
1. Skill 从 `technical-preferences.md` 读取命名规范和大小预算
2. Skill 递归扫描 `assets/`
3. 所有文件匹配 `snake_case` 规范；都在预算内
4. 审计表显示所有行 PASS
5. 判定为 COMPLIANT

**Assertions:**
- [ ] 审计涵盖美术和资源音频目录
- [ ] 每个文件检查命名规范和大小预算
- [ ] 合规时所有行显示 PASS
- [ ] 判定为 COMPLIANT
- [ ] 不写入任何文件

---

### Case 2: Non-Compliant — 纹理超出大小预算

**Fixture:**
- `assets/art/environment/` 包含 5 个纹理文件
- 3 个纹理文件各为 4MB（预算：≤2MB）
- 2 个纹理文件在预算内

**Input:** `/asset-audit`

**Expected behavior:**
1. Skill 从 `technical-preferences.md` 读取大小预算（纹理 2MB）
2. Skill 扫描 `assets/art/environment/` — 发现 3 个超大纹理
3. 审计表列出每个超大文件的实际大小和预算
4. 判定为 NON-COMPLIANT
5. Skill 建议对标记文件进行压缩或降低分辨率

**Assertions:**
- [ ] 所有 3 个超大文件按名称列出，附带实际大小和预算大小
- [ ] 任何文件超出预算时判定为 NON-COMPLIANT
- [ ] 为超大文件提供优化建议
- [ ] 预算内的文件也列出（显示 PASS）以保证完整性

---

### Case 3: Format Issue — 音频格式错误

**Fixture:**
- `technical-preferences.md` 指定音频格式：OGG
- `assets/audio/music/theme_main.wav` 存在（WAV 格式）
- `assets/audio/sfx/sfx_footstep.ogg` 存在（正确的 OGG 格式）

**Input:** `/asset-audit`

**Expected behavior:**
1. Skill 读取音频格式要求：OGG
2. Skill 扫描 `assets/audio/` — 发现 `theme_main.wav` 格式错误
3. 审计表将 `theme_main.wav` 标记为 FORMAT ISSUE（期望 OGG，实际 WAV）
4. `sfx_footstep.ogg` 显示 PASS
5. 判定为 WARNINGS（格式问题可纠正）

**Assertions:**
- [ ] `theme_main.wav` 标记为 FORMAT ISSUE，注明期望和实际格式
- [ ] 格式问题判定为 WARNINGS（非 NON-COMPLIANT），因为可纠正
- [ ] 正确格式的资源显示为 PASS
- [ ] Skill 不修改或转换任何资源文件

---

### Case 4: Missing Asset — GDD 引用但 assets/ 中不存在的资源

**Fixture:**
- `design/gdd/enemies.md` 引用 `enemy_boss_idle.png`
- `assets/art/characters/boss/` 目录为空 — 文件不存在

**Input:** `/asset-audit`

**Expected behavior:**
1. Skill 读取 GDD 引用以找到期望的资源（与 `/content-audit` 范围交叉引用）
2. Skill 扫描 `assets/art/characters/boss/` — 文件未找到
3. 审计表将 `enemy_boss_idle.png` 标记为 MISSING ASSET
4. 判定为 NON-COMPLIANT（缺少关键美术资源）

**Assertions:**
- [ ] Skill 检查 GDD 引用以识别期望的资源
- [ ] 缺失资源标记为 MISSING ASSET，注明 GDD 引用
- [ ] 关键资源缺失时判定为 NON-COMPLIANT
- [ ] Skill 不创建或添加占位资源

---

### Case 5: Gate Compliance — 无 gate；technical-artist 可单独咨询

**Fixture:**
- 2 个文件有命名规范违规（CamelCase 而非 snake_case）
- `review-mode.txt` 内容为 `full`

**Input:** `/asset-audit`

**Expected behavior:**
1. Skill 扫描资源，发现 2 个命名违规
2. 无论审查模式如何，不触发任何 director gate
3. 判定为 WARNINGS
4. 输出注明："Consider having a Technical Artist review naming conventions"
5. Skill 呈现结果；提供可选审计报告写入
6. 如果用户选择："May I write to `production/qa/asset-audit-[date].md`?"

**Assertions:**
- [ ] 任何审查模式下都不触发 director gate
- [ ] 建议（非强制）咨询 technical artist
- [ ] 结果表在任何写入提示之前呈现
- [ ] 可选审计报告写入在写入前询问 "May I write"

---

## Protocol Compliance

- [ ] 从 `technical-preferences.md` 读取命名规范、格式和大小预算
- [ ] 递归扫描 `assets/` 目录
- [ ] 审计表显示文件名、检查类型、期望值、实际值和结果
- [ ] 不修改任何资源文件
- [ ] 不触发任何 director gate
- [ ] 判定为以下之一：COMPLIANT、WARNINGS、NON-COMPLIANT

---

## Coverage Notes

- 元数据检查（如 Godot `.import` 文件中缺失的纹理导入设置）此处未显式测试；它们遵循相同的 FORMAT ISSUE 标记模式。
- `/asset-audit` 和 `/content-audit` 之间的交互（两者都检查 GDD 引用 vs. 资源）是有意的重叠；`/asset-audit` 关注合规性，而 `/content-audit` 关注完整性。
