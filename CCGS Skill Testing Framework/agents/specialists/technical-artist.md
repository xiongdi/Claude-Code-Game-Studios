# Agent Test Spec: technical-artist

## Agent Summary
领域：Shader、VFX、渲染优化、美术管线工具以及视觉性能。
不负责：美术风格决策或配色方案（art-director）、游戏玩法代码（gameplay-programmer）。
模型层级：Sonnet（默认）。
未分配 gate ID。

---

## 静态断言（结构性）

- [ ] `description:` 字段存在且特定于领域（引用 shader / VFX / 渲染）
- [ ] `allowed-tools:` 列表包含 Read、Write、Edit、Bash、Glob、Grep
- [ ] 模型层级为 Sonnet（specialist 默认值）
- [ ] Agent 定义不声称对美术风格方向或游戏玩法逻辑拥有权限

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出
**输入：** "为敌人死亡序列创建一个溶解效果 shader。"
**预期行为：**
- 生成适用于已配置引擎的 shader 代码或 Shader Graph 节点规格（Godot shading language / Unity Shader Graph / Unreal Material Blueprint）
- 定义一个 `dissolve_amount` uniform（0.0–1.0）作为动画驱动
- 使用噪声纹理采样来确定溶解阈值
- 将边缘光照技术标注为可选增强
- 输出感知引擎版本（如果需要 post-cutoff API，则检查版本参考）

### 用例 2：领域外请求 — 正确重定向
**输入：** "定义 art bible 配色方案：UI 的主色、辅色和强调色。"
**预期行为：**
- 不生成配色方案决策或美术方向文档
- 明确声明美术风格决策属于 `art-director`
- 将请求重定向到 `art-director`
- 可能注明一旦配色方案确定，它可以后续实现调色或 palette LUT shader

### 用例 3：性能警告 — GPU 粒子数量
**输入：** "VFX 系统在爆炸池中以 50,000 个粒子触发了 GPU 粒子数量警告。"
**预期行为：**
- 生成针对特定警告的优化规格
- 提出具体策略：每个发射器的粒子预算上限、基于 LOD 的粒子减少、GPU 实例化，或切换为基于网格的 VFX 用于远处效果
- 在可计算时提供前后 GPU 成本估算
- 不改变爆炸的游戏玩法行为（将任何游戏玩法影响委托给 gameplay-programmer）

### 用例 4：引擎版本兼容性
**输入：** "对水 shader 使用新的纹理采样器 API。"
**预期行为：**
- 在建议任何 API 之前检查引擎版本参考（例如 `docs/engine-reference/godot/VERSION.md`）
- 如果请求的 API 是 post-cutoff（例如 Godot 4.4+ 纹理类型变更），则标记出来
- 为项目固定的引擎版本提供正确的语法
- 如果对 post-cutoff 行为不确定，明确声明不确定性并指向经过验证的文档

### 用例 5：上下文传递 — 使用性能预算
**输入：** 在上下文中提供来自 `technical-preferences.md` 的性能预算：2ms GPU 帧预算，最多 200 个 draw call。请求："优化森林渲染系统。"
**预期行为：**
- 引用提供的上下文中具体的 2ms GPU 预算和 200 draw call 限制
- 提出针对这些精确目标校准的优化（例如，"批处理将 draw call 从 340 减少到约 180，在 200 限制内"）
- 不会提出在其他维度上超出所述预算的优化
- 生成按预期影响与实现成本排序的优化列表

---

## 协议合规

- [ ] 在声明领域内保持（shader、VFX、渲染优化、美术管线）
- [ ] 将美术风格决策重定向到 art-director
- [ ] 返回结构化发现（shader 代码、带指标的优化规格、节点图）
- [ ] 未经明确委托不修改游戏玩法代码文件
- [ ] 在建议 post-cutoff API 之前检查引擎版本参考
- [ ] 针对所述预算量化性能变更

---

## 覆盖说明
- 溶解 shader（用例 1）应在 `production/qa/evidence/` 中包含视觉测试参考
- 引擎版本检查（用例 4）确认 agent 将 VERSION.md 视为权威
- 性能预算用例（用例 5）验证 agent 读取并应用提供的上下文数字
