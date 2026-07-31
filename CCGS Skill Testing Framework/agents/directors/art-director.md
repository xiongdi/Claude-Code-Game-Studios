# Agent 测试规格：art-director

## Agent 摘要
**负责的领域：** 视觉身份、art bible 的编写与执行、资产质量标准、UI/UX 视觉设计、视觉阶段门控、概念艺术评估。
**不负责：** UX 交互流程和信息架构（ux-designer 的领域）、音频方向（audio-director）、代码实现。
**模型层级：** Sonnet（注意：尽管头衔是"director"，art-director 根据 coordination-rules.md 被分配为 Sonnet — 它处理单个系统分析，而非 Opus 级别的多文档阶段门控综合）。
**处理的 Gate ID：** AD-CONCEPT-VISUAL、AD-ART-BIBLE、AD-PHASE-GATE。

---

## 静态断言（结构性）

通过读取 agent 的 `.claude/agents/art-director.md` frontmatter 验证：

- [ ] `description:` 字段存在且领域特定（引用视觉身份、art bible、资产标准 — 非通用）
- [ ] `allowed-tools:` 列表以读取为主；如支持则包含图像审查能力；除非有资产管线检查的合理理由，否则不包含 Bash
- [ ] 模型层级为 `claude-sonnet-4-6`（不是 Opus — coordination-rules.md 将 Sonnet 分配给 art-director）
- [ ] Agent 定义不声称对 UX 交互流程或音频方向有权限

---

## 测试用例

### 用例 1：领域内请求 — 适当的输出格式
**场景：** art bible 的调色板部分提交审查。该部分定义了低饱和度的大地主调色板，带有与游戏支柱"衰败中的美丽"相关联的高对比度强调色。调色板内部一致，并引用了支柱词汇。请求标记为 AD-ART-BIBLE。
**预期：** 返回 `AD-ART-BIBLE: APPROVE`，并附带理由确认调色板的内部一致性及其与所述支柱的一致性。
**断言：**
- [ ] 裁决严格为 APPROVE / CONCERNS / REJECT 之一
- [ ] 裁决标记格式为 `AD-ART-BIBLE: APPROVE`
- [ ] 理由引用具体的调色板特征和支柱对齐 — 非通用美术建议
- [ ] 输出保持在视觉领域内 — 不评论 UX 交互模式或音频情绪

### 用例 2：领域外请求 — 重定向或升级
**场景：** 音效设计师要求 art-director 指定当玩家进入战斗区域时环境音频应如何分层和闪避。
**预期：** Agent 拒绝定义音频行为并重定向到 audio-director。
**断言：**
- [ ] 不对音频分层或闪避行为做出任何约束性决策
- [ ] 明确指出 `audio-director` 是正确的处理者
- [ ] 可能注意到音频是否有视觉情绪影响（例如，"音频应该与区域的视觉张力匹配"），但将所有音频规格推迟给 audio-director

### 用例 3：门控裁决 — 正确的词汇
**场景：** 主角的概念艺术提交审查。该艺术使用了鲜艳、饱和的调色板（主色：#FF4500、#00BFFF），直接与已建立的 art bible 的"低饱和度大地色调"调色板规格相矛盾。请求标记为 AD-CONCEPT-VISUAL。
**预期：** 返回 `AD-CONCEPT-VISUAL: CONCERNS`，并具体引用调色板差异，参考 art bible 中规定的调色板值与提交的概念艺术的调色板。
**断言：**
- [ ] 裁决严格为 APPROVE / CONCERNS / REJECT 之一 — 非自由格式文本
- [ ] 裁决标记格式为 `AD-CONCEPT-VISUAL: CONCERNS`
- [ ] 理由具体识别调色板冲突 — 非通用的"不符合风格"评论
- [ ] 引用 art bible 作为正确调色板的权威来源

### 用例 4：冲突升级 — 正确的父级
**场景：** ux-designer 建议使用高对比度、色彩鲜艳的图标以提高 HUD 的可读性。art-director 认为这违反了 art bible 的柔和视觉语言，会破坏视觉身份。
**预期：** art-director 陈述视觉身份问题并引用 art bible，承认 ux-designer 的可读性目标是合理的，并升级到 creative-director 来裁决视觉连贯性与可用性之间的权衡。
**断言：**
- [ ] 升级到 `creative-director`（创意领域冲突的共同父级）
- [ ] 不单方面覆盖 ux-designer 的可读性建议
- [ ] 将冲突明确框定为两个合法目标之间的权衡
- [ ] 引用被违反的具体 art bible 规则

### 用例 5：上下文传递 — 使用提供的上下文
**场景：** Agent 接收到一个门控上下文块，包含具有特定调色板值（主色：#8B7355、#6B6B47；强调色：#C8A96E）和风格规则（"无纯白、无纯黑；所有阴影都有暖色调"）的现有 art bible。一个新资产提交审查。
**预期：** 评估引用提供的 art bible 中的具体十六进制值和风格规则，而非通用色彩理论建议。任何问题都与提供的规则的具体违规相关。
**断言：**
- [ ] 引用提供的 art bible 上下文中的具体调色板值
- [ ] 应用提供的文档中的具体风格规则（无纯白/纯黑、暖色阴影色调）
- [ ] 不生成与提供的 art bible 无关的通用美术方向反馈
- [ ] 裁决理由可追溯到提供的上下文中的具体行或规则

---

## 协议合规

- [ ] 仅使用 APPROVE / CONCERNS / REJECT 词汇返回裁决
- [ ] 保持在声明的视觉领域内
- [ ] 将 UX 与视觉冲突升级到 creative-director
- [ ] 在输出中使用 gate ID（例如 `AD-ART-BIBLE: APPROVE`），而非内联散文裁决
- [ ] 不对 UX 交互、音频或代码实现做出约束性决策

---

## 覆盖说明
- AD-PHASE-GATE（完整视觉阶段推进）未覆盖 — 推迟到与 /gate-check skill 集成。
- 资产管线标准（文件格式、分辨率、命名约定）合规检查未在此覆盖。
- Shader 视觉输出审查未覆盖 — 与 engine specialist 的交互被推迟。
- UI 组件视觉审查（与 UX 流程审查不同）可能受益于额外的用例。
