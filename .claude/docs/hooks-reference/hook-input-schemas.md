# Hook Input/Output Schemas（Hook 输入/输出结构）

本文档记录了每个 Claude Code hook 在每种事件类型下通过 stdin 接收的 JSON 负载。

## PreToolUse

在工具执行之前触发。可以 **允许**（退出码 0）或 **阻止**（退出码 2）。

### PreToolUse: Bash

```json
{
  "tool_name": "Bash",
  "tool_input": {
    "command": "git commit -m 'feat: add player health system'",
    "description": "Commit changes with message",
    "timeout": 120000
  }
}
```

### PreToolUse: Write

```json
{
  "tool_name": "Write",
  "tool_input": {
    "file_path": "src/gameplay/health.gd",
    "content": "extends Node\n..."
  }
}
```

### PreToolUse: Edit

```json
{
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "src/gameplay/health.gd",
    "old_string": "var health = 100",
    "new_string": "var health: int = 100"
  }
}
```

### PreToolUse: Read

```json
{
  "tool_name": "Read",
  "tool_input": {
    "file_path": "src/gameplay/health.gd"
  }
}
```

## PostToolUse

在工具完成之后触发。**不能阻止**（退出码对阻止无效）。stderr 消息显示为警告。

### PostToolUse: Write

```json
{
  "tool_name": "Write",
  "tool_input": {
    "file_path": "assets/data/enemy_stats.json",
    "content": "{\"goblin\": {\"health\": 50}}"
  },
  "tool_output": "File written successfully"
}
```

### PostToolUse: Edit

```json
{
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "assets/data/enemy_stats.json",
    "old_string": "\"health\": 50",
    "new_string": "\"health\": 75"
  },
  "tool_output": "File edited successfully"
}
```

## SubagentStart

当通过 Task 工具派生 subagent 时触发。

```json
{
  "agent_name": "game-designer",
  "model": "sonnet",
  "description": "Design the combat healing mechanic"
}
```

## SessionStart

当 Claude Code 会话开始时触发。**无 stdin 输入** — hook 只是运行，其 stdout 作为上下文显示给 Claude。

## PreCompact

在上下文窗口压缩之前触发。**无 stdin 输入** — hook 在压缩发生之前运行以保存状态。

## Stop

当 Claude Code 会话结束时触发。**无 stdin 输入** — hook 运行以进行清理和日志记录。

## Exit Code Reference（退出码参考）

| 退出码 | 含义 | 适用事件 |
|-----------|---------|-------------------|
| 0 | 允许 / 成功 | 所有事件 |
| 2 | 阻止（stderr 显示给 Claude） | 仅 PreToolUse |
| 其他 | 视为错误，工具继续执行 | 所有事件 |

## Notes（备注）

- Hook 通过 **stdin**（管道）接收 JSON。使用 `INPUT=$(cat)` 捕获。
- 如果可用则用 `jq` 解析，否则回退到 `grep` 以保证跨平台兼容性。
- 在 Windows 上，`grep -P`（Perl 正则）通常不可用。使用 `grep -E`（POSIX 扩展）代替。
- 在 Windows 上路径分隔符可能是 `\`。比较路径时用 `sed 's|\\|/|g'` 规范化。
