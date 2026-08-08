# 技能测试规格: /asset-spec

## 技能摘要

`/asset-spec` 从设计需求生成每个资产的视觉规格文档。它读取相关的 GDD、art bible 和设计系统，生成一个结构化的资产规格表，定义：尺寸、动画状态（如适用）、调色板参考、风格说明、技术约束（格式、文件大小预算）和交付清单。

规格表在 "May I write" 询问后写入 `assets/specs/[asset-name]-spec.md`。如果规格已存在，该技能会提供更新。当在单次调用中请求多个资产时，每个资产都会进行 "May I write" 询问。不适用 director gate。当所有请求的规格都写入后裁决为 COMPLETE。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具有必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 具有 ≥2 个 phase 标题
- [ ] 包含裁决关键词：COMPLETE
- [ ] 包含 "May I write" 协作协议语言（每个资产）
- [ ] 具有下一步交接（例如，分配给艺术家，或之后的 `/asset-audit`）

---

## 导演门控检查

无。`/asset-spec` 是一个设计文档工具。技术艺术家可能单独审查规格，但这不是此技能中的 gate。

---

## 测试用例

### Case 1: Happy Path — Enemy sprite spec with full GDD and art bible

**Fixture:**
- `design/gdd/enemies.md` 存在并定义了敌人变体
- `design/art-bible.md` 存在并包含调色板和风格说明
- 不存在 "goblin-enemy" 的现有资产规格

**Input:** `/asset-spec goblin-enemy`

**Expected behavior:**
1. Skill 读取 enemies GDD 和 art bible
2. Skill 生成哥布林敌人精灵的规格：
   - 尺寸：从引擎默认值推断或明确从 GDD 获取
   - 动画状态：idle、walk、attack、hurt、death
   - 调色板参考：链接到 art-bible 调色板部分
   - 风格说明：来自 art bible 角色设计规则
   - 技术约束：格式（PNG）、大小预算
   - 交付清单
3. Skill 询问 "May I write to `assets/specs/goblin-enemy-spec.md`?"
4. 批准后写入文件；裁决为 COMPLETE

**Assertions:**
- [ ] 所有 6 个规格组件都存在（尺寸、动画、调色板、风格、技术、清单）
- [ ] 调色板参考链接到 art bible（不是复制）
- [ ] 动画状态来自 GDD（不是编造的）
- [ ] "May I write" 使用正确的路径
- [ ] 裁决为 COMPLETE

---

### Case 2: No Art Bible Found — Spec with Placeholder Style Notes, Dependency Flagged

**Fixture:**
- `design/gdd/player.md` 存在
- `design/art-bible.md` 不存在

**Input:** `/asset-spec player-sprite`

**Expected behavior:**
1. Skill 读取 player GDD 但找不到 art bible
2. Skill 生成带有占位符风格说明的规格："DEPENDENCY GAP: art bible
   not found — style notes are placeholders"
3. 调色板部分使用："TBD — see art bible when created"
4. Skill 询问 "May I write to `assets/specs/player-sprite-spec.md`?"
5. 文件写入并带有占位符和依赖标记；裁决为 COMPLETE 并附带建议

**Assertions:**
- [ ] 缺失的 art bible 被标记为 DEPENDENCY GAP
- [ ] 规格仍被生成（未阻塞）
- [ ] 风格说明包含占位符标记，不是编造的风格
- [ ] 裁决为 COMPLETE 并附带建议说明

---

### Case 3: Asset Spec Already Exists — Offers to Update

**Fixture:**
- `assets/specs/goblin-enemy-spec.md` 已存在
- 自规格写入以来 GDD 已更新（添加了新攻击动画）

**Input:** `/asset-spec goblin-enemy`

**Expected behavior:**
1. Skill 检测到现有规格文件
2. Skill 报告："Asset spec already exists for goblin-enemy — checking for updates"
3. Skill 对 GDD 与现有规格进行差异比较并识别：GDD 中添加了新的 "charge-attack"
   动画状态但规格中没有
4. Skill 展示差异："1 new animation state found — offering to update spec"
5. Skill 询问 "May I update `assets/specs/goblin-enemy-spec.md`?"（不是覆盖）
6. 规格已更新；裁决为 COMPLETE

**Assertions:**
- [ ] 检测到现有规格并提供 "update" 路径
- [ ] 显示 GDD 与现有规格之间的差异
- [ ] 使用 "May I update" 语言（不是 "May I write"）
- [ ] 保留现有规格内容；仅应用差异
- [ ] 裁决为 COMPLETE

---

### Case 4: Multiple Assets Requested — May-I-Write Per Asset

**Fixture:**
- GDD 和 art bible 存在
- 用户请求 3 个资产的规格：goblin-enemy、orc-enemy、treasure-chest

**Input:** `/asset-spec goblin-enemy orc-enemy treasure-chest`

**Expected behavior:**
1. Skill 按顺序生成所有 3 个规格
2. 对于每个资产，skill 展示草稿并单独询问 "May I write to
   `assets/specs/[name]-spec.md`?"
3. 用户可以批准全部 3 个或跳过单个资产
4. 写入所有批准的规格；裁决为 COMPLETE

**Assertions:**
- [ ] "May I write" 被询问 3 次（每个资产一次），不是一次性全部
- [ ] 用户可以拒绝一个资产而不阻塞其他资产
- [ ] 为批准的资产写入所有 3 个规格文件
- [ ] 当所有批准的规格都写入后裁决为 COMPLETE

---

### Case 5: Director Gate Check — No gate; asset-spec is a design utility

**Fixture:**
- GDD 和 art bible 存在

**Input:** `/asset-spec goblin-enemy`

**Expected behavior:**
1. Skill 生成并写入资产规格
2. 不派生 director agent
3. 输出中不出现 gate ID

**Assertions:**
- [ ] 未调用 director gate
- [ ] 不出现 gate 跳过消息
- [ ] 无需任何 gate 检查即可达到 COMPLETE

---

## 协议合规

- [ ] 在生成规格前读取 GDD、art bible 和设计系统
- [ ] 包含所有 6 个规格组件（尺寸、动画、调色板、风格、技术、清单）
- [ ] 用 DEPENDENCY GAP 注释标记缺失的依赖（art bible、GDD）
- [ ] 每个资产询问 "May I write"（或 "May I update"）
- [ ] 处理多个资产时使用单独的写入确认
- [ ] 当所有批准的规格都写入后裁决为 COMPLETE

---

## 覆盖说明

- 音频资产规格（音效、音乐）遵循相同的结构，但使用不同的字段（持续时间、采样率、循环），未单独测试。
- UI 资产规格（图标、按钮状态）遵循相同的流程，交互状态需求与 UX spec 对齐。
- GDD 也缺失（GDD 和 art bible 都不存在）的情况未单独测试；规格将生成并标记两个依赖差距。
