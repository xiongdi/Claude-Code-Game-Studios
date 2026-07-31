---
name: asset-audit
description: "Audits game assets for compliance with naming conventions, file size budgets, format standards, and pipeline requirements. Identifies orphaned assets, missing references, and standard violations."
argument-hint: "[category|all]"
user-invocable: true
allowed-tools: Read, Glob, Grep
model: sonnet
# 只读诊断 skill —— 不需要专家 agent 委托
---

## 阶段 1：读取标准

从相关设计文档和 CLAUDE.md 命名约定中读取艺术圣经或资产标准。

---

## 阶段 2：扫描资产目录

使用 Glob 扫描目标资产目录：

- `assets/art/**/*` 用于美术资产
- `assets/audio/**/*` 用于音频资产
- `assets/vfx/**/*` 用于 VFX 资产
- `assets/shaders/**/*` 用于着色器
- `assets/data/**/*` 用于数据文件

---

## 阶段 3：运行合规检查

**命名约定：**
- 美术：`[category]_[name]_[variant]_[size].[ext]`
- 音频：`[category]_[context]_[name]_[variant].[ext]`
- 所有文件必须为小写加下划线

**文件标准：**
- 纹理：二次方尺寸，正确格式（UI 用 PNG，3D 用压缩格式），在大小预算内
- 音频：正确采样率，格式（SFX 用 OGG，音乐用 OGG/MP3），在时长限制内
- 数据：有效的 JSON/YAML，符合模式

**孤立资产：** 在代码中搜索对每个资产文件的引用。标记任何没有引用的资产。

**缺失资产：** 在代码中搜索资产引用并验证文件是否存在。

---

## 阶段 4：输出审计报告

```markdown
# Asset Audit Report -- [Category] -- [Date]

## Summary
- **Total assets scanned**: [N]
- **Naming violations**: [N]
- **Size violations**: [N]
- **Format violations**: [N]
- **Orphaned assets**: [N]
- **Missing assets**: [N]
- **Overall health**: [CLEAN / MINOR ISSUES / NEEDS ATTENTION]

## Naming Violations
| File | Expected Pattern | Issue |
|------|-----------------|-------|

## Size Violations
| File | Budget | Actual | Overage |
|------|--------|--------|---------|

## Format Violations
| File | Expected Format | Actual Format |
|------|----------------|---------------|

## Orphaned Assets (no code references found)
| File | Last Modified | Size | Recommendation |
|------|-------------|------|---------------|

## Missing Assets (referenced but not found)
| Reference Location | Expected Path |
|-------------------|---------------|

## Recommendations
[Prioritized list of fixes]

## Verdict: [COMPLIANT / WARNINGS / NON-COMPLIANT]
```

此 skill 是只读的 —— 它生成报告但不写入文件。

---

## 阶段 5：下一步

- 使用 CLAUDE.md 中定义的模式修复命名违规。
- 手动审查后删除确认的孤立资产。
- 运行 `/content-audit` 根据 GDD 指定的需求交叉检查资产数量。
