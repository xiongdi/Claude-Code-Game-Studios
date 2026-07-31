# Godot UI — 快速参考

Last verified: 2026-02-12 | Engine: Godot 4.6

## 自 ~4.3 版本（LLM 截止）以来的变更

### 4.6 变更
- **双焦点系统**: 鼠标/触摸焦点现在与键盘/手柄焦点分离
  - 视觉反馈因输入方式而异
  - 自定义焦点实现可能需要更新
- **TabContainer**: Tab 属性可直接在 Inspector 中编辑
- **TileMapLayer 场景 tile 旋转**: 场景 tile 可以像 atlas tile 一样旋转

### 4.5 变更
- **FoldableContainer**: 新的手风琴式 UI 节点，用于可折叠部分
- **递归 Control 行为**: 单个属性禁用整个节点层级的鼠标/焦点
- **Screen reader 支持**: Control 节点与 AccessKit 配合工作
- **实时翻译预览**: 在编辑器中测试不同区域设置
- **`RichTextLabel.push_meta`**: 添加了可选的 `tooltip` 参数（自 4.4 起）

### 4.4 变更
- **`GraphEdit.connect_node`**: 添加了可选的 `keep_alive` 参数

## 当前 API 模式

### 主题和样式（4.6）
```gdscript
# 编辑器默认使用新的 "Modern" 主题
# 对于游戏 UI，像以前一样使用自定义主题：
var theme := Theme.new()
theme.set_color(&"font_color", &"Label", Color.WHITE)
theme.set_font_size(&"font_size", &"Label", 24)
```

### 焦点管理（4.6 — 已变更）
```gdscript
# 键盘/手柄焦点（grab_focus 仍然有效）
func _ready() -> void:
    %StartButton.grab_focus()

# 重要：在 4.6 中，鼠标悬停与键盘焦点分离
# 两者可以同时在不同控件上激活
# 使用鼠标和键盘/手柄两种方式测试你的 UI

# 焦点邻居（未变更）
%Button1.focus_neighbor_bottom = %Button2.get_path()
%Button1.focus_neighbor_right = %Button3.get_path()
```

### FoldableContainer（4.5 — 新增）
```gdscript
# 手风琴式可折叠容器
# 添加为要折叠内容的父节点
# 点击标题时子节点显示/隐藏
# 通过编辑器属性或代码配置
```

### 递归禁用（4.5 — 新增）
```gdscript
# 禁用层级的所有鼠标/焦点交互
# 用于禁用整个菜单部分
%SettingsPanel.mouse_filter = Control.MOUSE_FILTER_IGNORE
# 在 4.5+ 中，这可以递归传播到子节点
```

### 本地化就绪的 UI（最佳实践）
```gdscript
# 对所有可见字符串使用 tr()
label.text = tr("MENU_START_GAME")

# 对标签使用自动换行（文本长度因语言而异）
label.autowrap_mode = TextServer.AUTOWRAP_WORD_SMART

# 在编辑器中使用实时翻译预览测试（4.5+）
```

## 常见错误
- 假设 `grab_focus()` 影响鼠标焦点（在 4.6 中仅影响键盘/手柄）
- 升级到 4.6 后未使用鼠标和手柄两种方式测试 UI
- 硬编码字符串而非使用 `tr()` 进行本地化
- 对可折叠 UI 未使用 `FoldableContainer`（4.5 中新增，比自定义更简洁）
