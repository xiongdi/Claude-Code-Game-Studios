# CCGS 技能测试框架 — Claude 指令

此文件夹是 Claude Code Game Studios 技能/agent 框架的质量保证层。它是自包含的，与任何游戏项目分开。

## 关键文件

| 文件 | 用途 |
|------|---------|
| `catalog.yaml` | 所有 72 个技能和 49 个 agents 的主注册表。包含类别、规格路径和上次测试追踪字段。运行任何测试命令时首先读取此文件。 |
| `quality-rubric.md` | 类别特定的通过/失败指标。运行 `/skill-test category` 时读取技能类别的匹配 `###` 部分。 |
| `skills/[category]/[name].md` | 技能的行为规格 — 5个测试用例 + 协议合规断言。 |
| `agents/[tier]/[name].md` | Agent 的行为规格 — 5个测试用例 + 协议合规断言。 |
| `templates/skill-test-spec.md` | 编写新技能规格文件的模板。 |
| `templates/agent-test-spec.md` | 编写新 agent 规格文件的模板。 |
| `results/` | 由 `/skill-test spec` 在保存结果时写入。Gitignored。 |

## 路径约定

- 技能规格：`CCGS Skill Testing Framework/skills/[category]/[name].md`
- Agent 规格：`CCGS Skill Testing Framework/agents/[tier]/[name].md`
- 目录：`CCGS Skill Testing Framework/catalog.yaml`
- 规则：`CCGS Skill Testing Framework/quality-rubric.md`

`catalog.yaml` 中的 `spec:` 字段是每个技能/agent 规格的权威路径。始终读取它而不是猜测路径。

## 技能类别

```
gate        → gate-check
review      → design-review, architecture-review, review-all-gdds
authoring   → design-system, quick-design, architecture-decision, art-bible,
              create-architecture, ux-design, ux-review
readiness   → story-readiness, story-done
pipeline    → create-epics, create-stories, dev-story, create-control-manifest,
              propagate-design-change, map-systems
analysis    → consistency-check, balance-check, content-audit, code-review,
              tech-debt, scope-check, estimate, perf-profile, asset-audit,
              security-audit, test-evidence-review, test-flakiness
team        → team-combat, team-narrative, team-audio, team-level, team-ui,
              team-qa, team-release, team-polish, team-live-ops
sprint      → sprint-plan, sprint-status, milestone-review, retrospective,
              changelog, patch-notes
utility     → 所有剩余技能
```

## Agent 层级

```
directors   → creative-director, technical-director, producer, art-director
leads       → lead-programmer, narrative-director, audio-director, ux-designer,
              qa-lead, release-manager, localization-lead
specialists → gameplay-programmer, engine-programmer, ui-programmer,
              tools-programmer, network-programmer, ai-programmer,
              level-designer, sound-designer, technical-artist
godot       → godot-specialist, godot-gdscript-specialist, godot-csharp-specialist,
              godot-shader-specialist, godot-gdextension-specialist
unity       → unity-specialist, unity-ui-specialist, unity-shader-specialist,
              unity-dots-specialist, unity-addressables-specialist
unreal      → unreal-specialist, ue-gas-specialist, ue-replication-specialist,
              ue-umg-specialist, ue-blueprint-specialist
operations  → devops-engineer, security-engineer, performance-analyst,
              analytics-engineer, community-manager
creative    → writer, world-builder, game-designer, economy-designer,
              systems-designer, prototyper
```

## 测试技能的工作流程

1. 读取 `catalog.yaml` 获取技能的 `spec:` 路径和 `category:`
2. 读取 `.claude/skills/[name]/SKILL.md` 中的技能
3. 在 `spec:` 路径读取规格
4. 逐案评估断言
5. 提供写入结果到 `results/` 并更新 `catalog.yaml`

## 改进技能的工作流程

使用 `/skill-improve [name]`。它处理完整循环：
测试 → 诊断 → 提出修复 → 重写 → 重测 → 保留或回退。

## 规格有效性说明

此文件夹中的规格描述的是**当前行为**，而不是理想行为。它们是通过阅读技能编写的，因此可能编码了 bug。当技能在实践中行为不当时，先纠正技能，然后更新规格以匹配修复的行为。
将规格失败视为"这需要调查"，而不是"技能肯定错了"。

## 此文件夹可删除

`.claude/` 中没有任何东西从这里导入。删除此文件夹对 CCGS 技能或 agents 本身没有影响。`/skill-test` 和 `/skill-improve` 将报告 `catalog.yaml` 缺失并引导用户初始化它。
