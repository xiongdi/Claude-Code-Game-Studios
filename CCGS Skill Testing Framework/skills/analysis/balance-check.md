# 技能测试规格: /balance-check

## 技能摘要

`/balance-check` 读取平衡数据文件（`assets/data/` 中的 JSON 或 YAML），并根据 `design/gdd/` 中 GDD 定义的设计公式检查每个值。它生成一个结果表，列名为：Value → Formula → Deviation → Severity。不触发任何 director gate（只读分析）。Skill 可选择性地写入平衡报告，但会先询问 "May I write"。判定结果：BALANCED、CONCERNS 或 OUT OF BALANCE。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：BALANCED、CONCERNS、OUT OF BALANCE
- [ ] 包含 "May I write" 语言（可选的报告写入）
- [ ] 具有下一步交接说明（审查结果后如何处理）

---

## 导演门控检查

无。平衡检查是只读分析技能；不触发任何 gate。

---

## 测试用例

### Case 1: Happy Path — 所有平衡值在公式容差范围内

**Fixture:**
- `assets/data/combat-balance.json` 存在，包含 6 个属性值
- `design/gdd/combat-system.md` 包含所有 6 个属性的公式，容差为 ±10%
- 所有 6 个值都在容差范围内

**Input:** `/balance-check`

**Expected behavior:**
1. Skill 读取 `assets/data/` 中的所有平衡数据文件
2. Skill 读取 `design/gdd/` 中的 GDD 公式
3. Skill 计算每个值相对其公式的偏差
4. 所有偏差都在 ±10% 容差范围内
5. Skill 输出结果表，所有行显示 PASS
6. 判定为 BALANCED

**Assertions:**
- [ ] 显示所有检查值的结果表
- [ ] 每行显示：属性名、公式目标值、实际值、偏差百分比
- [ ] 容差范围内所有行显示 PASS 或等效标记
- [ ] 判定为 BALANCED
- [ ] 未经用户批准不写入任何文件

---

### Case 2: Out of Balance — 玩家伤害比公式目标高 40%

**Fixture:**
- `assets/data/combat-balance.json` 有 `player_damage_base: 140`
- `design/gdd/combat-system.md` 公式指定 `player_damage_base = 100`（±10%）
- 其他属性都在容差范围内

**Input:** `/balance-check`

**Expected behavior:**
1. Skill 读取 combat-balance.json 并计算 `player_damage_base` 的偏差
2. 偏差为 +40% — 远超 ±10% 容差
3. Skill 在结果表中将该行标记为 HIGH 严重级别
4. 判定为 OUT OF BALANCE
5. Skill 在表格之前显著呈现 HIGH 严重级别的项

**Assertions:**
- [ ] `player_damage_base` 行显示 +40% 偏差
- [ ] 偏差超过容差 2 倍以上时严重级别为 HIGH
- [ ] 任何属性有 HIGH 严重级别偏差时判定为 OUT OF BALANCE
- [ ] HIGH 严重级别的项被明确指出，而非埋在表格行中

---

### Case 3: No GDD Formulas — 无法验证，给出指导

**Fixture:**
- `assets/data/economy-balance.yaml` 存在，包含 10 个属性值
- `design/gdd/` 中没有 GDD 包含经济属性的公式定义

**Input:** `/balance-check`

**Expected behavior:**
1. Skill 读取平衡数据文件
2. Skill 搜索 GDD 中的公式定义 — 未找到经济属性的公式
3. Skill 输出："Cannot validate economy stats — no formulas defined. Run /design-system first."
4. 不生成经济属性的结果表
5. 判定为 CONCERNS（数据存在但无法验证）

**Assertions:**
- [ ] GDD 中不存在公式时 Skill 不编造公式目标值
- [ ] 输出明确命名缺失的公式来源
- [ ] 输出建议运行 `/design-system` 定义公式
- [ ] 判定为 CONCERNS（非 BALANCED，因为无法验证）

---

### Case 4: Orphan Reference — 平衡文件引用了未定义的属性

**Fixture:**
- `assets/data/combat-balance.json` 包含属性 `legacy_armor_mult: 1.5`
- `design/gdd/combat-system.md` 没有 `legacy_armor_mult` 的公式
- 其他属性都有公式定义并通过验证

**Input:** `/balance-check`

**Expected behavior:**
1. Skill 读取 combat-balance.json 中的所有属性
2. Skill 在任何 GDD 中都找不到 `legacy_armor_mult` 的公式
3. Skill 在结果表中将 `legacy_armor_mult` 标记为 ORPHAN REFERENCE
4. 其他属性正常评估；容差范围内显示 PASS
5. 判定为 CONCERNS（孤立引用阻止完全验证）

**Assertions:**
- [ ] `legacy_armor_mult` 在结果表中显示状态 ORPHAN REFERENCE
- [ ] 孤立引用在表中与公式偏差区分开
- [ ] 发现任何孤立引用时判定为 CONCERNS
- [ ] Skill 不静默跳过孤立属性

---

### Case 5: Gate Compliance — 只读；无 gate；可选报告需批准

**Fixture:**
- 平衡数据和 GDD 公式存在；1 个属性有 CONCERNS 级别偏差（比目标高 15%）
- `review-mode.txt` 内容为 `full`

**Input:** `/balance-check`

**Expected behavior:**
1. Skill 读取数据和 GDD；生成结果表
2. 判定为 CONCERNS（1 个属性略微超出范围）
3. 不触发任何 director gate
4. Skill 向用户呈现结果表
5. Skill 提供写入可选平衡报告
6. 如果用户同意：skill 询问 "May I write to `production/qa/balance-report-[date].md`?"
7. 如果用户拒绝：skill 不写入就结束

**Assertions:**
- [ ] 任何审查模式下都不触发 director gate
- [ ] 呈现结果表时不自动写入任何内容
- [ ] 提供可选报告写入但不强制
- [ ] 仅当用户选择报告时出现 "May I write" 提示

---

## Protocol Compliance

- [ ] 分析前读取平衡数据文件和 GDD 公式
- [ ] 结果表显示 Value、Formula、Deviation 和 Severity 列
- [ ] 未经用户明确批准不写入任何文件
- [ ] 不触发任何 director gate
- [ ] 判定为以下之一：BALANCED、CONCERNS、OUT OF BALANCE

---

## 覆盖说明

- `assets/data/` 完全为空的情况未测试；行为遵循 CONCERNS 模式，附带未找到数据文件的消息。
- 容差阈值（±10%、±20%）是 skill 的实现细节；测试验证偏差被检测和分类，而非精确阈值。
