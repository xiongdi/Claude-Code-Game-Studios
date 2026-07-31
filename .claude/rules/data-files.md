---
paths:
  - "assets/data/**"
---

# 数据文件规则

- 所有 JSON 文件必须是合法的 JSON — 损坏的 JSON 会阻塞整个构建管线
- 文件命名：仅使用小写字母和下划线，遵循 `[system]_[name].json` 模式
- 每个数据文件必须有文档化的 schema（JSON Schema 或在对应的设计文档中说明）
- 数值必须包含注释或配套文档，解释数字的含义
- 使用一致的键命名：JSON 文件内使用 camelCase
- 不允许存在孤立的数据条目 — 每个条目必须被代码或另一个数据文件引用
- 进行破坏性 schema 变更时，为数据文件添加版本标识
- 所有可选字段必须包含合理的默认值

## 示例

**正确**的命名和结构（`combat_enemies.json`）：

```json
{
  "goblin": {
    "baseHealth": 50,
    "baseDamage": 8,
    "moveSpeed": 3.5,
    "lootTable": "loot_goblin_common"
  },
  "goblin_chief": {
    "baseHealth": 150,
    "baseDamage": 20,
    "moveSpeed": 2.8,
    "lootTable": "loot_goblin_rare"
  }
}
```

**错误**的示例（`EnemyData.json`）：

```json
{
  "Goblin": { "hp": 50 }
}
```

违规项：文件名大写、键名大写、未遵循 `[system]_[name]` 模式、缺少必需字段。
