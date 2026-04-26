# 活跃 Hooks

Hooks 在 `.claude/settings.json` 中配置并自动触发：

| Hook | 事件 | 触发条件 | 操作 |
| ---- | ----- | ------- | ------ |
| `validate-commit.sh` | PreToolUse (Bash) | `git commit` 命令 | 验证设计文档章节、JSON 数据文件、硬编码值、TODO 格式 |
| `validate-push.sh` | PreToolUse (Bash) | `git push` 命令 | 警告推送到受保护分支（develop/main） |
| `validate-assets.sh` | PostToolUse (Write/Edit) | 资源文件更改 | 检查 `assets/` 中文件的命名约定和 JSON 有效性 |
| `session-start.sh` | SessionStart | 会话开始 | 加载 Sprint 上下文、里程碑、git 活动；检测并预览活动会话状态文件以供恢复 |
| `detect-gaps.sh` | SessionStart | 会话开始 | 检测新项目（建议 `/start`）以及代码/原型存在时缺少文档的情况，建议 `/reverse-document` 或 `/project-stage-detect` |
| `pre-compact.sh` | PreCompact | 上下文压缩 | 在压缩前将会话状态（active.md、修改的文件、WIP 设计文档）转储到对话中以便在摘要中存活 |
| `post-compact.sh` | PostCompact | 压缩后 | 提醒 Claude 从 `active.md` 检查点恢复会话状态 |
| `notify.sh` | Notification | 通知事件 | 通过 PowerShell 显示 Windows Toast 通知 |
| `session-stop.sh` | Stop | 会话结束 | 总结成就并更新会话日志 |
| `log-agent.sh` | SubagentStart | Agent 启动 | 审计追踪开始 — 记录带时间戳的子 Agent 调用 |
| `log-agent-stop.sh` | SubagentStop | Agent 停止 | 审计追踪停止 — 完成子 Agent 记录 |
| `validate-skill-change.sh` | PostToolUse (Write/Edit) | 技能文件更改 | 建议在任何 `.claude/skills/` 文件写入或编辑后运行 `/skill-test` |

Hook 参考文档：`.claude/docs/hooks-reference/`
Hook 输入 schema 文档：`.claude/docs/hooks-reference/hook-input-schemas.md`