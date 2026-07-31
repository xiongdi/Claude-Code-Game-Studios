---
name: perf-profile
description: "Structured performance profiling workflow. Identifies bottlenecks, measures against budgets, and generates optimization recommendations with priority rankings."
argument-hint: "[system-name or 'full']"
user-invocable: true
agent: performance-analyst
allowed-tools: Read, Glob, Grep, Bash
model: sonnet
---

## Phase 1: 确定范围

读取参数：

- 系统名称 → 聚焦于该特定系统的性能分析
- `full` → 跨所有系统进行全面的性能分析

---

## Phase 2: 加载性能预算

检查设计文档或 CLAUDE.md 中是否存在现有性能目标：

- 目标 FPS（例如 60fps = 16.67ms 帧预算）
- 内存预算（总量和每系统）
- 加载时间目标
- Draw call 预算
- 网络带宽限制（如果是多人游戏）

---

## Phase 3: 分析代码库

**CPU 性能分析目标：**
- `_process()` / `Update()` / `Tick()` 函数 — 列出所有函数并估算成本
- 大集合上的嵌套循环
- 热路径中的字符串操作
- 每帧代码中的分配模式
- 游戏实体上未优化的搜索/排序
- 每帧昂贵的物理查询（raycast、overlap）

**内存性能分析目标：**
- 大型数据结构及其增长模式
- 纹理/资源内存占用估算
- 对象池 vs instantiate/destroy 模式
- 泄漏引用（应该释放但未释放的对象）
- 缓存大小和淘汰策略

**渲染目标（如适用）：**
- Draw call 估算
- 重叠透明物体导致的过度绘制
- Shader 复杂度
- 未优化的粒子系统
- 缺少 LOD 或遮挡剔除

**I/O 目标：**
- 保存/加载性能
- 资源加载模式（同步 vs 异步）
- 网络消息频率和大小

---

## Phase 4: 生成性能分析报告

```markdown
## 性能分析: [系统或全面]
生成时间: [日期]

### 性能预算
| 指标 | 预算 | 估算当前值 | 状态 |
|--------|--------|-------------------|--------|
| 帧时间 | [16.67ms] | [估算] | [OK/WARNING/OVER] |
| 内存 | [目标] | [估算] | [OK/WARNING/OVER] |
| 加载时间 | [目标] | [估算] | [OK/WARNING/OVER] |
| Draw calls | [目标] | [估算] | [OK/WARNING/OVER] |

### 已识别热点
| # | 位置 | 问题 | 估算影响 | 修复工作量 |
|---|----------|-------|------------------|------------|

### 优化建议（按优先级排序）
1. **[标题]** — [描述]
   - 位置: [文件:行号]
   - 预期收益: [估算]
   - 风险: [低/中/高]
   - 方法: [如何实现]

### 快速优化（每个 < 1 小时）
- [简单优化 1]

### 需要进一步调查
- [需要实际运行时性能分析才能确认影响的区域]
```

输出报告并附带摘要：前 3 个热点、相对预算的估算余量，以及推荐的后续操作。

---

## Phase 5: 范围与时间线决策

仅当任何热点的修复工作量为 M 或 L 时，才激活此阶段。

展示需要大量工作量的项目，并要求用户为每个项目选择：

- **A) 实施优化**（立即修复或安排修复）
- **B) 缩减功能范围**（运行 `/scope-check [功能]` 分析权衡）
- **C) 接受性能影响并推迟到 Polish 阶段**（记录为已知问题）
- **D) 升级到 technical-director 进行架构决策**（运行 `/architecture-decision`）

如果多个项目被推迟到 Polish（选择 C），将它们记录在 `### Deferred to Polish` 下。

此 skill 是只读的 — 不写入任何文件。判定：**COMPLETE** — 性能分析已生成。

---

## Phase 6: 后续步骤

- 如果瓶颈需要架构变更：运行 `/architecture-decision`。
- 如果需要缩减范围：运行 `/scope-check [功能]`。
- 要安排优化：运行 `/sprint-plan update`。

### 规则
- 永远不要在没有测量的情况下先优化 — 对性能的直觉是不可靠的
- 建议必须包含估算影响 — "让它更快" 是不可操作的
- 在目标硬件上进行性能分析，而不仅仅是在开发机上
- 静态分析（本 skill）识别候选者；运行时性能分析确认
