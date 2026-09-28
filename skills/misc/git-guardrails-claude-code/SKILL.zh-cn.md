---
name: git-guardrails-claude-code
description: 设置 Claude Code 钩子，在危险 git 命令执行前拦截（push、reset --hard、clean、branch -D 等）。当用户想要阻止破坏性 git 操作、添加 git 安全钩子，或在 Claude Code 中禁用 git push/reset 时使用。
---

# 设置 Git 防护栏

设置一个 PreToolUse 钩子，在 Claude 执行危险 git 命令之前拦截并阻止。

## 拦截范围

- `git push`（所有变体，包括 `--force`）
- `git reset --hard`
- `git clean -f` / `git clean -fd`
- `git branch -D`
- `git checkout .` / `git restore .`

被拦截后，Claude 会看到一条消息，告知它无权访问这些命令。

## 步骤

### 1. 确认作用域

询问用户：只给**当前项目**安装（`.claude/settings.json`），还是给**所有项目**安装（`~/.claude/settings.json`）？

### 2. 复制钩子脚本

随附脚本位于：[scripts/block-dangerous-git.sh](scripts/block-dangerous-git.sh)

按作用域复制到目标位置：

- **项目级**：`.claude/hooks/block-dangerous-git.sh`
- **全局**：`~/.claude/hooks/block-dangerous-git.sh`

用 `chmod +x` 赋予可执行权限。

### 3. 把钩子加入设置

加入对应的设置文件：

**项目级**（`.claude/settings.json`）：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

**全局**（`~/.claude/settings.json`）：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

如果设置文件已存在，把该钩子合并进现有的 `hooks.PreToolUse` 数组。不要覆盖其他设置。

### 4. 确认是否自定义

询问用户是否要在拦截清单中增删规则。按需编辑复制过去的脚本。

### 5. 验证

快速测试一次：

```bash
echo '{"tool_input":{"command":"git push origin main"}}' | <path-to-script>
```

应以退出码 2 退出，并向 stderr 打印 BLOCKED 消息。
