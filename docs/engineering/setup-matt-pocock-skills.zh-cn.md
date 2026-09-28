## 功能简介

`setup-matt-pocock-skills` 回答关于一个仓库的三个问题：issue 存在哪里、分诊标签叫什么、领域文档放在哪里。它把答案记录为 `docs/agents/` 目录下的 markdown 文件。

这些文件是不同仓库之间唯一变化的东西。技能本身在任何地方都完全相同；它们在运行时读取 `docs/agents/issue-tracker.md` 并照着执行。这就是为什么这套技能不绑定 GitHub，也是为什么永远不需要编辑技能文件来指向别处。用“把技能接到自定义问题跟踪器上”这样的说法调用它，只要能用程序连接，任何跟踪器都可用，而且技能本身零改动。

它是一个提示驱动 (prompt-driven) 的技能，不是确定性脚本。它会读取你的 `git remote`、已有的 `CLAUDE.md`、已有的 `CONTEXT.md`，给出它的发现请你确认，在你确认之后才写入任何内容。

## 何时使用

输入 `/setup-matt-pocock-skills` 即可调用；[智能体 (agent)](https://www.aihero.dev/ai-coding-dictionary/agent)不会自动选用它。它被有意标记为不可调用，因此其他技能也无法替你触发它。

每个仓库用一次，在首次使用其他工程技能之前运行。如果 [triage](https://aihero.dev/skills-triage)、[to-spec](https://aihero.dev/skills-to-spec)、[to-tickets](https://aihero.dev/skills-to-tickets) 或 [wayfinder](https://aihero.dev/skills-wayfinder) 开始猜测你的 issue 该去哪里，或者套用了你的跟踪器中不存在的标签，说明这里还没有配置过。项目做到一半的仓库也可以运行它；技能会读取已有的内容，不会浪费之前的工作。

## 前置条件

它会写入你运行它的仓库：

| 写入内容 | 位置 |
| --- | --- |
| `issue-tracker.md` | `docs/agents/` |
| `domain.md` | `docs/agents/` |
| `triage-labels.md` | `docs/agents/`，仅在安装了 `triage` 技能时 |
| `## Agent skills` 代码块 | 已存在的 `CLAUDE.md` / `AGENTS.md` 中的其一 |

所有产出都是已提交的 markdown。没有用户级或全局模式：配置存在于仓库中，所以每个仓库都有自己的一份。

## 三个决策

它在每一节开头给出推荐答案，已经明确的部分就跳过不再探索。大多数运行只需要两次确认即可完成。

| 决策 | 推荐方案 | 何时真正提问 |
| --- | --- | --- |
| **问题跟踪器 (issue tracker)** | 与你的 `git remote` 匹配的那一个 | 都会问：这是唯一真正的选择 |
| **分诊标签 (triage label)** | 保留五个规范名称（`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`） | 仅在安装了 `triage` 技能时 |
| **领域文档** | 单上下文：根目录下一个 `CONTEXT.md` 加 `docs/adr/` | 仅在发现 monorepo 信号时，这时会提供多上下文的 `CONTEXT-MAP.md` 方案 |

跟踪器选项：

| 选项 | issue 存在哪里 | 需要什么 |
| --- | --- | --- |
| **GitHub** | 该仓库的 GitHub Issues | `gh` CLI |
| **GitLab** | 该仓库的 GitLab Issues | `glab` CLI |
| **本地 markdown** | 本仓库中 `.scratch/<feature>/` 下的文件 | 不需要：完全不需要远端 |
| **其他 (Other)** | 你指定的任何地方 | 你用一段话描述工作流 |

前三种随技能附带模板，开箱即用。本地 markdown 是一等选项，不是降级方案：没有远端的个人项目也完全支持。有一条注意事项值得重复：如果用 GitHub，就不要用本地 markdown。它们是二选一，不是叠加使用。

“其他”也不是占位符。正因为有它，Jira、Linear、Azure DevOps 和 Beads 才能用：你描述工作流，技能把你的描述记录在 `docs/agents/issue-tracker.md` 中，下游技能照着描述执行。社区已经这样做过：有基于 [MCP](https://www.aihero.dev/ai-coding-dictionary/mcp) 的 Jira 变体、有形似 `gh` 的 Gitea CLI、有手搭的本地看板。

## 常见问题

**必须用 GitHub 吗？**

不用。GitHub、GitLab 和 `.scratch/` 下的本地 markdown 都有现成模板，其他任何跟踪器都可以走“其他”路径。这是有记录以来被问得最多的问题，大致是这些原话：*“hard locked to github”*、*“可以用 GitLab / Jira 吗”*、*“Azure DevOps 呢”*。每次的答案都一样：跟踪器是配置阶段的答案，不是技能的属性。

**更新技能后需要重新运行它吗？**

v1.1 之后有人直接问过，Matt 说要。技能自己的结束语则更宽松：它说只有换跟踪器或推倒重来时才需要重跑。两种说法都有道理，造成差异的原因也真实存在：种子模板会在版本之间变化，所以旧版本写出的 `docs/agents/issue-tracker.md` 可能和现在读取它的技能脱节。如果下游技能的行为和文档描述不一致，重跑就是成本最低的修复办法。

**它写到了 `CLAUDE.md`，但我用的是 Codex。**

已知缺口，仍未修复。文件选择规则是“`CLAUDE.md` 存在就改它，否则改 `AGENTS.md`”：它只检查哪个文件存在，而不检查当前运行的[运行环境 (harness)](https://www.aihero.dev/ai-coding-dictionary/harness)。一个留有 Claude Code 时代 `CLAUDE.md` 的仓库，会把 `## Agent skills` 代码块写到 Codex 永远不读的地方。流传中的两种变通办法是：手工把该代码块移到 `AGENTS.md`，或者以 `AGENTS.md` 为准，让 `CLAUDE.md` 只留一行指向它的说明。如果两个文件都不存在，技能会问你创建哪一个，而不是自作主张，这让期待它直接决定的用户感到困惑。

**它没有创建我的分诊标签。**

它本来就不创建。`docs/agents/triage-labels.md` 是一个*映射*：告诉 `/triage` 你的跟踪器中的哪些字符串对应五个规范角色。它不会运行 `gh label create`。在全新的 GitHub 仓库上，这些标签确实还不存在，这已经不止一次被当作 bug 提交。还有两个后续结论：

- 如果你的跟踪器本来就用规范名称，映射就是一张恒等表，无需配置。这正是预期的常见情况，不是缺失的步骤。
- [wayfinder](https://aihero.dev/skills-wayfinder) 的 `wayfinder:map` 和 `wayfinder:<type>` 标签也不会在这里创建，而且 `gh issue create --label <missing>` 会直接失败，不会自动建标签。在 GitHub 仓库上首次运行 wayfinder 之前，请手工创建它们。

**可以在这里配置其他技能的行为吗（[追问 (grilling)](https://www.aihero.dev/ai-coding-dictionary/grilling) 频率、提问格式、语气）？**

不可以。它只配置三件事：跟踪器、标签、文档布局。已经有人直接要求把它做成用户偏好的统一配置处，坚持下来的答案是技能保持有主见：*“Config is death.”*（配置即死亡。）偏好应该以普通指令的形式写在你的 `CLAUDE.md` 中，每个技能本来就会读它。

**可以把配置放在 `~/.claude` 里，而不是每个仓库提交一份吗？**

目前不行。确实有人在多仓库使用技能时提出过这个需求，目前不存在用户级模式。每个仓库都带有自己的 `docs/agents/`。

**用一个技能来配置其他技能，不是很奇怪吗？**

一项长期存在的抱怨正是这么说的，原话是：*“having a skill to set up the other skill does not feel right to me: that means the LLM is configuring its own skills.”*（让我用一个技能去配置另一个技能，感觉不对：这等于让大模型自己配置自己的技能。）这个取舍真实存在且已被承认：不做配置步骤的替代方案，是把跟踪器说明复制到每个触碰 issue 的技能里。缓解办法是产出为可检查、可编辑的 markdown：它写出的每个文件你都能阅读并手工修改，日常微调就该这样做，而不是再跑一次。

## 生效标志

- `docs/agents/issue-tracker.md` 和 `docs/agents/domain.md` 已存在；若安装了 `triage`，还有 `triage-labels.md`。
- 你的运行环境实际读取的指令文件中出现了 `## Agent skills` 小节，并用一句话指明上述每个文件。
- 它推荐的跟踪器与你实际使用的远端一致，标签字符串与跟踪器中真实存在的标签一致。
- 此后 `/to-tickets` 不再问你 issue 发哪里，`/triage` 会套用标签而不是自创标签。
- 技能文件本身没有任何变化。如果配置过程改了某个 `SKILL.md`，说明出问题了。

## 在流程中的位置

`setup-matt-pocock-skills` 是工程流程的**一次性配置**，是其他步骤默认已成立的前置条件，而不是链条中的一步。它的邻居都是它的读者：[triage](https://aihero.dev/skills-triage) 套用这里写下的标签词汇；[to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets) 发布到这里指定的跟踪器；[wayfinder](https://aihero.dev/skills-wayfinder) 则读取同一份跟踪器文件的“Wayfinding operations”小节，以知道地图和子[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)如何存放。它记录的领域文档布局由 [domain-modeling](https://aihero.dev/skills-domain-modeling) 在之后填充：`CONTEXT.md` 和 ADR 都是惰性创建的，只在某个术语或决策真正明确时才写，因此配置完后仓库是空的属于正常状态。要找下一步该用哪个技能，由 [ask-matt](https://aihero.dev/skills-ask-matt) 为整套技能指路。
