# Skill 测试规格：/prototype

## Skill 摘要

`/prototype` 管理一个快速原型工作流，用于在投入全面生产实现之前验证游戏机制。
原型在 `prototypes/[mechanic-name]/` 中创建，故意设计为一次性的——编码标准
放宽（不需要 ADR，AC 可以最小化，允许硬编码值）。实现后，该 skill 生成一份发现文档，
总结所学内容并建议下一步行动。

该 skill 在创建文件前询问"可以写入 `prototypes/[name]/` 吗？"。如果原型已存在，
skill 提供扩展、替换或归档选项。不适用 director 关卡。判定结果：PROTOTYPE COMPLETE
（原型已构建且发现已记录）或 PROTOTYPE ABANDONED（机制被发现不可行）。

---

## 静态断言（结构性）

由 `/skill-test static` 自动验证——不需要 fixture。

- [ ] 具备必需的 frontmatter 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- [ ] 有 ≥2 个阶段标题
- [ ] 包含判定关键词：PROTOTYPE COMPLETE、PROTOTYPE ABANDONED
- [ ] 在创建原型文件前包含"可以写入吗"语言
- [ ] 有下一步交接（例如，`/design-system` 正式化，或归档）

---

## Director 关卡检查

无。原型是一次性的验证产物。不适用 director 关卡。

---

## 测试用例

### 用例 1：正常路径——机制概念已原型化，发现已记录

**Fixture：**
- `prototypes/` 目录存在
- 不存在 "grapple-hook" 的原型

**输入：** `/prototype grapple-hook`

**预期行为：**
1. Skill 询问"可以写入 `prototypes/grapple-hook/` 吗？"
2. 批准后：创建 `prototypes/grapple-hook/` 目录和基础
   实现骨架（主场景、玩家控制器扩展）
3. Skill 实现最小的抓钩机制（故意粗糙——无
   打磨，允许硬编码值）
4. Skill 生成 `prototypes/grapple-hook/findings.md`，包含：
   - 测试了什么
   - 什么有效
   - 什么无效
   - 建议（继续 / 放弃 / 修改概念）
5. 判定为 PROTOTYPE COMPLETE

**断言：**
- [ ] 在创建任何文件前询问"可以写入 `prototypes/grapple-hook/` 吗？"
- [ ] 实现隔离在 `prototypes/` 中（不是 `src/`）
- [ ] 创建 `findings.md`，至少包含：tested/worked/didn't-work/recommendation
- [ ] 判定为 PROTOTYPE COMPLETE

---

### 用例 2：原型已存在——提供扩展、替换或归档选项

**Fixture：**
- `prototypes/grapple-hook/` 已存在于先前的原型会话中
- 它包含基础实现和 findings.md

**输入：** `/prototype grapple-hook`

**预期行为：**
1. Skill 检测到现有的 `prototypes/grapple-hook/` 目录
2. Skill 报告："Prototype already exists for grapple-hook"
3. Skill 提供 3 个选项：
   - Extend：向现有原型添加新功能
   - Replace：重新开始（询问"可以替换 `prototypes/grapple-hook/` 吗？"）
   - Archive：移至 `prototypes/archive/grapple-hook/` 并重新开始
4. 用户选择；skill 相应继续

**断言：**
- [ ] 检测到现有原型并报告
- [ ] 恰好提供 3 个选项（extend、replace、archive）
- [ ] Replace 路径包含"可以替换吗"确认
- [ ] Archive 路径移动（不是删除）现有原型

---

### 用例 3：原型验证机制——建议进入生产

**Fixture：**
- 原型实现完成
- 发现：抓钩机制有趣且技术上可行

**输入：** `/prototype grapple-hook`（原型会话完成）

**预期行为：**
1. 原型构建和测试完成后，总结发现
2. findings.md 中的建议："Mechanic validated——recommend proceeding
   to `/design-system` for full specification"
3. Skill 交接消息明确建议 `/design-system grapple-hook`
4. 判定为 PROTOTYPE COMPLETE

**断言：**
- [ ] `findings.md` 包含明确建议
- [ ] 建议引用 `/design-system` 当机制被验证时
- [ ] 交接消息呼应建议
- [ ] 判定为 PROTOTYPE COMPLETE（不是 PROTOTYPE ABANDONED）

---

### 用例 4：原型揭示机制不可行——PROTOTYPE ABANDONED

**Fixture：**
- 为 "procedural-dialogue" 实现了原型
- 测试后：该机制产生不连贯的对话树，
  玩起来令人沮丧

**输入：** `/prototype procedural-dialogue`

**预期行为：**
1. 原型已构建
2. 发现记录失败：输出不连贯、玩家困惑、技术复杂性
3. findings.md 中的建议："Mechanic not viable——abandoning"
4. `findings.md` 记录机制失败的具体原因
5. Skill 在交接中建议替代方案（例如，策划的对话）
6. 判定为 PROTOTYPE ABANDONED

**断言：**
- [ ] 判定为 PROTOTYPE ABANDONED（不是 PROTOTYPE COMPLETE）
- [ ] `findings.md` 记录具体失败原因（不是模糊的）
- [ ] 在交接中建议替代方法
- [ ] 保留原型文件（不删除）以供参考

---

### 用例 5：Director 关卡检查——无关卡；原型是验证产物

**Fixture：**
- 提供了机制概念

**输入：** `/prototype wall-jump`

**预期行为：**
1. Skill 创建并记录原型
2. 不派生任何 director agent
3. 输出中不出现 gate ID

**断言：**
- [ ] 未调用 director 关卡
- [ ] 不出现 gate 跳过消息
- [ ] 判定为 PROTOTYPE COMPLETE 或 PROTOTYPE ABANDONED——无 gate 判定

---

## 协议合规性

- [ ] 创建任何文件前询问"可以写入 `prototypes/[name]/` 吗？"
- [ ] 在 `prototypes/` 下创建所有文件（不是 `src/`）
- [ ] 生成包含 tested/worked/didn't-work/recommendation 的 `findings.md`
- [ ] 注意生产编码标准故意放宽
- [ ] 原型已存在时提供 extend/replace/archive
- [ ] 判定为 PROTOTYPE COMPLETE 或 PROTOTYPE ABANDONED

---

## 覆盖说明

- 原型实现质量（代码风格）故意不测试——
  原型是一次性产物，质量标准不适用。
- 归档机制在用例 2 中提到，但归档格式未进行详细断言测试。
- 引擎特定的原型脚手架（GDScript 场景 vs. C# MonoBehaviour）
  遵循相同的流程，使用适合引擎的文件类型。
