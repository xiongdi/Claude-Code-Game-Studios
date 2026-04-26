# CCGS 技能测试框架

**Claude Code Game Studios** 框架的质量保证基础设施。
测试技能和 agents 本身 — 而不是用它们构建的任何游戏。

> **此文件夹是自包含的和可选的。**
> 使用 CCGS 的游戏开发者不需要它。要完全删除：
> `rm -rf "CCGS Skill Testing Framework"` — `.claude/` 中没有任何东西依赖它。

---

## 这里有什么

```
CCGS Skill Testing Framework/
├── README.md              ← 你在这里
├── CLAUDE.md              ← 告诉 Claude 如何使用此框架
├── catalog.yaml           ← 主注册表：所有 72 个技能 + 49 个 agents，覆盖追踪
├── quality-rubric.md      ← /skill-test category 的类别特定通过/失败指标
│
├── skills/                ← 技能的行为规格文件（每个技能一个）
│   ├── gate/              ← gate 类别规格
│   ├── review/            ← review 类别规格
│   ├── authoring/         ← authoring 类别规格
│   ├── readiness/         ← readiness 类别规格
│   ├── pipeline/          ← pipeline 类别规格
│   ├── analysis/          ← analysis 类别规格
│   ├── team/              ← team 类别规格
│   ├── sprint/            ← sprint 类别规格
│   └── utility/           ← utility 类别规格
│
├── agents/                ← agents 的行为规格文件（每个 agent 一个）
│   ├── directors/         ← creative-director, technical-director, producer, art-director
│   ├── leads/             ← lead-programmer, narrative-director, audio-director 等
│   ├── specialists/       ← engine/code/shader/UI 专家
│   ├── godot/             ← Godot 特定专家
│   ├── unity/             ← Unity 特定专家
│   ├── unreal/            ← Unreal 特定专家
│   ├── operations/        ← QA、live-ops、release、本地化等
│   └── creative/           ← writer, world-builder, game-designer 等
│
├── templates/             ← 编写新规格的规格文件模板
│   ├── skill-test-spec.md ← 技能行为规格模板
│   └── agent-test-spec.md ← Agent 行为规格模板
│
└── results/               ← 测试运行输出（由 /skill-test spec 写入，gitignored）
```

---

## 如何使用

所有测试由框架中已有的两个技能驱动：

### 检查结构合规性

```
/skill-test static [skill-name]     # 检查一个技能（7项检查）
/skill-test static all              # 检查所有 72 个技能
```

### 运行行为规格测试

```
/skill-test spec gate-check         # 根据其书面规格评估一个技能
/skill-test spec design-review
```

### 对照类别规则检查

```
/skill-test category gate-check     # 根据其类别指标评估一个技能
/skill-test category all            # 对所有分类技能运行规则检查
```

### 查看完整覆盖情况

```
/skill-test audit                   # 技能 + agents：has-spec、上次测试、结果
```

### 改进失败的技能

```
/skill-improve gate-check           # 测试 → 诊断 → 提出修复 → 重写 → 重测循环
```

---

## 技能类别

| 类别 | 技能 | 关键指标 |
|----------|--------|-------------|
| `gate` | gate-check | Review 模式读取、full/lean/solo director 面板、无自动推进 |
| `review` | design-review, architecture-review, review-all-gdds | 只读、8章节检查、正确裁决 |
| `authoring` | design-system, quick-design, art-bible, create-architecture, … | 逐节 May-I-write、骨架优先 |
| `readiness` | story-readiness, story-done | 阻塞项 surfaced、full 模式下的 director gate |
| `pipeline` | create-epics, create-stories, dev-story, map-systems, … | 上游依赖检查、交接路径清晰 |
| `analysis` | consistency-check, balance-check, code-review, tech-debt, … | 只读报告、裁决关键词、无写入 |
| `team` | team-combat, team-narrative, team-audio, … | 所有必需 agents 派生、阻塞 surfaced |
| `sprint` | sprint-plan, sprint-status, milestone-review, … | 读取 sprint 数据、状态关键词存在 |
| `utility` | start, adopt, hotfix, localize, setup-engine, … | 通过静态检查 |

---

## Agent 层级

| 层级 | Agents |
|------|--------|
| `directors` | creative-director, technical-director, producer, art-director |
| `leads` | lead-programmer, narrative-director, audio-director, ux-designer, qa-lead, release-manager, localization-lead |
| `specialists` | gameplay-programmer, engine-programmer, ui-programmer, tools-programmer, network-programmer, ai-programmer, level-designer, sound-designer, technical-artist |
| `godot` | godot-specialist, godot-gdscript-specialist, godot-csharp-specialist, godot-shader-specialist, godot-gdextension-specialist |
| `unity` | unity-specialist, unity-ui-specialist, unity-shader-specialist, unity-dots-specialist, unity-addressables-specialist |
| `unreal` | unreal-specialist, ue-gas-specialist, ue-replication-specialist, ue-umg-specialist, ue-blueprint-specialist |
| `operations` | devops-engineer, security-engineer, performance-analyst, analytics-engineer, community-manager |
| `creative` | writer, world-builder, game-designer, economy-designer, systems-designer, prototyper |

---

## 更新目录

`catalog.yaml` 追踪每个技能和 agent 的测试覆盖率。运行测试后：

- `/skill-test spec [name]` 将提供更新 `last_spec` 和 `last_spec_result`
- `/skill-test category [name]` 将提供更新 `last_category` 和 `last_category_result`
- `last_static` 和 `last_static_result` 手动或通过 `/skill-improve` 更新

---

## 编写新规格

1. 在 `templates/skill-test-spec.md` 找到规格模板
2. 复制到 `skills/[category]/[skill-name].md`
3. 更新 `catalog.yaml` 中的 `spec:` 字段指向新文件
4. 运行 `/skill-test spec [skill-name]` 验证

---

## 删除此框架

此文件夹没有挂钩到主项目。要删除：

```bash
rm -rf "CCGS Skill Testing Framework"
```

技能 `/skill-test` 和 `/skill-improve` 仍会工作 — 它们只是报告 `catalog.yaml` 缺失并建议运行 `/skill-test audit` 初始化它。
