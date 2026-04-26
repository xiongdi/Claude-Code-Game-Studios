# Agent 规格: [agent-name]

> **层级**: [directors | leads | specialists | godot | unity | unreal | operations | creative]
> **类别**: [director | lead | specialist | engine | operations | creative]
> **规格编写**: [YYYY-MM-DD]

## Agent 摘要

[一段描述此 agent 的领域、它拥有什么决策、以及它委托什么 vs. 直接处理什么。包括它触发哪些 gates（如果有）。]

**领域**: [此 agent 拥有的文件/目录]
**升级到**: [父 agent — 如 design 冲突的 creative-director]
**委托给**: [此 agent 通常派生的 sub-agents]

---

## 静态断言

- [ ] Agent 文件存在于 `.claude/agents/[name].md`
- [ ] Frontmatter 有 `name`、`description`、`model`、`tools` 字段
- [ ] 领域清楚说明
- [ ] 升级路径已记录
- [ ] 不在其领域外做决策

---

## 测试用例

### 案例 1: 领域内请求 — [简要名称]

**场景**: 明显在此 agent 领域内的请求。

**Fixture**:
- [相关项目状态]
- [提供给 agent 的输入]

**预期行为**:
1. Agent 接受请求
2. Agent 产生 [特定输出类型]
3. Agent 在写入文件前询问（如适用）

**断言**:
- [ ] Agent 在其领域内处理请求而不升级
- [ ] 输出格式匹配预期结构
- [ ] 遵循协作协议（问 → 草稿 → 批准）

**案例裁决**: PASS / FAIL / PARTIAL

---

### 案例 2: 领域外重定向 — [简要名称]

**场景**: 属于此 agent 领域之外的请求。

**Fixture**:
- [属于不同 agent 的请求]

**预期行为**:
1. Agent 识别请求在领域外
2. Agent 重定向到正确的 agent
3. Agent 不尝试处理它

**断言**:
- [ ] Agent 拒绝并重定向（不静默处理跨域工作）
- [ ] 在重定向中命名正确的 agent

**案例裁决**: PASS / FAIL / PARTIAL

---

### 案例 3: Gate 裁决 — [简要名称]

**场景**: Agent 作为 director gate 检查的一部分被调用。

**Fixture**:
- [呈现审查的项目状态]
- [gate ID: 如 CD-PHASE-GATE]

**预期行为**:
1. Agent 读取相关文档
2. Agent 产生 PASS / CONCERNS / FAIL 裁决
3. Agent 在 CONCERNS 或 FAIL 时不自动推进

**断言**:
- [ ] 输出中存在裁决关键词（PASS, CONCERNS, FAIL）
- [ ] 提供裁决推理
- [ ] 在 CONCERNS/FAIL 时：工作被阻塞，不静默继续

**案例裁决**: PASS / FAIL / PARTIAL

---

### 案例 4: 冲突升级 — [简要名称]

**场景**: 此 agent 的领域与另一个 agent 的决策冲突。

**Fixture**:
- [来自同层级的两个 agent 的冲突决策]

**预期行为**:
1. Agent 识别冲突
2. Agent 升级到共享父级（或者 creative-director / technical-director）
3. Agent 不单方面解决跨域冲突

**断言**:
- [ ] 冲突被明确 surfaced
- [ ] 遵循正确的升级路径
- [ ] 不做单方面跨域变更

**案例裁决**: PASS / FAIL / PARTIAL

---

### 案例 5: 上下文传递 — [简要名称]

**场景**: Agent 从父 agent 接收带有完整上下文的任务。

**Fixture**:
- [从父 agent 传递的上下文块]
- [要执行的特定子任务]

**预期行为**:
1. Agent 读取并使用提供的上下文
2. Agent 完成子任务
3. Agent 返回结果给父级（不不必要的提示用户）

**断言**:
- [ ] Agent 使用提供的上下文而不是重新请求它
- [ ] 结果限定于子任务，不超出范围
- [ ] 输出格式适合父 agent 消费

**案例裁决**: PASS / FAIL / PARTIAL

---

## 协议合规性

- [ ] 保持在声明的领域内 — 不做单方面跨域变更
- [ ] 将冲突升级到正确的父级
- [ ] 在文件写入前使用"May I write"（或是只读的）
- [ ] 在请求批准前展示发现
- [ ] 不在委托层级中跳过层级

---

## 覆盖范围备注

[覆盖范围中的任何差距、已知未测试的边缘情况，或需要 live agent 调用才能验证的行为。]
