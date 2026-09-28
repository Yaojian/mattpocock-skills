## 它的作用

`implement` 负责构建已经定下来的工作。你把它指向一张[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)、一份[规格说明 (spec)](https://www.aihero.dev/ai-coding-dictionary/spec)，或者你们刚在对话中敲定的计划，它就会写代码，在接缝处用 [tdd](https://aihero.dev/skills-tdd) 驱动开发，边写边做类型检查，最后运行 [code-review](https://aihero.dev/skills-code-review)，并提交到当前分支。

它从不重新打开计划。没有访谈，没有澄清回合，不会提议另一种方案。上游定下来的是什么就是输入，这个技能的全部工作就是把它变成一次提交。这正是它和在全新[智能体 (agent)](https://www.aihero.dev/ai-coding-dictionary/agent)面前输入“构建这个”的区别：后者会在构建的同时欣然重新设计。

## 什么时候用它

由你输入 `/implement` 亲自调用：智能体不会自行调用它。它自带 `disable-model-invocation: true`，所以其他技能也无法调用它。[ask-matt](https://aihero.dev/skills-ask-matt) 或 [to-tickets](https://aihero.dev/skills-to-tickets) 所说的“然后每张工单 `/implement`”，都是说给你听的指令，不是智能体会主动做的事。

工作目前落在什么位置，决定了这是不是合适的技能：

| 工作处于… | 该用 |
| --- | --- |
| 跟踪器上的一张工单 | `/implement #42`，每个[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)一张工单，工单之间[清空 (clearing)](https://www.aihero.dev/ai-coding-dictionary/clearing)上下文 |
| 一份规格说明，尚未拆分，构建会跨会话 | 先 [to-tickets](https://aihero.dev/skills-to-tickets)，再每张工单 `/implement` |
| 一份规格说明，且构建规模小 | 直接对着规格说明 `/implement` |
| 只存在于你刚聊完的对话里，且规模还小 | 就在同一窗口直接 `/implement` |
| 还没写在任何地方 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，如果没有代码库则用 [grill-me](https://aihero.dev/skills-grill-me) |
| 一个想用测试先行的具体行为，没有规格说明 | 直接 [tdd](https://aihero.dev/skills-tdd) |
| 已经构建完，想检查一下 | 直接 [code-review](https://aihero.dev/skills-code-review) |

同会话的情形值得单独点名，因为技能自己的第一行没有覆盖它。`SKILL.md` 写的是“规格说明或工单”，这会引导[模型 (model)](https://www.aihero.dev/ai-coding-dictionary/model)去找一个并不存在的文件。如果计划只存在于当前对话线程里，调用时就直接说明。

## 前提条件

`implement` 会提交到你所在的分支。它不创建分支，也不询问。开始前先确认你在想要的分支上。

如果工单来自 [to-tickets](https://aihero.dev/skills-to-tickets)，它们所在的跟踪器是由 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 配置的。`code-review` 在收尾时会读同一份配置来找到源头规格说明。

## 单次运行做什么

一次运行有五个节拍，按顺序执行：

1. 读工单或规格说明，找出接缝。
2. 在事先约定的接缝处用 [tdd](https://aihero.dev/skills-tdd) 驱动开发，一次一个红绿小步。
3. 频繁做类型检查，过程中运行单个测试文件。
4. 最后运行一次完整测试套件。
5. 运行 [code-review](https://aihero.dev/skills-code-review)，然后提交到当前分支。

一次运行只覆盖一张工单。[to-tickets](https://aihero.dev/skills-to-tickets) 产出的工单是穿针引线式的纵向切片，每张的大小都适合装进一个全新的[上下文窗口 (context window)](https://www.aihero.dev/ai-coding-dictionary/context-window)，所以预期的节奏是：清空上下文，实现一张工单，提交，再清空。每张工单自包含，这也是上一张工单的上下文可以丢弃的原因。

## 事先约定的接缝

这个技能运转的核心概念是**接缝 (seam)**：在不深入内部的情况下观察行为的公开边界。测试就住在接缝处。在写任何代码之前先约定接缝，测试才能持久，因为底下的实现可以重写而不必搬动测试。

“事先约定”四个字有实际分量，也是这个技能最弱的一环。`implement` 内部并不约定接缝。约定接缝的是 `tdd`，它拒绝在未经确认的接缝处写测试。所以实践中，约定要么发生在上游的规格说明里，要么发生在单次运行的头几个来回里。如果两处都没有，这个前置条件就永远不触发，运行会悄然变成“直接写代码”。在规格说明里写明接缝，正是防止这种情况的办法。

## 常见问题

**跑完了，但我的工单还开着，验收标准也没勾。**

对，这是预期的。`implement` 没有收尾步骤。它止于提交，从不碰工作项，在 GitHub Issues 和本地 markdown 跟踪器上都已确认如此，所以这不是跟踪器集成问题。它也不会处理 `code-review` 的发现，也不会勾掉源头 issue 上的 `- [ ]` 框。请自己关闭工单、自己核对标准。在依赖链上这最要命，因为 `to-tickets` 把前沿定义为阻塞项全部关闭的工单。如果什么都不关闭，就没有任何工单会显示为解除阻塞。

**能把它一次性指向所有工单，或者并行跑多个吗？**

不能。一次调用，一张工单。跨工单队列的批量分发和[子智能体 (subagent)](https://www.aihero.dev/ai-coding-dictionary/subagent)扇出都被反复要求过，两者都不存在。在同一个检出目录里并排跑多个 `/implement` 会话，比“不支持”更糟：一份现场报告描述了同一个下午横跨三个 issue，一个会话里的 `git commit --amend` 落到了另一个会话的提交上，stash 从 `refs/stash` 里消失，提交落到了错误的分支。这些会话共享同一个工作目录、同一个暂存区、同一个 HEAD。Git worktree 是社区的变通办法，注意 `refs/stash` 在 worktree 之间也是共享的，所以光用 worktree 修不好 stash 的问题。如果今天想要并行，只能自己拼装。

**能让它开 pull request 而不是直接提交吗？**

没有内置支持。它直接提交到当前分支，几个人都觉得太急：代码在他们有机会验证之前就落地了。没有配置开关，也没有 PR 模式。人们会在调用时覆盖（“提交到一个分支并开 PR”），或者改自己本地的技能副本。

**`code-review` 说它看不到我的改动。**

`code-review` 评审的是 `git diff <fixed-point>...HEAD`，不含已暂存和工作区的改动。`implement` 在提交前运行它，所以除非已经存在中间提交，否则那个 diff 里没有任何可评审的内容。好几个人报告过这个问题，双方都还没修。先提交，再对照你切出分支的基点做评审。

另外，有些人本来就不想要运行内置的评审，因为智能体评审自己刚写的代码会偏向自己的方案。在全新会话里对照固定基点运行 [code-review](https://aihero.dev/skills-code-review) 是合理的替代方案，这也是该技能把两个评审轴放到不同子智能体里跑的原因。

**一张工单烧了 15 万 token，是我用错了吗？**

更可能是工单太大，而不是技能用错。一次运行要做代码库探索、每个接缝一轮红绿循环、一次全量套件加一次评审，所以非 trivial 的工单超过 10 万 [token](https://www.aihero.dev/ai-coding-dictionary/token) 属于正常，不是出故障的信号。杠杆在上游：在 [to-tickets](https://aihero.dev/skills-to-tickets) 里把工单拆到每张适合一个全新窗口。如果单张工单持续爆掉，就拆分它，而不是调高[投入档位 (effort)](https://www.aihero.dev/ai-coding-dictionary/effort)。

**在全新会话里 `/implement #2`，却做了完全不相关的事。**

`#2` 会对照智能体能看到的某个编号列表来解析，在全新会话里那可能是 todo 文件、checklist 或其他工作清单，而不是配置好的跟踪器。解析是自信的而非 fail-closed（失败封闭式）的，所以直到开工你都看不出错了。请传完整引用（issue URL 或 `owner/repo#2`），并让它开工前先把标题复述回来确认。

## 生效的标志

- 会话开场先读工单或规格说明，并复述要构建什么，而不是问你要构建什么。
- 你能在执行轨迹里看到一次真实的 `/tdd` 调用，而不是 diff 里凭空出现测试。
- 运行中反复做类型检查和单个测试文件，完整套件在接近结束时跑一次。
- 不用你催着继续，运行自己走到当前分支的一次提交。
- diff 就是一张工单的量：穿过每一层的纵向切片，而不是几张工单扫在一起。

## 它在流程中的位置

`implement` 是主链条中的构建步骤，倒数第二环：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它的邻居是 [to-tickets](https://aihero.dev/skills-to-tickets)（产出它消费的工单，并声明决定顺序的阻塞边）、[tdd](https://aihero.dev/skills-tdd)（它在每个接缝内部驱动）和 [code-review](https://aihero.dev/skills-code-review)（它在提交前运行）。它位于规划技能下游，并信任它们。它不重新校验拿到手的东西是什么形状，所以结构糟糕的地图或横向分层的工单会按原样被构建出来。

这正是 [wayfinder](https://aihero.dev/skills-wayfinder) 在 [to-spec](https://aihero.dev/skills-to-spec) 处汇入主链，而不是把地图直接灌进 `implement` 的原因。只有当工作量确实很小时，才从地图直达 `implement`。

当你不确定自己在哪个流程里时，[ask-matt](https://aihero.dev/skills-ask-matt) 是覆盖全套技能的路由器。
