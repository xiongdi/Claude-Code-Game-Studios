# 示例会话：/gate-check 阶段转换 — 系统设计 → 技术设置

**背景：** 独立开发者，第5周末。系统设计阶段感觉完成——6个 MVP GDD 已写，跨审查完成，引擎已设置。用户想正式推进到技术设置。

**起始状态：**
- `production/stage.txt` — 当前 `systems-design`
- `design/gdd/systems-index.md` — 6个 MVP 系统列出，全部标记 `状态：GDD 完成`
- `design/gdd/` — 6个 GDD 文件存在
- `design/gdd/gdd-cross-review-2026-03-10.md` — 跨审查报告，判定：咨询（1个低优先级备注）
- 引擎已配置（Godot 4.6，`/setup-engine` 完成）

**会话时长：** 约20分钟
**轮次：** 7
**结果：** 门通过，`stage.txt` 更新为 `technical-setup`，清晰的接下来步骤列表

---

## 第1轮：用户发起门检查

**用户：**
> /gate-check

## 第2轮：Agent 检测当前阶段并运行检查

**Agent（通过 /gate-check）：**
> 读取 `production/stage.txt` — 当前阶段：**systems-design**
>
> 运行门：**系统设计 → 技术设置**
>
> ---
> **必需工件检查清单：**
>
> | 工件 | 要求 | 状态 |
> |----------|-------------|--------|
> | `design/gdd/systems-index.md` | 存在，所有 MVP 系统列出 | ✅ |
> | 每系统 GDD | 所有 MVP 系统有 GDD | ✅ (6/6) |
> | 每个 GDD 的 `/design-review` | 每个 GDD 已审查 | ✅ (6/6 审查注释找到) |
> | 跨 GDD 审查报告 | `design/gdd/gdd-cross-review-*.md` 存在 | ✅ |
> | 跨审查判定 | PASS 或 CONCERNS（不是 FAIL） | ✅ CONCERNS |
> | 引擎已配置 | `production/stage.txt` 或 CLAUDE.md 引擎条目 | ✅ Godot 4.6 |
>
> ---
> **内容扫描：**
>
> 检查 GDD 完整性（每个8个必需章节）...
>
> | GDD | 缺失章节 |
> |-----|-----------------|
> | movement-system.md | 无 ✅ |
> | stamina-system.md | 无 ✅ |
> | combat-system.md | 无 ✅ |
> | inventory-system.md | 无 ✅ |
> | crafting-system.md | 无 ✅ |
> | exploration-system.md | 无 ✅ |
>
> ---
> **跨审查咨询（来自 `gdd-cross-review-2026-03-10.md`）：**
>
> 1. ⚠️ 低优先级 — 制作和库存都独立定义"物品堆叠大小"且默认值不同（制作：99，库存：64）。一个系统应该是权威；另一个应该引用它。不阻塞——可在 ADR 中解决。
>
> ---
> **判定：通过**
>
> 所有必需工件存在。跨审查咨询是低严重性，可以在技术设置期间通过 ADR 解决。这是此阶段的正常发现。
>
> **可以将 `production/stage.txt` 推进到 `technical-setup` 吗？**

## 第3轮：用户确认

**用户：**
> 是，推进它。

## 第4轮：Agent 更新阶段并呈现下一步

**Agent：**
> 已更新 `production/stage.txt` → `technical-setup`
>
> ---
> **你现在在：技术设置**
>
> 此阶段在生产编码开始前建立技术基础。你的目标是完整、批准的架构，以便在预生产中写的每个 story 都有清晰、无歧义的技术指导。
>
> **本阶段必需的步骤（按顺序）：**
>
> 1. **`/create-architecture`** — 创作主架构文档。涵盖系统分解、数据流、层边界和模块职责。
>
> 2. **`/architecture-decision`（每个重要技术选择一个）** — 最少：
>    - 渲染方法（2D 相机、视口等）
>    - 保存/加载策略
>    - 音频总线架构
>    - 跨审查中的"物品堆叠大小权威"问题（制作 vs 库存）
>
> 3. **`/architecture-review`** — 验证所有 ADR 的完整性、依赖排序和 GDD 可追溯性。产生 TR-registry。
>
> 4. **`/create-control-manifest`** — 将已批准的 ADR 转换为扁平的程序员规则表。这是实施 agents 在编码期间将 consult 的内容。
>
> 5. **`/gate-check`** — 技术设置 → 预生产门。要求：架构文档、所有 ADR 已批准、控制清单、依赖排序已解决。
>
> **可选但推荐：**
> - 如果你有计划携带的任何原型代码，运行 `/tech-debt` 扫描
> - 如果你还没有获取 Godot 4.6 API 快照，重新运行 `/setup-engine`
>
> **提示：** 从制作和库存的堆叠大小冲突的 `/architecture-decision` 开始。这是一个快速胜利，在它在未来两个 ADR 中创建歧义之前关闭一个悬而未决的问题。

## 第5轮：用户询问跨审查咨询

**用户：**
> 在栈大小冲突的 ADR 中我实际写什么？

## 第6轮：Agent 建议

**Agent：**
> ADR 决策是：**哪个系统拥有规范的 `max_stack_size` 值？**
>
> 两个选项考虑：
>
> **选项A：库存拥有它**
> - `ItemData` 资源（由库存拥有）获取 `max_stack_size: int = 64`
> - 制作系统从 `ItemData` 读取，从不设置自己的值
> - 优点：库存是所有物品属性的真实来源
> - 缺点：制作团队需要知道查看库存数据
>
> **选项B：物品数据资源拥有它（共享资源）**
> - `ItemData.max_stack_size` 是两个系统的权威
> - 制作和库存都从 `ItemData` 导入
> - 优点：单一来源，制作和库存模块之间无耦合
> - 缺点：共享数据层中又多了一件事
>
> 大多数工作室在有>2个系统接触物品数据后选择 B（共享资源）。但对于 solo 项目任一都有效。
>
> 运行 `/architecture-decision`，它将指导你完成 ADR 格式，包括将这个 ADR 链接回两个 GDD 的依赖章节。

## 第7轮：用户结束

**用户：**
> 明白了。我将从这个的 `/architecture-decision` 开始。

**Agent：**
> 好的起点。ADR 将文档化决策、推理和它解决的 GDD 要求——所以当你在几个月后写代码时，理由被保留。
>
> 你的阶段门已记录。在技术设置期间的任何时候运行 `/help` 以查看你相对于必需检查清单的当前进度。

---

## 此示例演示什么

- **门检查是自动化的，不是手动的**：agent 读取工件并检查——用户不需要填写表单
- **CONCERNS ≠ FAIL**：跨审查咨询是低严重性并通过门。FAIL 需要在推进前解决。
- **Stage.txt 是权威**：状态行、`/help` 和所有技能从 `production/stage.txt` 读取——在这里更新改变每个后续技能看到的内容
- **下一步是阶段特定的**：agent 不给通用的"祝你好运"——它给出技术设置的具体有序检查清单
- **门检查浮现携带工作**：栈大小冲突是跨审查注释；门检查确保它成为一个具体的 ADR 而不是丢失
- **每个门一次推进**：用户明确确认推进。门不自动推进；需要人类确认。
