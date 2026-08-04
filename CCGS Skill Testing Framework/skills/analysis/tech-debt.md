# 技能测试规格: /tech-debt

## 技能摘要

`/tech-debt` 跟踪、分类和优先处理整个代码库的技术债务。它读取 `docs/tech-debt-register.md` 获取现有债务登记，并扫描 `src/` 中的源文件查找内联 `TODO` 和 `FIXME` 注释。它按严重级别合并和排序项目。不触发任何 director gate。Skill 在更新前会询问 "May I write to `docs/tech-debt-register.md`?"。判定结果：REGISTER UPDATED 或 NO NEW DEBT FOUND。

---

## Static Assertions（结构性）

由 `/skill-test static` 自动验证 — 无需 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含判定关键词：REGISTER UPDATED、NO NEW DEBT FOUND
- [ ] 包含 "May I write" 语言（skill 写入债务登记）
- [ ] 具有下一步交接说明（更新登记后如何处理）

---

## 导演门控检查

无。技术债务跟踪是内部代码库分析技能；不触发任何 gate。

---

## 测试用例

### Case 1: Happy Path — 内联 TODO 加现有登记项合并

**Fixture:**
- `docs/tech-debt-register.md` 存在，包含 2 个项目（LOW 和 MEDIUM 严重级别）
- `src/gameplay/combat.gd` 有 2 个 `# TODO` 注释和 1 个 `# FIXME` 注释
- `src/ui/hud.gd` 有 0 个内联债务注释

**Input:** `/tech-debt`

**Expected behavior:**
1. Skill 读取 `docs/tech-debt-register.md` — 找到 2 个现有项目
2. Skill 扫描 `src/` — 找到 3 个内联注释（2 个 TODO、1 个 FIXME）
3. Skill 检查内联注释是否已存在于登记中（去重）
4. Skill 呈现按严重级别排序的合并列表（默认 FIXME 在 TODO 之前）
5. Skill 询问 "May I write to `docs/tech-debt-register.md`?"
6. 用户批准；登记更新；判定 REGISTER UPDATED

**Assertions:**
- [ ] 通过递归扫描 `src/` 找到内联注释
- [ ] 现有登记项目不重复
- [ ] 合并列表按严重级别排序
- [ ] 任何写入前出现 "May I write" 提示
- [ ] 判定为 REGISTER UPDATED

---

### Case 2: Register Doesn't Exist — 提供创建

**Fixture:**
- `docs/tech-debt-register.md` 不存在
- `src/` 包含 4 个内联 TODO/FIXME 注释

**Input:** `/tech-debt`

**Expected behavior:**
1. Skill 尝试读取 `docs/tech-debt-register.md` — 未找到
2. Skill 通知用户："No tech-debt-register.md found"
3. Skill 提供用找到的内联项目创建登记
4. Skill 询问 "May I write to `docs/tech-debt-register.md`?"（创建）
5. 用户批准；登记创建包含 4 个项目；判定 REGISTER UPDATED

**Assertions:**
- [ ] 登记文件不存在时 Skill 不崩溃
- [ ] 向用户提供登记创建（非静默跳过）
- [ ] "May I write" 提示反映文件创建（非更新）
- [ ] 创建后判定为 REGISTER UPDATED

---

### Case 3: Resolved Item Detected — 在登记中标记为已解决

**Fixture:**
- `docs/tech-debt-register.md` 有 3 个项目；一个引用 `src/gameplay/legacy_input.gd`
- `src/gameplay/legacy_input.gd` 已被删除（重构移除）
- 引用的 TODO 注释在源中不再存在

**Input:** `/tech-debt`

**Expected behavior:**
1. Skill 读取登记 — 找到 3 个项目
2. Skill 扫描 `src/` — 未找到项目 2 引用的源位置
3. Skill 将项目 2 标记为 RESOLVED（源已消失）
4. Skill 向用户呈现已解决项目供确认
5. 批准后，登记更新，项目 2 标记为 `Status: Resolved`

**Assertions:**
- [ ] Skill 检查每个登记项目的源引用是否仍然存在
- [ ] 缺失源位置导致项目标记为 RESOLVED
- [ ] 已解决项目写入前用户确认
- [ ] RESOLVED 项目保留在登记中（不删除）以供审计历史

---

### Case 4: Edge Case — CRITICAL 债务项目显著呈现

**Fixture:**
- `src/core/network_sync.gd` 有注释：`# FIXME(CRITICAL): race condition in sync buffer — can corrupt save data`
- `docs/tech-debt-register.md` 存在，包含 5 个较低严重级别项目

**Input:** `/tech-debt`

**Expected behavior:**
1. Skill 扫描源并找到 CRITICAL 标记的 FIXME
2. Skill 在输出顶部呈现 CRITICAL 项目 — 在完整表格之前
3. Skill 要求用户在继续前确认关键项目
4. 确认后，skill 呈现完整债务表格并询问是否写入
5. 登记更新，CRITICAL 项目在顶部；判定 REGISTER UPDATED

**Assertions:**
- [ ] CRITICAL 项目出现在输出顶部，不埋在表格中
- [ ] Skill 在请求写入前呈现 CRITICAL 项目
- [ ] 请求用户确认 CRITICAL 项目
- [ ] CRITICAL 严重级别在写入的登记条目中保留

---

### Case 5: Gate Compliance — 无 gate；登记仅在批准后更新

**Fixture:**
- 内联扫描发现 2 个新 TODO；登记有 3 个现有项目
- `review-mode.txt` 内容为 `full`

**Input:** `/tech-debt`

**Expected behavior:**
1. Skill 扫描源并读取登记；编译合并债务列表
2. 无论审查模式如何，不触发任何 director gate
3. Skill 向用户呈现排序的债务表格
4. Skill 询问 "May I write to `docs/tech-debt-register.md`?"
5. 用户批准；登记更新；判定 REGISTER UPDATED

**Assertions:**
- [ ] 任何审查模式下都不触发 director gate
- [ ] 债务表格在任何写入提示之前呈现
- [ ] 文件更新前出现 "May I write" 提示
- [ ] 仅在用户明确批准时写入

---

## Protocol Compliance

- [ ] 编译前读取 `docs/tech-debt-register.md` 并扫描 `src/`
- [ ] 对现有登记项目去重内联注释
- [ ] 按严重级别排序合并列表
- [ ] 更新登记前始终询问 "May I write"
- [ ] 不触发任何 director gate
- [ ] 判定为 REGISTER UPDATED 或 NO NEW DEBT FOUND

---

## 覆盖说明

- `src/` 为空或不存在的情况未测试；行为对内联扫描遵循 NO NEW DEBT FOUND 路径，但登记项目仍会被读取和呈现。
- 无严重级别标记的 TODO 注释默认视为 LOW 严重级别；此分类细节是实现问题，此处不测试。
