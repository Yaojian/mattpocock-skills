---
name: claude-handoff
description: 将当前会话交接给一个新的后台智能体，由它立即接手继续工作。
argument-hint: "下个会话用来做什么？"
disable-model-invocation: true
---

为当前会话撰写一份交接总结，让新的智能体可以继续工作。不要保存这份总结，而是启动一个后台智能体，并以该总结作为它的提示词：`claude --bg --name "<descriptive name>" "<handoff summary>"`。它在当前工作目录中启动并立即返回；用户使用 `claude agents` 管理它。

始终传入 `-n`/`--name` 并附上描述性名称（例如 `--name "Fix login bug"`）；它会设置作业列表、会话选择器和终端标题中显示的名称。

在总结中加入 "suggested skills" 一节，写明下一个智能体应该通过 Skill 工具调用哪些技能。

不要重复其他产物中已有的内容（规格说明、计划、ADR、工单、提交记录、diff）。改用路径或 URL 引用它们。

脱敏所有敏感信息，例如 API 密钥、密码或个人身份信息，因为这份总结将成为该智能体的提示词。

如果用户传入了参数，将其视为对下个会话重点工作的描述，并据此调整总结内容。
