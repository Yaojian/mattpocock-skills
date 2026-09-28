## 它能做什么

`wait-what` 是当一条消息没讲明白时你输入的话。然后[智能体 (agent)](https://www.aihero.dev/ai-coding-dictionary/agent)会把刚说的内容换个讲法重讲一遍。它补上你缺的背景，用简明英语 (plain English) 写，并使用你项目 `CONTEXT.md` 里的词汇。

这个技能只有三行。这是设计，不是没写完的草稿。治啰嗦的技能败于变长：四百行的简洁技能还是治不好[模型 (model)](https://www.aihero.dev/ai-coding-dictionary/model)的啰嗦，因为模型读的是体量，而不是恳求。这个技能只带一个精确的引导词 (leading word)，别无其他。

## 何时使用

输入 `/wait-what` 即可调用。智能体不会自行调用，也不该自行调用。只有你知道自己哪里没跟上。

一发现自己在略读就用它。智能体开始用自造的行话、堆五个缩写，或者解释一个你从没见过前提的决策。它修复的是你所在的这段对话。如果想让行话根本不出现，用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它在前面先建好共同语言。

## 名字就是机制

引导词是 **wait**。“简洁点”是关于智能体输出的指令，模型听到后靠删字执行，结果你更跟不上了。**Wait** 是关于**你的状态**。它宣告理解在这里失败了。听到“简短点”的智能体写电报，听到“等等，我没跟上”的智能体会退回去解释。

区别就是整个技能。每种流行的治啰嗦方法都在命名**输出**：`/tldr`、`/no-fluff`、`/talk-normal`。模型矫枉过正，写出更短但同样难懂的穴居人语。命名**听者**则一次要到两半：更少的字**加**你缺的背景。

技能说的是重讲**那个意思 (that)**，而不是“上一条消息”。让你没跟上的通常不止一段，所以由智能体决定回溯多远。

## 它接入你已有的语言

正文复用你的全局 `CLAUDE.md` 和项目 `CONTEXT.md` 里已有的引导词。ASD-STE100 简化技术英语定语气，统一语言 (ubiquitous language) 定名词。技能、`CLAUDE.md` 和 `CONTEXT.md` 指向同一组[词元 (token)](https://www.aihero.dev/ai-coding-dictionary/token)，所以调用它不是新指令，而是提醒智能体履行它已经答应过的那一条。

如果你没有 `CONTEXT.md`（也没有 `CONTEXT-MAP.md` 指向当下上下文对应的那份），技能照样能用，只是少了领域词汇那一半效果。

## 生效标志

- 重讲版本**更短且更清楚**，而不是更短且更生硬。
- 它补上了你缺的前提，而不只是删字。
- 项目名词替代了自造词。你 `CONTEXT.md` 里的术语回来了。
- 可以连续用两次，而不会退化成惜字如金。

## 它在整体中的位置

你可以在任何对话、任何时点、在任何其他技能内部使用 `wait-what`。它事后修复一条消息。真正的治本是事先约定共同语言，那是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)：在[追问 (grilling)](https://www.aihero.dev/ai-coding-dictionary/grilling)进行的同时跑 [domain-modeling](https://aihero.dev/skills-domain-modeling)，让你们共同用的词落进你的 `CONTEXT.md`。如果你不确定当下适合哪个技能，[ask-matt](https://aihero.dev/skills-ask-matt) 会帮你分发。
