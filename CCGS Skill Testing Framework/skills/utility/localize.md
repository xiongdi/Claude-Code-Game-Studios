# Skill Test Spec: /localize

## Skill Summary

`/localize` 管理完整的本地化管道：它从源文件中提取所有面向玩家的字符串，管理 `assets/localization/` 中的翻译文件，并验证所有区域设置文件的完整性。对于新语言，它创建一个以所有当前字符串作为键、值为空的区域设置文件骨架。对于现有区域设置文件，它生成一个显示添加、删除和更改键的差异报告。

翻译文件在 "May I write" 询问后写入 `assets/localization/[locale-code].csv`（或引擎适当的格式）。不适用 director gate。裁决：LOCALIZATION COMPLETE（所有区域设置都完整）或 GAPS FOUND（至少一个区域设置缺少字符串键）。

---

## Static Assertions (Structural)

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：LOCALIZATION COMPLETE、GAPS FOUND
- [ ] 在写入区域设置文件前包含 "May I write" 协作协议语言
- [ ] 具有下一步交接（例如，将区域设置骨架发送给翻译人员）

---

## Director Gate Checks

无。`/localize` 是一个管道工具。不适用 director gate。localization lead agent 可能单独审查，但不会在此 skill 中调用。

---

## Test Cases

### Case 1: New Language — String Extraction and Locale Skeleton Created

**Fixture:**
- `src/` 中的源代码包含面向玩家的字符串（UI 文本、教程消息）
- 现有区域设置：`assets/localization/en.csv`
- 不存在法语区域设置

**Input:** `/localize fr`

**Expected behavior:**
1. Skill 从源文件中提取所有面向玩家的字符串
2. Skill 在 `en.csv` 中找到相同字符串作为参考
3. Skill 生成包含所有字符串键和空值的 `fr.csv` 骨架
4. Skill 询问 "May I write to `assets/localization/fr.csv`?"
5. 批准后写入文件；裁决为 GAPS FOUND（文件已创建但值为空）
6. Skill 注明："fr.csv created — send to translator to fill values"

**Assertions:**
- [ ] `en.csv` 中的所有字符串键都存在于 `fr.csv` 中
- [ ] `fr.csv` 中的所有值都为空（不是从英语复制的）
- [ ] 在创建文件前询问 "May I write"
- [ ] 裁决为 GAPS FOUND（文件已创建但未翻译）

---

### Case 2: Existing Locale Diff — Additions, Removals, and Changes Listed

**Fixture:**
- `assets/localization/fr.csv` 存在并包含 20 个已翻译的字符串键
- 源代码已更改：添加了 3 个新字符串，删除了 1 个字符串，2 个字符串的英语源文本已更改

**Input:** `/localize fr`

**Expected behavior:**
1. Skill 从源代码中提取当前字符串
2. Skill 与现有 `fr.csv` 进行差异比较
3. Skill 生成差异报告：
   - 3 个新键（需要翻译——在 fr.csv 中列为空）
   - 1 个已删除键（标记为过时——建议删除）
   - 2 个已更改键（英语源已更改——法语可能需要更新，已标记）
4. Skill 询问 "May I update `assets/localization/fr.csv`?"
5. 文件更新，添加新的空键，标记过时键；裁决为 GAPS FOUND

**Assertions:**
- [ ] 新键在更新文件中显示为空（不是自动翻译）
- [ ] 已删除键被标记为过时（不是静默删除）
- [ ] 已更改的源字符串被标记为供翻译人员审查
- [ ] 裁决为 GAPS FOUND（存在新的空键）

---

### Case 3: String Missing in One Locale — GAPS FOUND With Missing Key List

**Fixture:**
- 存在 3 个区域设置文件：`en.csv`、`fr.csv`、`de.csv`
- `de.csv` 缺少 `en.csv` 和 `fr.csv` 中都存在的 4 个键

**Input:** `/localize`

**Expected behavior:**
1. Skill 读取所有 3 个区域设置文件并交叉引用键
2. `de.csv` 缺少 4 个键
3. Skill 生成 GAPS FOUND 报告，按区域设置列出 4 个缺失键：
   "de.csv missing: [key1], [key2], [key3], [key4]"
4. Skill 提出将缺失键作为空值添加到 `de.csv`
5. 批准后：文件更新；裁决仍为 GAPS FOUND（值仍为空）

**Assertions:**
- [ ] 明确列出缺失键（不仅是计数）
- [ ] 缺失键归属于特定的区域设置文件
- [ ] 裁决为 GAPS FOUND（不是 LOCALIZATION COMPLETE）
- [ ] 缺失键作为空值添加（不是从英语自动翻译）

---

### Case 4: Translation File Has Syntax Error — Error With Line Reference

**Fixture:**
- `assets/localization/fr.csv` 在第 47 行有格式错误的行
  （缺少引号闭合）

**Input:** `/localize fr`

**Expected behavior:**
1. Skill 读取 `fr.csv` 并在第 47 行遇到解析错误
2. Skill 输出："Parse error in fr.csv at line 47: [error detail]"
3. Skill 在错误修复前无法对文件进行差异比较或验证
4. Skill 不会尝试覆盖或自动修复格式错误的文件
5. Skill 建议手动修复文件并重新运行 `/localize`

**Assertions:**
- [ ] 错误消息包含行号（第 47 行）
- [ ] 错误详情描述了解析错误的性质
- [ ] Skill 不会覆盖或修改格式错误的文件
- [ ] 建议手动修复 + 重新运行作为修复方案

---

### Case 5: Director Gate Check — No gate; localization is a pipeline utility

**Fixture:**
- 包含面向玩家字符串的源代码

**Input:** `/localize fr`

**Expected behavior:**
1. Skill 提取字符串并管理区域设置文件
2. 不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 裁决为 LOCALIZATION COMPLETE 或 GAPS FOUND——无 gate 裁决

---

## Protocol Compliance

- [ ] 在对区域设置文件进行操作前从源代码提取字符串
- [ ] 创建所有键为空值的新区域设置文件（不是自动翻译）
- [ ] 将现有区域设置文件与当前源代码字符串进行差异比较
- [ ] 按区域设置和键名标记缺失键
- [ ] 在创建或更新任何区域设置文件前询问 "May I write"
- [ ] 裁决为 LOCALIZATION COMPLETE（所有区域设置完全翻译）或 GAPS FOUND

---

## Coverage Notes

- 仅当所有区域设置文件的所有键都有非空值时才能实现 LOCALIZATION COMPLETE；新语言骨架创建总是导致 GAPS FOUND。
- 引擎特定的区域设置格式（Godot `.translation`、Unity `.po` 文件）由技能正文处理；`.csv` 在测试中用作规范格式。
- 源代码字符串以极高频率变化的情况（新 UI 文本的持续集成）未测试；差异逻辑处理这种情况。
