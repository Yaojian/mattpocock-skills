## 它能做什么

`triage` 会逐一处理项目跟踪器上的 issue，让每个 issue 走过一个小型的状态机，状态机由**分流角色（triage roles）**组成（一个类别角色和一个状态角色），最终留下一份 Agent 可直接执行的简报（agent-ready brief）、一个给报告人的具体问题，或者一个附有已记录原因的已关闭 issue。

它只处理**不是你创建的 issue**。原始的 bug 报告、外部发来的功能请求、突然出现的外部 pull request：这些都是从外部落到跟踪器里的工作，保持着报告人留下的原样。[工单 Tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 如果是由 [to-tickets](https://aihero.dev/skills-to-tickets) 生成的，天然就是 Agent 可直接执行的，再跑一遍 `triage` 充其量也是白费力气。规则很明确：`/triage` 只用于外部流入的 issue，不用于你自己创建的 issue。

它和手工打标签的第二个区别是：它先给出建议并等待。它会告诉你它的类别判断和状态判断及理由，以及它在代码库中的发现，在你明确指示之前不会做任何修改。

## 何时使用它

输入 `/triage` 并用自然语言描述你的需求来调用它。[Agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自动调用它。比如“看看有哪些需要我关注的”、“我们看下 #42”、“把 #42 移到 ready-for-agent”。

| 你的情况 | 该去哪里 |
| --- | --- |
| 跟踪器里堆满了别人提交的原始报告 | `/triage` |
| 只有一个粗略的想法，还没写下来 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 有一段已经讨论充分的对话，想转成[规格文档 spec](https://www.aihero.dev/ai-coding-dictionary/spec) | [to-spec](https://aihero.dev/skills-to-spec) |
| 有一份规格文档，想拆成 Agent 可执行的工单 | [to-tickets](https://aihero.dev/skills-to-tickets) |
| 有一个已确认的 bug，需要找根因而不是打标签 | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

## 前提条件

`triage` 需要读写你的 issue 跟踪器，所以 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须先配置好该跟踪器及其标签词汇表。下面出现的角色名是**标准名（canonical）**；你跟踪器里的实际标签字符串可能不同，setup 提供的就是两者之间的映射。如果你的跟踪器本来就完全使用标准名，那就无需映射，也无需额外配置。

跟踪器配置还会决定外部 pull request 是否算作一种需求来源，以及谁算作外部人员。该开关默认关闭，也不再是 setup 中的提问项，如果你想把 PR 纳入范围，请去 `docs/agents/issue-tracker.md` 中打开它。

## 状态机

每个经过分流的条目最终恰好带有一个类别角色和一个状态角色。两个类别：`bug`（有东西坏了）和 `enhancement`（新功能或改进）。五个状态：

| 状态 | 含义 |
| --- | --- |
| `needs-triage` | 需要你评估。未打标签的 issue 通常先落到这里。 |
| `needs-info` | 等待报告人回复。对方回复后回到 `needs-triage`。 |
| `ready-for-agent` | 已完整描述，并附有 Agent 简报。[AFK](https://www.aihero.dev/ai-coding-dictionary/afk) Agent 可以直接接手。 |
| `ready-for-human` | 同样的简报，外加说明为什么这件事无法委派：需要判断力、需要外部访问、需要手工测试。 |
| `wontfix` | 已关闭，并记录了原因。 |

词汇表就这么多，“恰好一个状态角色”这个不变式让查询保持简单。这也是[技能 skill](https://www.aihero.dev/ai-coding-dictionary/skill) 中被问得最多的地方：用户要求为“已明确但被另一个 issue 阻塞”的工作增加第六个状态，为“延期（deferred）”的未来触发型工作增加状态，以及为终态增加 `implemented` 状态。这些都没有发布。详见下面的常见问题。

`wontfix` 分三种情况，区别很重要，因为其中只有一种会写入知识库：

| 为什么关闭它 | 会发生什么 |
| --- | --- |
| 已经实现 | 留一条评论，指出现有功能的位置。不会写入 `.out-of-scope/`，因为这是已构建的功能，不是被拒绝的功能，写进去会污染去重检查。 |
| 被拒绝的 bug | 礼貌解释后关闭。 |
| 被拒绝的 enhancement | 在 `.out-of-scope/` 中建一个文件，在关闭评论中链接它，然后关闭。 |

`.out-of-scope/` 是每个被拒绝的**概念**对应一个 markdown 文件，而不是每个 issue 一个文件，写法是简短的设计文档而不是数据库行：拒绝了什么、为什么拒绝、有哪些 issue 问过它。`triage` 在评估任何内容之前会先读完整个目录，并按概念而非关键词匹配，所以 “night theme” 能匹配到 `dark-mode.md`。一旦命中，它会展示旧的决定，并问你现在是否依然这么看，而不是从头重新争论这个需求。

## 先验证，再写简报

在任何[追问 grilling](https://www.aihero.dev/ai-coding-dictionary/grilling)之前，`triage` 会先核实报告的主张是否成立。对于 bug，它按报告人的步骤复现。对于 PR，它检出分支并运行相关测试。然后报告三种结果之一：已确认，并给出代码路径；未能复现；或信息不足到无法尝试，而这本身就是最强的 `needs-info` 信号。

它在同一轮中还会对代码库做另外两项检查：**冗余（redundancy）**（是否已经实现，按领域概念搜索，而不是按报告人的措辞搜索）和**历史拒绝（prior rejection）**（`.out-of-scope/` 是否已经拒绝过）。这两项检查都很便宜，一旦命中就会得到 `wontfix`。

所有这一切都是为了做好一件产物：**Agent 简报（agent brief）**，即 issue 进入 `ready-for-agent` 时发布的格式化评论。一旦发布，简报就是约定，原报告只算背景。简报追求**持久耐用（durable）**而非精确，因为一个 issue 可能在 `ready-for-agent` 里放几周，而代码一直在变。所以简报只写类型、签名和行为约定，从不写文件路径或行号。经过确认的复现过程写出的简报，远比猜测写出的简报可靠。

## PR 就是附带代码的 issue

在跟踪器把外部 pull request 视为需求来源的地方，PR 走同样的状态机，用同样的类别、同样的状态、同样的流转。状态只是对照 diff 来理解：`ready-for-agent` 表示已附上简报，Agent 应该对现有代码采取下一步；`ready-for-human` 表示可以由人工合并了。PR 上的简报描述的是“对现有 diff 还剩什么要做”，而不是“如何从零构建”。

发现阶段只展示*外部* PR，因为协作者正在开发的分支不属于分流工作。该过滤只用于发现阶段，直接点名某个 PR 则无论作者是谁都会处理。一个已知的粗糙点：GitHub 模板中列出外部 PR 的命令要求 `gh pr list` 返回 `authorAssociation` 字段，而 `gh` 并不提供该字段，所以该命令会直接失败（[#468](https://github.com/mattpocock/skills/issues/468)）。

## 常见问题

**我跑了 `/to-spec` 和 `/to-tickets`，现在那些工单堆在那里没人分流。我要对它们跑 `/triage` 吗？**
不用。它们已经是 Agent 可执行的，因为 `to-tickets` 在发布时就会打上 `ready-for-agent` 标签，正是为了让 AFK 执行器无需再过一遍就能接走。遇到这个问题的用户是跑完规格流程后，在输出上看到了 `needs-triage`，发现 AFK 执行器忽略了所有内容。`triage` 是外部流入工作的入口；规格流程是你自己发起工作的通道。它们在 `ready-for-agent` 处汇合，而不是在此之前。

**现在有了 `to-spec` → `to-tickets` → `implement` 流程，`triage` 还有用吗？**
只有当你有外部流入的工作时才有用。`triage` 比那条主链更早存在，职责也不同：它是处理别人提交的报告的通道。如果跟踪器里的所有内容都来自你自己的规划，你很少会打开它。如果你维护公开项目，或者团队成员不断给你提 bug，它就是前门。主要用途是接收外部贡献者 issue 的开源仓库。

**Agent 试图打 `ready-for-agent` 标签，`gh` 说该标签不存在。**
已知 bug（[#616](https://github.com/mattpocock/skills/issues/616)）。`setup-matt-pocock-skills` 把标签词汇表写进 `docs/agents/triage-labels.md`，但不会在你的跟踪器里创建这些标签。请自己创建一次五个状态标签和两个类别标签，用 `gh label create` 或跟踪器的界面操作，之后就正常了。该 issue 下挂了一个社区修复分支，尚未合并。

**五个状态不够用：阻塞中、延期、已实现怎么办？**
这是该技能被提得最多的缺口，有三种形态。已完整描述但在等另一个 issue 关闭的 issue（[#139](https://github.com/mattpocock/skills/issues/139)），报告人的抱怨是那里的 `ready-for-agent` “严格来说没错”但有误导性，Agent 接过去会直接撞墙。由未来触发条件控制的、有意向但暂时不可执行的延期工作（[#297](https://github.com/mattpocock/skills/issues/297)）。以及“已实现、待验证”的终态，没有它，AFK 执行器可能把已完成的工单重新排队。Matt 已认可阻塞场景是真实存在的，名称还在犹豫（`blocked` 还是 `paused`）。这些都没发布。大家常用的变通方法是，在类别之外再加一个仓库本地的额外标签，让标准状态槽位上放一个诚实的值，代价是技能感知不到那个额外标签。有个社区衍生版走得更远，加了 `needs-slicing`、`tracking` 和工作量标签。那能用，但那是他们的，不是技能自带的。

**它和 `/diagnosing-bugs` 有什么区别？**
这里的验证步骤是有意做浅的（只回答“这是真的吗，大概在哪里”），不是找根因。当一个 bug 按报告人的步骤在几分钟内复现不了，诚实的做法是标 `needs-info`，或者如果你想现在就追查，就用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)。两个技能的文本目前都没有提到对方，有用户发现了这个断层，目前仍未处理。

**我能把它指向整个 backlog 让它全自动跑吗？**
你可以提，但要注意它读了什么。“展示需要关注的内容”那一轮是便宜的列表调用，用于*挑选*，你挑一个，它再对你挑中的那一个收集完整[上下文 context](https://www.aihero.dev/ai-coding-dictionary/context)。如果你让它一次处理二十个 issue，Agent 可能悄悄把那份便宜列表当作证据库，而列表只返回 issue 正文，不返回评论。有用户正好踩中：三个 issue 明明已有评论写着“已修复，建议关闭”，结果都收到了全新的 Agent 简报。如果你想批量处理，请明确要求每个 issue 都要逐个读评论。

**它支持 Linear 或 GitHub Issues 之外的工具吗？**
支持，跟踪器是配置项，不是写死的假设，有人在 Linear（通过 `linear` CLI）、GitLab 和 `.scratch/` 下的纯 markdown 文件上跑它。常见分工是 Linear 管 issue 和规划，GitHub 管代码和 PR：提到“issue tracker”的技能对应 Linear，提到“PR”的技能对应 GitHub。在本地 markdown 跟踪器上有个未修复的模板 bug，生成的文件可能把验收标准带两遍，顶层一次，Agent 简报里又一次（[#200](https://github.com/mattpocock/skills/issues/200)）。

## 达到这些就说明它正常工作

- 它经手的每个条目最终恰好有一个类别角色和一个状态角色，不多不少，状态之间不冲突。
- 它给出带理由的建议后停下来等你，而不是直接改标签走人。
- bug 已复现，或 PR 已检出并运行过，之后才进入 `ready-for-agent`。
- 它写的简报只写类型和行为，不含文件路径和行号。
- 六个月前被拒绝的需求又出现了，它能指出来并引用旧理由，而不是当新需求重新分流。
- 它发的每条评论都以 `> *This was generated by AI during triage.*` 开头。

## 它在整体中的位置

`triage` 是**入口（on-ramp）**，不是主链中的一步。主流程从你自己的想法开始（追问、规格、工单、实现、评审），`triage` 是给外部流入工作的并行通道。它在同一个地方汇合：打上 `ready-for-agent` 标签并附有简报的 issue，[implement](https://aihero.dev/skills-implement) 会像接 [to-tickets](https://aihero.dev/skills-to-tickets) 的工单一样接走它。当一个需求在写简报前需要先打磨清楚，`triage` 会把[追问 grilling](https://aihero.dev/skills-grilling)和[领域建模 domain-modeling](https://aihero.dev/skills-domain-modeling)一起跑，一轮一轮地问，定下来的决策随手记进 `CONTEXT.md` 和 ADR。如果你不确定自己在哪条通道上，[ask-matt](https://aihero.dev/skills-ask-matt) 会帮你分流。
