## 它能做什么

`grilling` 是在动手前压测计划、决策或想法的访谈循环。它把对象画成一棵**设计树（design tree）**：每个决策分叉出挂在它下面的决策，然后逐分支访谈你，直到没有悄悄的假设剩下。

它既不是一次一问，也不是一次全问。每一**轮（round）**问完整个**前沿（frontier）**：所有前置条件已确定的决策，不多不少。如果两个问题有依赖，它们绝不会同轮出现；依赖于尚未揭晓答案的问题属于后面的轮次。你的回答确定决策，前沿向外推进，下一轮问被解锁的内容。十三个问题通常落在三轮左右，而不是十三轮。

## 何时使用它

输入 `/grilling`，[Agent](https://www.aihero.dev/ai-coding-dictionary/agent) 也会在任务合适时主动调用它。它是追问家族里唯一的模型可调用[技能 skill](https://www.aihero.dev/ai-coding-dictionary/skill)，这也是你很少亲手输入它的原因：通常是你输入过的某个技能在替你运行它。

直接输入 `/grilling` 得到的是纯访谈，别无其他。想要更多东西时：

| 你的情况 | 该用哪个 |
| --- | --- |
| 不在工作目录里工作 | [grill-me](https://aihero.dev/skills-grill-me)：同样的[会话 session](https://www.aihero.dev/ai-coding-dictionary/session)，挂在一个 Agent 绝不会自己触发的名字下 |
| 在工作目录里 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)：同样的会话，边问边写 `CONTEXT.md` 和 ADR |
| 大到一次会话装不下的工作 | [wayfinder](https://aihero.dev/skills-wayfinder)：它画地图，在决策工单里跑追问 |
| 光靠聊解决不了的问题：长什么样、手感如何 | [prototype](https://aihero.dev/skills-prototype)：先做一次性版本，再回来 |
| 你自己的技能需要一段访谈 | 从里面调用 `/grilling`，别再写一套访谈 |

## 轮次、前沿和谁拍板

三个概念撑起整个技能。

**设计树（design tree）**是对访谈对象的建模：决策上挂着决策。**前沿（frontier）**是所有前置条件已确定的决策集合：目前唯一能诚实提问的部分。**轮次（round）**是一个完整的前沿，问完并答完。

一轮之内每个问题都是固定形状：`❓` 后面是编号和标题，然后是正文，然后 Agent 的推荐答案单独占一个 `➡️` 行。正因为如此，整轮可以用编号回答（“1 同意，2 选第二个，3 不同意，原因是……”），不用把问题复述一遍。这个格式有个已知粗糙点：推荐意见有时*反对*问题的字面表述，这时同意推荐就等于对问题回答“否”。撞上时就回答推荐意见，并说明。

另一半设计是事实与决策的分工。事实是技能自己的活：当前沿问题需要[环境 environment](https://www.aihero.dev/ai-coding-dictionary/environment)能确定的东西，它派[子智能体 sub-agent](https://www.aihero.dev/ai-coding-dictionary/subagent)去查，而不是问你。它不会为此阻塞；只有下游依赖正在查的东西的问题才等。决策是你的，必须等你拍板。跑 `grilling` 的 Agent 自己替你做了决策，就是违规，不是灵活解读。当前沿为空会话结束，它在你确认达成共识之前不会按约定动手。

诚实的局限：前沿是 Agent 的判断，不是算出来的图。它可能把两个问题放进同一轮，之后才发现前一个答案本该改变后一个。这没有防护，只能你指出来，受影响的分支在下一轮重开。

## 什么在这里讲，什么在上层讲

本页只讲机制。人们最常问的东西在上一层。

| 问题 | 在哪里回答 |
| --- | --- |
| 树、前沿、轮次、问题格式、事实与决策 | 这里 |
| 会话该多长、聊解决不了的问题怎么办、如何避免一路点头 | [grill-me](https://aihero.dev/skills-grill-me) |
| `CONTEXT.md` 写什么、什么变成 ADR | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |

## 常见问题

**能回到一次一问吗？**
可以，而且很多用户就这么用。在全局 `CLAUDE.md` 里加这一行：

```
When grilling, ask one question at a time.
```

按轮提问的默认值确实有争议。读得慢的用户、用第二语言的用户、拿顺序格式当专注脚手架的用户，都反馈一次一问的节奏更适合他们，这个退出选项是被支持的，不是被容忍的。

**`/batch-grill-me` 去哪了？**
合进这个技能了。按轮提问曾作为独立技能短暂发布，后来搬进 `grilling` 本体，所以所有建在该原语上的东西（`grill-me`、`grill-with-docs`、`triage`、`wayfinder`）一次全拿到。没有 `batch-grill-me` 可装，也没有独立的顺序版技能；上面那行 `CLAUDE.md` 配置就是回到一次一问的办法。

**整轮一起问，不会丢掉我前面答案本该带出的新问题吗？**
这是对轮次设计最常见的质疑，前沿正是答案：一轮里只放互不依赖的问题，所以本轮内的任何答案都不会推翻本轮内的另一个问题。答案依然重塑下游的一切：下一轮是重新算的，不是预写好的。你失去的东西比“一次全问”暗示的要小，比零要大：见上面前沿的局限。

**它问完就开始构建了。**
确认门正是为此存在的：当前沿清空时技能还没结束，你说达成共识了才结束。偏弱偏快的[模型 models](https://www.aihero.dev/ai-coding-dictionary/model)还是会打破它；低投入或非前沿模型上报告最多，它们把“访谈到共识”压成几个问题加一份大纲。如果你的模型这样，可靠修法是在自己的 `AGENTS.md` 或 `CLAUDE.md` 里加一行，告诉 Agent 未经允许不得实现。

**它自己回答了问题，没问我。**
这是某次运行的 bug，不是设计行为，事实与决策分离正是为修它而写的。它最常出现在别的技能以“解决这张工单”的框架运行 `grilling` 时，周围的任务读起来像继续走的许可证。同样的约束也是没有异步模式的原因：有人想要一个读 GitHub issue 然后贴一份综合决策纪要的变体，那是另一个技能，因为没人回答的追问会话产出的是 Agent 的意见，而不是你的。

**能限制问题数量吗？**
不能，设上限是有意不做的事。有的计划要三个问题，有的要五十个；固定天花板要么截断难例，要么在简单例上显得随意。大白话指挥才是预期的控制方式：让它收尾，或停下并接受当前计划。会话特别长时，原因通常是范围太大；把工作拆开逐块追问。

**我只装了 `grill-me`，什么都没发生。**
`grill-me` 是只有一行的技能，全文就是“跑一次 `/grilling` 会话”，所以它需要这个技能一起装。`grill-with-docs` 同理，还额外需要 [domain-modeling](https://aihero.dev/skills-domain-modeling)。整套全装就没这个问题；选择性安装就要把原语一起装上。

**`grill-with-docs` 跑了，但从没加载 `grilling`。**
真实且未修复的粗糙点，跨[运行环境 harnesses](https://www.aihero.dev/ai-coding-dictionary/harness)和模型都有报告：一个技能点名另一个技能，并不能可靠地让后者加载，`grill-with-docs` 一次点了两个。特征是会话一次全问且不附推荐：那是模型在即兴访谈，而不是在跑这个技能。直接问 Agent 是否加载了 `grilling` 和 `domain-modeling`，通常能救回来。

## 达到这些就说明它正常工作

- 一轮以编号列表到来，每个问题附带单独 `➡️` 行的推荐，你可以整轮按编号回答。
- 同轮内没有问题需要先答同轮的另一个问题。
- 后面的轮次问出第一轮问不出的东西。
- 它去查事实（读文件、派子智能体），而不是问你本可查到的东西。
- 后台的研究不阻塞本轮；只有依赖它的那些问题才等。
- 结束时它停下并请你确认达成共识，而不是直接开工。
- 问题数保持高位，轮数保持低位。

## 它在整体中的位置

`grilling` 是**原语（primitive）**，不是排进计划的步骤：访谈技术的唯一可信来源，放在一处以便每个需要访谈的技能都来复用，而不是各写一套。[grill-me](https://aihero.dev/skills-grill-me) 和 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 是它的两个用户可调用前门，而 `grill-with-docs` 是主构建链的起点，在 [to-spec](https://aihero.dev/skills-to-spec) 之前。[wayfinder](https://aihero.dev/skills-wayfinder) 用它解决决策工单，[triage](https://aihero.dev/skills-triage) 用它把模糊报告追问成可做的报告，[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 在你选中候选方案深挖时用它走一遍树。拿不准哪个入口合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会帮你分流。
