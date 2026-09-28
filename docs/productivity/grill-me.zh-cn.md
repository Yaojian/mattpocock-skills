## 它能做什么

`grill-me` 接过一个**粗糙想法（loose idea）**，对你访谈直到你可以为它拍板。开始时不需要成型的计划：产出一份计划正是这次[会话 session](https://www.aihero.dev/ai-coding-dictionary/session)的任务。它按**轮次（rounds）**提问：每一轮问完整个**前沿（frontier）**（所有前置条件已被你确定的问题），所以你永远不会被问到依赖于还没听到的答案的问题。

它是**[无状态的 stateless](https://www.aihero.dev/ai-coding-dictionary/stateless)**。不写文件，不留工作区。它留下的唯一东西，是在你自己脑子里变锋利的想法。

## 何时使用它

输入 `/grill-me` 来调用；[Agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自动调用它。请在**全新对话**里启动它，不要叠在 Agent 已经写好的计划上。

只要有个值得认真的想法（功能、产品方向、业务决策、一篇写作），就尽早用，远在你想清楚它涉及什么之前。模糊不是等待的理由，模糊正是这次会话要吃掉的东西。如果你已经能精确描述它，就不需要追问了。

三个追问技能用哪个，看你面前有什么：

- **任何事、任何地方**：`grill-me`。不需要仓库，不写文件，主题也不必是代码。
- **有代码库要对齐**：[grill-with-docs](https://aihero.dev/skills-grill-with-docs)。同样的访谈，但是[有状态的 stateful](https://www.aihero.dev/ai-coding-dictionary/stateful)：它读你的代码，并把学到的东西记进 `CONTEXT.md` 和 ADR。
- **大到装不进一次会话**：[wayfinder](https://aihero.dev/skills-wayfinder)。它把工作画成地图，在地图里跑追问会话。

关掉[计划模式 plan mode](https://www.aihero.dev/ai-coding-dictionary/agent-mode)。计划模式会催 Agent 赶快产出计划，正好和保持追问相反。

## 这是对话，不是审问

技能负责提问，但**你**拥有范围。这是人们最容易错过的一点，也是把想法变成决策的会话和产出自信废话的会话的分界线。

失败模式是**被动**：四十个问题一路“同意、同意、同意”，最后拿出一份 Agent 写、你点头的计划。感觉很高效，因为很长。其实什么都没决定，结果还带着它没挣到的确定性。

主动意味着掌舵。精度不够的问题就顶回去。范围跑偏就指出来。不知道就说“我不知道”，而且是真不知道。这个技能是给工程师助攻的，不是替代工程师的：产出的质量取决于你的回答质量，而不是问题数量。

反向错误真实存在但更少见：在访谈里待太久，永远写不了代码。

## 可追问与不可追问

有些问题靠聊能解决。有些不行，再多追问也到不了。

“一个长表单还是三个页面？”和“这个交互应该是什么感觉？”是**不可追问的**：它们需要有个东西让你反应。撞上这种问题就停下追问。用 [prototype](https://aihero.dev/skills-prototype) 做个一次性版本，看一眼，回来用一句话回答。

硬聊不可追问的问题是会话膨胀的源头。Agent 反复换说法，你反复猜，范围膨胀去填满不确定性。

## 达到这些就说明它正常工作

- 你提出过反对。如果一次会话里你一次都没顶回去，那次会话你不需要。
- 问题分几轮到来，而不是一次长 drip，后面的轮次明显建立在你前面说过的话上。
- 你走到了没预料到的地方，因为某个问题翻出了你一直在隐式做的决策。
- 结束时你能向不在场的人为每个选择辩护。

## 常见问题

**会有多少问题，怎么知道何时结束？**
数轮次，不数问题。四轮四十六个问题是普通会话。当前沿为空时结束：每条分支都走过，没有悄悄的假设剩下。

**它问了我两百个问题。哪里出问题了？**
通常是范围太大。先让 Agent 把工作拆小，逐块追问。超长会话还会滑进**[低效区 dumb zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)**，[上下文窗口 context window](https://www.aihero.dev/ai-coding-dictionary/context-window)满到问题质量下降。

**能回到一次一问吗？**
可以。在全局 `CLAUDE.md` 里加这一行：

```
When grilling, ask one question at a time.
```

**如果我真的不知道答案怎么办？**
直说。“我不知道”是有效回答，而你答不上来的问题通常是去做原型的信号，不是去猜的信号。

**写规格前要开新会话吗？**
不要。这次会话的价值正是你刚建好的[上下文 context](https://www.aihero.dev/ai-coding-dictionary/context)。把同一个对话直接交给 [to-spec](https://aihero.dev/skills-to-spec)。

**模型重要吗？**
比大多数技能重要。追问依赖[模型 model](https://www.aihero.dev/ai-coding-dictionary/model)自己对系统会怎么坏的直觉，所以把最好的模型给它。实现主要跟着上下文走，用便宜模型也扛得住。

## 它在整体中的位置

`grill-me` 是**随处可跑、什么都能问的独立件**。无状态让它便携：无仓库、无工作区、无配置，也不假设想法和软件有关。有人拿它问业务决策、写作、下一步做什么：一切在脑子里坐不住的东西。

便携性正是它和 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 的全部区别，后者跑同样的访谈，但读代码库做对齐，并把学到的记成 `CONTEXT.md` 和 ADR。两者都站在[追问 grilling](https://aihero.dev/skills-grilling)这个原语上；`grill-me` 是用户可调用的前门，两手空空也能进。

如果追问完发现它确实是软件，把同一个对话交给 [to-spec](https://aihero.dev/skills-to-spec)，继续走进构建流（这是选项，不是这个技能的目的）。拿不准哪条流合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会帮你分流。
