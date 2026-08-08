# Godot Input — 快速参考

最后验证: 2026-02-12 | 引擎: Godot 4.6

## 自 ~4.3 版本（LLM 截止）以来的变更

### 4.6 变更
- **双焦点系统**: 鼠标/触摸焦点现在与键盘/手柄焦点分离
  - 视觉反馈因输入方式而异
  - 自定义焦点实现可能需要更新
- **Select Mode 快捷键变更**: "Select Mode" 现在是 `v` 键；旧模式重命名为 "Transform Mode"（`q` 键）

### 4.5 变更
- **SDL3 手柄驱动**: 手柄处理委托给 SDL 库以获得更好的跨平台支持
- **递归 Control 禁用**: 单个属性禁用整个节点层级的鼠标/焦点

### 4.3 变更（在训练数据中）
- **InputEventShortcut**: 菜单快捷键的专用事件类型（可选）

## 当前 API 模式

### Input Actions（未变更）
```gdscript
func _physics_process(delta: float) -> void:
    var input_dir: Vector2 = Input.get_vector(
        &"move_left", &"move_right", &"move_forward", &"move_back"
    )
    if Input.is_action_just_pressed(&"jump"):
        jump()
```

### Input Events（未变更）
```gdscript
func _unhandled_input(event: InputEvent) -> void:
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
            handle_click(event.position)
    elif event is InputEventKey:
        if event.keycode == KEY_ESCAPE and event.pressed:
            toggle_pause()
```

### Focus Management（4.6 — 已变更）
```gdscript
# 鼠标/触摸和键盘/手柄焦点现在分离
# 根据当前激活的输入方式，视觉样式可能不同
# 如果有自定义焦点绘制，请用两种输入方式测试

# 标准方法仍然有效：
func _ready() -> void:
    %StartButton.grab_focus()  # 键盘/手柄焦点

# 但请注意：在 4.6 中，鼠标悬停焦点 != 键盘焦点
```

### Gamepad（4.5+ — SDL3 后端）
```gdscript
# API 未变，但 SDL3 提供：
# - 更好的跨平台设备检测
# - 改进的震动支持
# - 更一致的按键映射

func _input(event: InputEvent) -> void:
    if event is InputEventJoypadButton:
        if event.button_index == JOY_BUTTON_A and event.pressed:
            confirm_selection()
```

## 常见错误
- 未测试鼠标和键盘两种焦点路径（4.6 中的双焦点）
- 假设 `grab_focus()` 影响鼠标焦点（在 4.6 中仅影响键盘/手柄）
- 在热路径中使用字符串字面量而非 `StringName`（`&"action"`）作为动作名称
