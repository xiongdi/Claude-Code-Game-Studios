# 设置要求

此模板需要安装一些工具以实现完整功能。
如果工具缺失，所有 hooks 都会优雅地失败 — 不会破坏任何东西，
只是失去验证功能。

## 必需

| 工具 | 用途 | 安装 |
| ---- | ---- | ---- |
| **Git** | 版本控制、分支管理 | [git-scm.com](https://git-scm.com/) |
| **Claude Code** | AI agent CLI | `npm install -g @anthropic-ai/claude-code` |

## 推荐

| 工具 | 使用者 | 用途 | 安装 |
| ---- | ---- | ---- | ---- |
| **jq** | Hooks（4/8 个） | 提交/推送/资源/agent hooks 中的 JSON 解析 | 见下文 |
| **Python 3** | Hooks（2/8 个） | 数据文件的 JSON 验证 | [python.org](https://www.python.org/) |
| **Bash** | 所有 hooks | Shell 脚本执行 | Git for Windows 自带 |

### 安装 jq

**Windows**（任选其一）：
```
winget install jqlang.jq
choco install jq
scoop install jq
```

**macOS**:
```
brew install jq
```

**Linux**:
```
sudo apt install jq     # Debian/Ubuntu
sudo dnf install jq     # Fedora
sudo pacman -S jq       # Arch
```

## 平台说明

### Windows
- Git for Windows 包含 **Git Bash**，提供所有 hooks 在 `settings.json` 中使用的 `bash` 命令
- 确保 Git Bash 在你的 PATH 上（如果通过 Git 安装程序安装则为默认）
- Hooks 使用 `bash .claude/hooks/[name].sh` — 这在 Windows 上工作，因为
  Claude Code 通过能找到 `bash.exe` 的 shell 调用命令

### macOS / Linux
- Bash 本地可用
- 通过包管理器安装 `jq` 以获得完整 hook 支持

## 验证你的设置

运行这些命令检查前提条件：

```bash
git --version          # 应显示 git 版本
bash --version         # 应显示 bash 版本
jq --version           # 应显示 jq 版本（可选）
python3 --version      # 应显示 python 版本（可选）
```

## 没有可选工具会发生什么

| 缺失工具 | 影响 |
| ---- | ---- |
| **jq** | 提交验证、推送保护、资源验证和 agent 审计 hooks 静默跳过其检查。提交和推送仍然有效。 |
| **Python 3** | 提交和资源 hooks 中的 JSON 数据文件验证被跳过。无效 JSON 可以在没有警告的情况下提交。 |
| **两者都是** | 所有 hooks 仍然执行而不出错（exit 0）但提供无验证。你在没有任何安全网的情况下飞行。 |

## 推荐 IDE

Claude Code 可与任何编辑器配合，但模板针对以下优化：
- **VS Code** 配合 Claude Code 扩展
- **Cursor**（Claude Code 兼容）
- 基于终端的 Claude Code CLI