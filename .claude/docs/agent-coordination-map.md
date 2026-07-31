# Agent 协调与委托地图

## 组织层级

```
                           [Human Developer]
                                 |
                 +---------------+---------------+
                 |               |               |
         creative-director  technical-director  producer
                 |               |               |
        +--------+--------+     |        (coordinates all)
        |        |        |     |
  game-designer art-dir  narr-dir  lead-programmer  qa-lead  audio-dir
        |        |        |         |                |        |
     +--+--+     |     +--+--+  +--+--+--+--+--+   |        |
     |  |  |     |     |     |  |  |  |  |  |  |   |        |
    sys lvl eco  ta   wrt  wrld gp ep  ai net tl ui qa-t    snd
                                 |
                             +---+---+
                             |       |
                          perf-a   devops   analytics

  Additional Leads (report to producer/directors):
    release-manager         -- 发布管线、版本控制、部署
    localization-lead       -- i18n、字符串表、翻译管线
    prototyper              -- 快速一次性原型、概念验证
    security-engineer       -- 反作弊、漏洞、数据隐私、网络安全
    accessibility-specialist -- WCAG、色盲、键位重映射、文本缩放
    live-ops-designer       -- 赛季、活动、战斗通行证、留存、实时经济
    community-manager       -- 补丁说明、玩家反馈、危机沟通

  Engine Specialists (use the SET matching your engine):
    unreal-specialist  -- UE5 lead: Blueprint/C++、GAS 概览、UE 子系统
      ue-gas-specialist         -- GAS: abilities、effects、attributes、tags、prediction
      ue-blueprint-specialist   -- Blueprint: BP/C++ 边界、graph 标准、优化
      ue-replication-specialist -- 网络: replication、RPCs、prediction、bandwidth
      ue-umg-specialist         -- UI: UMG、CommonUI、widget 层级、数据绑定

    unity-specialist   -- Unity lead: MonoBehaviour/DOTS、Addressables、URP/HDRP
      unity-dots-specialist         -- DOTS/ECS: Jobs、Burst、hybrid renderer
      unity-shader-specialist       -- Shaders: Shader Graph、VFX Graph、SRP 定制
      unity-addressables-specialist -- Assets: async loading、bundles、memory、CDN
      unity-ui-specialist           -- UI: UI Toolkit、UGUI、UXML/USS、数据绑定

    godot-specialist   -- Godot 4 lead: GDScript、node/scene、signals、resources
      godot-gdscript-specialist    -- GDScript: 静态类型、模式、signals、性能
      godot-csharp-specialist      -- C#: .NET 模式、[Signal] 委托、async、类型安全节点访问
      godot-shader-specialist      -- Shaders: Godot shading language、visual shaders、VFX
      godot-gdextension-specialist -- Native: C++/Rust 绑定、GDExtension、构建系统
```

### 图例
```
sys  = systems-designer       gp  = gameplay-programmer
lvl  = level-designer          ep  = engine-programmer
eco  = economy-designer        ai  = ai-programmer
ta   = technical-artist        net = network-programmer
wrt  = writer                  tl  = tools-programmer
wrld = world-builder           ui  = ui-programmer
snd  = sound-designer          qa-t = qa-tester
narr-dir = narrative-director perf-a = performance-analyst
art-dir = art-director
```

## 委托规则

### 谁可以委托给谁

| 从 | 可以委托给 |
|------|----------------|
| creative-director | game-designer、art-director、audio-director、narrative-director |
| technical-director | lead-programmer、devops-engineer、performance-analyst、technical-artist（技术决策） |
| producer | 任何 Agent（仅在其领域内的任务分配） |
| game-designer | systems-designer、level-designer、economy-designer |
| lead-programmer | gameplay-programmer、engine-programmer、ai-programmer、network-programmer、tools-programmer、ui-programmer |
| art-director | technical-artist、ux-designer |
| audio-director | sound-designer |
| narrative-director | writer、world-builder |
| qa-lead | qa-tester |
| release-manager | devops-engineer（发布构建）、qa-lead（发布测试） |
| localization-lead | writer（字符串审查）、ui-programmer（文本适配） |
| prototyper | （独立工作，向 producer 和相关主管报告结果） |
| security-engineer | network-programmer（安全审查）、lead-programmer（安全模式） |
| accessibility-specialist | ux-designer（无障碍模式）、ui-programmer（实现）、qa-tester（a11y 测试） |
| [engine]-specialist | 引擎子专家（委托子系统特定工作） |
| [engine] 子专家 | （为所有程序员提供引擎子系统模式和优化建议） |
| live-ops-designer | economy-designer（实时经济）、community-manager（活动沟通）、analytics-engineer（参与度指标） |
| community-manager | （与 producer 一起工作以获得批准，与 release-manager 一起确定补丁时机） |

### 升级路径

| 情况 | 升级到 |
|-----------|------------|
| 两个设计师不同意某个机制 | game-designer |
| 游戏设计 vs 叙事冲突 | creative-director |
| 游戏设计 vs 技术可行性 | producer（促进），然后 creative-director + technical-director |
| 美术 vs 音频基调冲突 | creative-director |
| 代码架构分歧 | technical-director |
| 跨系统代码冲突 | lead-programmer，然后 technical-director |
| 部门间进度冲突 | producer |
| 范围超出容量 | producer，然后 creative-director 削减 |
| 质量门分歧 | qa-lead，然后 technical-director |
| 性能预算违规 | performance-analyst 标记，technical-director 决定 |

## 常见工作流程模式

### 模式 1：新功能（完整管线）

```
1. creative-director  -- 批准功能概念符合愿景
2. game-designer      -- 创建带完整规格的设计文档
3. producer           -- 安排工作，识别依赖
4. lead-programmer    -- 设计代码架构，创建接口草图
5. [专家程序员] -- 实现功能
6. technical-artist   -- 实现视觉效果（如需要）
7. writer             -- 创建文本内容（如需要）
8. sound-designer     -- 创建音频事件列表（如需要）
9. qa-tester          -- 编写测试用例
10. qa-lead           -- 审查并批准测试覆盖率
11. lead-programmer   -- 代码审查
12. qa-tester         -- 执行测试
13. producer          -- 标记任务完成
```

### 模式 2：Bug 修复

```
1. qa-tester          -- 用 /bug-report 提交 Bug 报告
2. qa-lead            -- 分拣严重性和优先级
3. producer           -- 分配到 Sprint（如果不是 S1）
4. lead-programmer    -- 识别根本原因，分配给程序员
5. [专家程序员] -- 修复 Bug
6. lead-programmer    -- 代码审查
7. qa-tester          -- 验证修复并运行回归
8. qa-lead            -- 关闭 Bug
```

### 模式 3：平衡调整

```
1. analytics-engineer -- 从数据（或玩家报告）识别不平衡
2. game-designer      -- 根据设计意图评估问题
3. economy-designer   -- 建模调整
4. game-designer      -- 批准新值
5. [数据文件更新] -- 更改配置值
6. qa-tester          -- 受影响系统的回归测试
7. analytics-engineer -- 监控变更后指标
```

### 模式 4：新区域/关卡

```
1. narrative-director -- 定义区域叙事目的和节奏点
2. world-builder      -- 创建传说和环境背景
3. level-designer     -- 设计布局、遭遇、节奏
4. game-designer      -- 审查遭遇的机械设计
5. art-director       -- 定义区域视觉方向
6. audio-director     -- 定义区域音频方向
7. [由相关程序员和美术实现]
8. writer             -- 创建区域特定文本内容
9. qa-tester          -- 测试完整区域
```

### 模式 5：Sprint 周期

```
1. producer           -- 用 /sprint-plan new 规划 Sprint
2. [所有 Agent]       -- 执行分配的任务
3. producer           -- 用 /sprint-plan status 每日状态
4. qa-lead            -- Sprint 期间持续测试
5. lead-programmer    -- Sprint 期间持续代码审查
6. producer           -- 用 post-sprint hook 进行 Sprint 回顾
7. producer           -- 结合学习内容规划下一个 Sprint
```

### 模式 6：里程碑检查点

```
1. producer           -- 运行 /milestone-review
2. creative-director  -- 审查创意进度
3. technical-director -- 审查技术健康状况
4. qa-lead            -- 审查质量指标
5. producer           -- 促进 go/no-go 讨论
6. [所有导演]    -- 如需要就范围调整达成一致
7. producer           -- 记录决策并更新计划
```

### 模式 7：发布管线

```text
1. producer             -- 宣布发布候选，确认里程碑标准已满足
2. release-manager      -- 切出发布分支，生成 /release-checklist
3. qa-lead              -- 运行完整回归，签署质量
4. localization-lead    -- 验证所有字符串已翻译，文本适配通过
5. performance-analyst  -- 确认性能基准在目标内
6. devops-engineer      -- 构建发布产物，运行部署管线
7. release-manager      -- 生成 /changelog，标记发布，创建发布说明
8. technical-director   -- 主要发布最终签字
9. release-manager      -- 部署并监控 48 小时
10. producer            -- 标记发布完成
```

### 模式 8：概念原型（早期 — GDD 之前）

```text
1. game-designer        -- 定义假设和成功标准
2. prototyper           -- 用 /prototype 搭建概念原型
3. prototyper           -- 构建最小实现（1-3 天）
4. game-designer        -- 根据标准评估原型
5. prototyper           -- 在 REPORT.md 中记录发现
6. creative-director    -- PROCEED / PIVOT / KILL 决策（仅 full 模式）
7. game-designer        -- 如 PROCEED 则结合原型经验撰写 GDD
```

### 模式 8b：垂直切片（pre-production — GDD 和架构之后）

```text
1. game-designer        -- 对照 GDD 确认切片范围
2. prototyper           -- 用 /vertical-slice 构建生产级端到端构建
3. prototyper           -- 进行内部试玩会话（最少 1 次）
4. prototyper           -- 在 REPORT.md 中记录发现
5. creative-director    -- 投入 Production 的 go/no-go 决策（full 模式）
6. producer             -- 如 PROCEED 则安排 Production epic/sprint
```
```

### 模式 9：直播活动/赛季发布

```text
1. live-ops-designer     -- 设计活动/赛季内容、奖励、日程
2. game-designer         -- 验证活动游戏机制
3. economy-designer      -- 平衡活动经济和奖励值
4. narrative-director    -- 提供季节性叙事主题
5. writer                -- 创建活动描述和传说
6. producer              -- 安排实现工作
7. [由相关程序员实现]
8. qa-lead               -- 端到端测试活动流程
9. community-manager     -- 起草活动公告和补丁说明
10. release-manager      -- 部署活动内容
11. analytics-engineer   -- 监控活动参与度和指标
12. live-ops-designer    -- 活动后分析和学习
```

## 跨域通信协议

### 设计变更通知

当设计文档变更时，game-designer 必须通知：
- lead-programmer（实现影响）
- qa-lead（需要更新测试计划）
- producer（进度影响评估）
- 根据变更情况通知相关专家 Agent

### 架构变更通知

当创建或修改 ADR 时，technical-director 必须通知：
- lead-programmer（需要代码更改）
- 所有受影响的专业程序员
- qa-lead（测试策略可能更改）
- producer（进度影响）

### 资源标准变更通知

当美术圣经或资源标准变更时，art-director 必须通知：
- technical-artist（管线更改）
- 使用受影响资源的所有内容创建者
- devops-engineer（如果构建管线受影响）

## 应避免的反模式

1. **绕过层级**：专家 Agent 永远不应在未经协商的情况下做出属于其主管的决策。
2. **跨域实现**：Agent 永远不应在未经相关所有者明确委托的情况下修改其指定区域外的文件。
3. **暗箱决策**：所有决策必须有文档记录。口头协议没有书面记录会导致矛盾。
4. ** monolithic 任务**：分配给 Agent 的每个任务应在 1-3 天内完成。如果更大，必须先分解。
5. **基于假设的实现**：如果规格不明确，实现者必须向规格制定者提问而不是猜测。错误猜测比提问更昂贵。