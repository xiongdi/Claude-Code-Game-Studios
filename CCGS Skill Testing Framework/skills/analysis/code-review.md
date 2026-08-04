# 技能测试规格: /code-review

## 技能摘要

`/code-review` 对 `src/` 中的源文件执行架构代码审查，检查 `CLAUDE.md` 中的编码标准（公共 API 的文档注释、依赖注入优于单例、数据驱动的值、可测试性）。审查结果为建议性（advisory）。不触发任何 director gate。不修改任何代码。判定结果：APPROVED、CONCERNS 或 NEEDS CHANGES。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：APPROVED、CONCERNS、NEEDS CHANGES
- [ ] 不要求 "May I write" 语言（只读；结果为建议性输出）
- [ ] 具有下一步交接说明（如何处理审查结果）

---

## 导演门控检查

无。代码审查是只读的建议性技能；不触发任何 gate。

---

## 测试用例

### Case 1: Happy Path — 源文件遵循所有编码标准

**Fixture:**
- `src/gameplay/health_component.gd` 存在，包含：
  - 所有公共方法都有文档注释（`##` 标记）
  - 未使用单例；依赖通过构造函数注入
  - 无硬编码值；所有常量引用 `assets/data/`
  - 文件头部有 ADR 引用：`# ADR: docs/architecture/adr-004-health.md`
  - 引用的 ADR 状态为 `Status: Accepted`

**Input:** `/code-review src/gameplay/health_component.gd`

**Expected behavior:**
1. Skill 读取源文件
2. Skill 检查所有编码标准：文档注释、DI、数据驱动、ADR 状态
3. 所有检查通过
4. Skill 输出结果摘要，所有检查显示 PASS
5. 判定为 APPROVED

**Assertions:**
- [ ] 输出中列出每项编码标准检查
- [ ] 标准满足时所有检查显示 PASS
- [ ] Skill 读取引用的 ADR 以确认其状态
- [ ] 判定为 APPROVED
- [ ] 未对任何文件进行修改

---

### Case 2: Needs Changes — 缺少文档注释和使用单例

**Fixture:**
- `src/ui/inventory_ui.gd` 有：
  - 2 个公共方法缺少文档注释
  - 使用 `GameManager.instance`（单例模式）
  - 其他标准均已满足

**Input:** `/code-review src/ui/inventory_ui.gd`

**Expected behavior:**
1. Skill 读取源文件
2. Skill 检测到：2 个公共方法缺少文档注释
3. Skill 检测到：特定行使用单例（如第 42 行、第 87 行）
4. 结果列出具体的方法名和行号
5. 判定为 NEEDS CHANGES

**Assertions:**
- [ ] 缺少的文档注释列出方法名
- [ ] 单例使用标注了文件和行号
- [ ] 存在 BLOCKING 级别的标准违规时判定为 NEEDS CHANGES
- [ ] Skill 不修改文件 — 结果供开发者处理
- [ ] 输出建议用依赖注入替换单例

---

### Case 3: Architecture Risk — ADR 引用为 Proposed 而非 Accepted

**Fixture:**
- `src/core/save_system.gd` 有头部注释：`# ADR: docs/architecture/adr-010-save.md`
- `adr-010-save.md` 存在但状态为 `Status: Proposed`
- 代码本身遵循所有其他编码标准

**Input:** `/code-review src/core/save_system.gd`

**Expected behavior:**
1. Skill 读取源文件
2. Skill 读取引用的 ADR — 发现状态为 `Status: Proposed`
3. Skill 将其标记为 ARCHITECTURE RISK（代码正在实现未接受的 ADR）
4. 其他编码标准检查通过
5. 判定为 CONCERNS（风险标记为建议性，非硬性 NEEDS CHANGES）

**Assertions:**
- [ ] Skill 读取引用的 ADR 文件以检查其状态
- [ ] ADR 状态为 Proposed 时标记 ARCHITECTURE RISK
- [ ] ADR 风险判定为 CONCERNS（非 NEEDS CHANGES）— 建议性严重级别
- [ ] 输出建议在代码进入生产环境前先解决 ADR

---

### Case 4: Edge Case — 指定路径下未找到源文件

**Fixture:**
- 用户调用 `/code-review src/networking/`
- `src/networking/` 目录不存在

**Input:** `/code-review src/networking/`

**Expected behavior:**
1. Skill 尝试读取 `src/networking/` 中的文件
2. 目录或文件未找到
3. Skill 输出错误："No source files found at `src/networking/`"
4. Skill 建议检查 `src/` 中的有效目录
5. 不输出任何判定（没有审查内容）

**Assertions:**
- [ ] 路径不存在时 Skill 不会崩溃
- [ ] 错误消息中命名了尝试的路径
- [ ] 输出建议检查 `src/` 中的有效文件路径
- [ ] 没有审查内容时不输出判定

---

### Case 5: Gate Compliance — 无 gate；LP 可单独咨询

**Fixture:**
- 源文件遵循大部分标准，但有 1 个 CONCERNS 级别的结果（魔法数字）
- `review-mode.txt` 内容为 `full`

**Input:** `/code-review src/gameplay/loot_system.gd`

**Expected behavior:**
1. Skill 读取并审查源文件
2. 不触发任何 director gate（代码审查结果为建议性）
3. Skill 以 CONCERNS 判定呈现结果
4. 输出注明："Consider requesting a Lead Programmer review for architecture concerns"
5. Skill 不自动触发任何 agent

**Assertions:**
- [ ] 任何审查模式下都不触发 director gate
- [ ] 输出中建议（非强制）咨询 LP
- [ ] 不修改任何代码
- [ ] 建议性级别的结果判定为 CONCERNS

---

## Protocol Compliance

- [ ] 审查前读取源文件和编码标准
- [ ] 结果输出中列出每项编码标准检查
- [ ] 不修改任何源文件（只读技能）
- [ ] 不触发任何 director gate
- [ ] 判定为以下之一：APPROVED、CONCERNS、NEEDS CHANGES

---

## 覆盖说明

- 未显式测试目录中所有文件的批量审查行为；假设逐文件应用相同检查并汇总判定。
- 测试覆盖率检查（验证对应测试文件是否存在）是延伸目标，此处不测试；这主要是 `/test-evidence-review` 的领域。
