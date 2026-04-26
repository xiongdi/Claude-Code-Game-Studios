# 引擎参考文档

此目录包含此项目使用的游戏引擎的策划、版本固定的文档快照。
这些文件存在是因为 **LLM 知识有截止日期**且游戏引擎频繁更新。

## 为什么存在

Claude 的训练数据有知识截止日期（目前是 2025 年 5 月）。
像 Godot、Unity 和 Unreal 这样的游戏引擎发布引入破坏性 API 更改、
新功能和弃用模式的更新。没有这些参考文件，Agent 会建议过时的代码。

## 结构

每个引擎有自己的目录：

```
<engine>/
├── VERSION.md              # 固定版本、验证日期、知识差距窗口
├── breaking-changes.md     # 版本间的 API 更改，按风险级别组织
├── deprecated-apis.md      # "不要使用 X → 使用 Y" 查找表
├── current-best-practices.md  # 模型训练数据中没有的新实践
└── modules/                # 每个子系统快速参考（每个最多约 150 行）
    ├── rendering.md
    ├── physics.md
    └── ...
```

## Agent 如何使用这些文件

引擎专家 Agent 被指示：

1. 读取 `VERSION.md` 确认当前引擎版本
2. 在建议任何引擎 API 前检查 `deprecated-apis.md`
3. 查阅 `breaking-changes.md` 了解版本特定问题
4. 阅读相关 `modules/*.md` 用于子系统特定工作

## 维护

### 何时更新

- 升级引擎版本后
- 当 LLM 模型更新时（新知识截止日期）
- 运行 `/refresh-docs` 后（如可用）
- 当你发现模型错误的 API 时

### 如何更新

1. 用新引擎版本和日期更新 `VERSION.md`
2. 为版本转换在 `breaking-changes.md` 添加新条目
3. 将新弃用的 API 移入 `deprecated-apis.md`
4. 用新模式更新 `current-best-practices.md`
5. 用 API 更改更新相关 `modules/*.md`
6. 在所有修改的文件上设置"最后验证"日期

### 质量规则

- 每个文件必须有"最后验证：YYYY-MM-DD"日期
- 保持模块文件在 150 行以下（上下文预算）
- 包括显示正确/错误模式的代码示例
- 链接到官方文档 URL 用于验证
- 只记录与模型训练数据不同的内容