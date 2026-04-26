# CLAUDE.local.md 模板

将此文件复制到项目根目录作为 `CLAUDE.local.md` 用于个人覆盖。
此文件是 gitignored 的，不会被提交。

```markdown
# 个人偏好

## 模型偏好
- 复杂设计任务优先使用 Opus
- 快速查询和简单编辑使用 Haiku

## 工作流程偏好
- 代码更改后始终运行测试
- 在 60% 使用率时主动压缩上下文
- 在不相关任务之间使用 /clear

## 本地环境
- Python 命令：python（或 py / python3）
- Shell：Windows 上的 Git Bash
- IDE：带 Claude Code 扩展的 VS Code

## 沟通风格
- 保持响应简洁
- 所有代码引用中显示文件路径
- 简要解释架构决策

## 个人快捷方式
- 当我说 "review" 时，对最后更改的文件运行 /code-review
- 当我说 "status" 时，显示 git 状态 + Sprint 进度
```

## 设置

1. 将此模板复制到项目根目录：`cp .claude/docs/CLAUDE-local-template.md CLAUDE.local.md`
2. 编辑以匹配你的偏好
3. 验证 `CLAUDE.local.md` 在 `.gitignore` 中（Claude Code 从项目根目录读取它）