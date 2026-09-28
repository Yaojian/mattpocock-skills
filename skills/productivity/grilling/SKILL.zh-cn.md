---
name: grilling
description: 围绕计划、决策或想法不留情面地追问用户。当用户想压测自己的思考，或用到任何 'grill' 触发语时使用。
---

不留情面地访谈用户，直到达成共识。把访谈对象画成一棵**设计树 (design tree)**：每个决策分叉出挂在它下面的决策。

按**轮次 (rounds)**推进。**前沿 (frontier)** 是所有前置条件已确定的决策：即*现在*不用猜没听到的答案就能问的部分。一轮问完整个前沿：每个问题编号并给出你的推荐答案。然后等用户回答后再进下一轮。

一轮的格式如下：

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

用户每轮的回答都会重塑树：确定的决策把前沿向外推，解锁依赖它们的问题。重新计算前沿，问下一轮。答案依赖本轮其他未决问题的提问属于*后面*的轮次，不属于本轮。

找*事实*是你的活，绝不是用户的。当前沿问题需要从环境（文件系统、工具等）确定的事实时，派子智能体去查；凡是你自己能查到的，都不要问用户。不要为此阻塞：正在进行的探索是一个未确定的前置条件，所以只有它下游的问题等子智能体汇报；前沿的其余问题现在就问。*决策*是用户的：逐个摆给他们并等待。

当前沿为空时会话结束：设计树的每个分支都走过，没有悄悄的假设剩下。在用户确认达成共识之前，不要按约定动手。
