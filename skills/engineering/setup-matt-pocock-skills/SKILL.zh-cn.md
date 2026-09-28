---
name: setup-matt-pocock-skills
description: "为工程技能配置本仓库：设置问题跟踪器、分诊标签词汇和领域文档布局。在首次使用其他工程技能之前运行一次。"
disable-model-invocation: true
---

# 配置 Matt Pocock 技能

搭建工程技能所依赖的按仓库配置：

- **问题跟踪器**：问题（issue）存放在哪里（默认 GitHub，也开箱支持本地 Markdown）
- **分诊标签**：五个标准分诊角色所用的字符串
- **领域文档**：`CONTEXT.md` 和 ADR 存放在哪里，以及阅读它们的用户规则

这是一个提示驱动的技能，不是确定性脚本。先探索，展示发现，与用户确认，然后再写。

## 流程

### 1. 探索

查看当前仓库，了解它的初始状态。看到什么读什么，不要假设：

- `git remote -v` 和 `.git/config`：这是 GitHub 仓库吗？是哪个？
- 仓库根目录的 `AGENTS.md` 和 `CLAUDE.md`：是否存在？是否已经有 `## Agent skills` 小节？
- 仓库根目录的 `CONTEXT.md` 和 `CONTEXT-MAP.md`
- `docs/adr/` 以及任何 `src/*/docs/adr/` 目录
- `docs/agents/`：本技能之前产出是否已存在？
- `.scratch/`：是否已在使用本地 Markdown 问题跟踪器约定
- `triage` 技能是否已安装？（旁边是否有 `triage` 技能目录，或可用技能中有 `triage`。）这决定 B 部分是否执行。
- 单体仓库信号：`pnpm-workspace.yaml`、`package.json` 中的 `workspaces` 字段，或自带 `src/` 的 `packages/*`。这些只出现在真正的大型多包仓库中；没有它们就是单上下文，而几乎所有仓库都是单上下文。

### 2. 展示发现并提问

总结已有什么、缺什么。然后按顺序过各部分。一次只问一部分，答完再问下一部分。

每部分先给出推荐答案，让用户一个词就能接受。只有在选择确实会分叉时才加一句解释；探索阶段已经能定论的部分直接跳过（未安装 `triage` 时跳过 B 部分，没有单体仓库迹象时跳过 C 部分的提问）。

**A 部分：问题跟踪器。**

> 解释：“问题跟踪器”是本仓库记录问题的地方。`to-tickets`、`triage`、`to-spec` 等技能都要从它读写。它们需要知道该调用 `gh issue create`，还是在 `.scratch/` 下写 Markdown 文件，或遵循你描述的其他流程。请选择你实际用来跟踪本仓库工作的地方。

默认立场：这些技能为 GitHub 设计。如果 `git remote` 指向 GitHub，就推荐它。如果 `git remote` 指向 GitLab（`gitlab.com` 或自建主机），就推荐 GitLab。其他情况（或用户有偏好）提供以下选项：

- **GitHub**：问题放在本仓库的 GitHub Issues 中（使用 `gh` CLI）
- **GitLab**：问题放在本仓库的 GitLab Issues 中（使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI）
- **本地 Markdown**：问题以文件形式放在本仓库的 `.scratch/<feature>/` 下（适合个人项目或没有远端的仓库）
- **其他**（Jira、Linear 等）：请用户用一段话描述工作流；本技能会把它记录为自由文本

把选择记录在 `docs/agents/issue-tracker.md` 中。GitHub 和 GitLab 模板都带有一个“PR 作为请求入口”开关，默认为**关闭**。保持关闭，不要主动提：想把外部 PR 纳入分诊队列的用户以后可以在文件中手动打开。

**B 部分：分诊标签词汇。** 如果未安装 `triage` 技能（探索阶段已知），整个跳过，因为没安装的技能不需要标签。

如果已安装，只问一个问题：

> 是否保留默认分诊标签？（推荐：**是**）

默认值是五个标准角色，标签字符串与角色名相同：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。回答**是**就原样写入。只有当用户说否（通常是因为其跟踪器已有其他名称，例如用 `bug:triage` 表示 `needs-triage`），才收集覆盖映射，让 `triage` 使用已有标签而不是创建重复标签。

**C 部分：领域文档。** 默认采用**单上下文**（仓库根目录下一个 `CONTEXT.md` 加 `docs/adr/`）。这适合几乎所有仓库，直接写，不用问。

只有在探索发现单体仓库信号时，才提供**多上下文**选项（根目录 `CONTEXT-MAP.md` 指向各上下文的 `CONTEXT.md` 文件）。然后确认他们想要哪种布局。

### 3. 确认并编辑

向用户展示以下草稿：

- 要加入 `CLAUDE.md` 或 `AGENTS.md` 其中之一的 `## Agent skills` 块（选哪个见第 4 步规则）
- `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 和 `docs/agents/triage-labels.md` 的内容（最后一个只在安装了 `triage` 时提供）

写入前让用户编辑。

### 4. 写入

**选择要编辑的文件：**

- 如果 `CLAUDE.md` 存在，编辑它。
- 否则如果 `AGENTS.md` 存在，编辑它。
- 如果两个都不存在，问用户要创建哪一个；不要替他们选。

当 `CLAUDE.md` 已存在时绝不创建 `AGENTS.md`（反之亦然）；始终编辑已有的那一个。

如果所选文件已有 `## Agent skills` 块，就地更新内容，不要重复追加。不要覆盖周围小节的用户编辑。

该块内容：

```markdown
## Agent skills

### Issue tracker

[问题跟踪位置的一句话总结]。见 `docs/agents/issue-tracker.md`。

### Triage labels

[标签词汇的一句话总结]。见 `docs/agents/triage-labels.md`。

### Domain docs

[布局的一句话总结："single-context" 或 "multi-context"]。见 `docs/agents/domain.md`。
```

只有在安装了 `triage` 且 B 部分已执行时，才包含 `### Triage labels` 子块并写入 `docs/agents/triage-labels.md`。未安装时两者都省略。

然后以本技能目录下的种子模板为起点写入文档文件：

- [issue-tracker-github.md](./issue-tracker-github.md)：GitHub 问题跟踪器
- [issue-tracker-gitlab.md](./issue-tracker-gitlab.md)：GitLab 问题跟踪器
- [issue-tracker-local.md](./issue-tracker-local.md)：本地 Markdown 问题跟踪器
- [triage-labels.md](./triage-labels.md)：标签映射（只在安装了 `triage` 时）
- [domain.md](./domain.md)：领域文档消费规则加布局

对于“其他”问题跟踪器，根据用户描述从零编写 `docs/agents/issue-tracker.md`。

### 5. 完成

告诉用户配置已完成，以及哪些工程技能会开始读取这些文件。提醒他们以后可以直接编辑 `docs/agents/*.md`；只有想切换问题跟踪器或从零重来时才需要重跑本技能。
