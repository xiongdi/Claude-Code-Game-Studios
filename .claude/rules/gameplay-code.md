---
paths:
  - "src/gameplay/**"
---

# Gameplay 代码规则

- 所有 gameplay 数值**必须**来自外部配置/数据文件，绝不允许硬编码
- 所有与时间相关的计算必须使用 delta time（帧率无关）
- 不允许直接引用 UI 代码 — 使用事件/信号进行跨系统通信
- 每个 gameplay 系统必须实现清晰的接口
- 状态机必须具有显式的转换表，并记录各状态
- 为所有 gameplay 逻辑编写单元测试 — 将逻辑与表现分离
- 在代码注释中记录每个功能实现了哪个设计文档
- 不允许使用静态单例管理游戏状态 — 使用依赖注入

## 示例

**正确**的示例（数据驱动）：

```gdscript
var damage: float = config.get_value("combat", "base_damage", 10.0)
var speed: float = stats_resource.movement_speed * delta
```

**错误**的示例（硬编码）：

```gdscript
var damage: float = 25.0   # 违规：硬编码 gameplay 数值
var speed: float = 5.0      # 违规：不来自配置，未使用 delta
```
