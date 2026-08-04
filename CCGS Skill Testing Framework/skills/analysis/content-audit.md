# 技能测试规格: /content-audit

## 技能摘要

`/content-audit` 读取 `design/gdd/` 中的 GDD，检查其中指定的所有内容项（敌人、物品、关卡等）是否都在 `assets/` 中有对应。它生成一个缺口表：Content Type → Specified Count → Found Count → Missing Items。不触发任何 director gate。Skill 未经用户批准不写入。判定结果：COMPLETE、GAPS FOUND 或 MISSING CRITICAL CONTENT。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：COMPLETE、GAPS FOUND、MISSING CRITICAL CONTENT
- [ ] 不要求 "May I write" 语言（只读输出；写入为可选报告）
- [ ] 具有下一步交接说明（审查缺口表后如何处理）

---

## 导演门控检查

无。内容审计是只读分析技能；不触发任何 gate。

---

## 测试用例

### Case 1: Happy Path — 所有指定内容都存在

**Fixture:**
- `design/gdd/enemies.md` 指定 4 种敌人类型：Grunt、Sniper、Tank、Boss
- `assets/art/characters/` 包含文件夹：`grunt/`、`sniper/`、`tank/`、`boss/`
- `design/gdd/items.md` 指定 3 种物品类型；全部 3 种在 `assets/data/items/` 中找到

**Input:** `/content-audit`

**Expected behavior:**
1. Skill 读取 `design/gdd/` 中的所有 GDD
2. Skill 扫描 `assets/` 中的每个指定内容项
3. 所有 4 种敌人类型和 3 种物品类型都找到
4. 缺口表显示：所有行的 Found Count = Specified Count，无缺失项
5. 判定为 COMPLETE

**Assertions:**
- [ ] 缺口表涵盖 GDD 中找到的所有内容类型
- [ ] 每行显示 Specified Count 和 Found Count
- [ ] 数量匹配时无缺失项
- [ ] 判定为 COMPLETE
- [ ] 不写入任何文件

---

### Case 2: Gaps Found — 敌人类型在 assets 中缺失

**Fixture:**
- `design/gdd/enemies.md` 指定 3 种敌人类型：Grunt、Sniper、Boss
- `assets/art/characters/` 仅包含：`grunt/`、`sniper/`（Boss 文件夹缺失）

**Input:** `/content-audit`

**Expected behavior:**
1. Skill 读取 GDD — 找到 3 种指定的敌人类型
2. Skill 扫描 `assets/art/characters/` — 只找到 2 种
3. 敌人缺口表行：Specified 3、Found 2、Missing: Boss
4. 判定为 GAPS FOUND

**Assertions:**
- [ ] 缺口表行按名称标识 "Boss" 为缺失项
- [ ] 同时显示 Specified Count (3) 和 Found Count (2)
- [ ] 任何内容项缺失时判定为 GAPS FOUND
- [ ] Skill 不假设资源后续会被添加 — 现在就标记

---

### Case 3: No GDD Content Specs Found — 给出指导

**Fixture:**
- `design/gdd/` 仅包含 `core-loop.md`，无内容清单部分
- 没有其他包含内容规格的 GDD

**Input:** `/content-audit`

**Expected behavior:**
1. Skill 读取所有 GDD — 未找到内容清单部分
2. Skill 输出："No content specifications found in GDDs — run /design-system first to define content lists"
3. 不生成缺口表
4. 判定为 GAPS FOUND（没有规格无法确认完整性）

**Assertions:**
- [ ] 没有 GDD 内容规格时 Skill 不生成缺口表
- [ ] 输出建议运行 `/design-system`
- [ ] 判定反映无法确认完整性

---

### Case 4: Edge Case — 资源格式与目标平台不匹配

**Fixture:**
- `design/gdd/audio.md` 指定音频资源为 OGG 格式
- `assets/audio/sfx/jump.wav` 存在（WAV 格式，非 OGG）
- `assets/audio/sfx/land.ogg` 存在（正确格式）
- `technical-preferences.md` 指定音频格式：OGG

**Input:** `/content-audit`

**Expected behavior:**
1. Skill 读取 GDD 音频规格和技术偏好中的格式要求
2. Skill 找到 `jump.wav` — 存在但格式错误
3. 音频缺口表行：Specified 2、Found 2（按名称），但 `jump.wav` 标记为 FORMAT ISSUE
4. 判定为 GAPS FOUND（格式合规是内容完整性的一部分）

**Assertions:**
- [ ] Skill 在指定格式时根据 GDD 或技术偏好检查资源格式
- [ ] `jump.wav` 标记为 FORMAT ISSUE，注明期望格式 (OGG)
- [ ] 格式问题在缺口表中与缺失内容区分开
- [ ] 存在格式问题时判定为 GAPS FOUND

---

### Case 5: Gate Compliance — 只读；无 gate；缺口表供人工审查

**Fixture:**
- GDD 指定 10 个内容项；9 个在 assets 中找到；1 个缺失
- `review-mode.txt` 内容为 `full`

**Input:** `/content-audit`

**Expected behavior:**
1. Skill 读取 GDD 并扫描资源；生成缺口表
2. 无论审查模式如何，不触发任何 director gate
3. Skill 以只读输出向用户呈现缺口表
4. 判定为 GAPS FOUND
5. Skill 提供写入审计报告但不自动写入

**Assertions:**
- [ ] 任何审查模式下都不触发 director gate
- [ ] 呈现缺口表时不自动写入任何文件
- [ ] 提供可选报告写入但不强制
- [ ] Skill 不修改任何资源文件

---

## Protocol Compliance

- [ ] 生成缺口表前读取 GDD 和资源目录
- [ ] 缺口表显示 Content Type、Specified Count、Found Count、Missing Items
- [ ] 未经用户明确批准不写入文件
- [ ] 不触发任何 director gate
- [ ] 判定为以下之一：COMPLETE、GAPS FOUND、MISSING CRITICAL CONTENT

---

## 覆盖说明

- MISSING CRITICAL CONTENT 判定（对比 GAPS FOUND）在缺失项在 GDD 中被标记为 critical 时触发；此处未显式测试，但遵循相同的检测路径。
- `assets/` 目录不存在的情况未测试；skill 会对所有指定项产生 MISSING CRITICAL CONTENT 判定。
