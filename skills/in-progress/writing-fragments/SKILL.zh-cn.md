---
name: writing-fragments
description: "写作，探索阶段：挖掘原始碎片，暂不定结构。"
disable-model-invocation: true
---

<what-to-do>

这是纯粹的**探索（explore）**：拓宽可写内容的空间，不急于定结构。定结构是利用，另一个技能的工作。运行一场追问访谈，不松懈地问用户想写的东西。规定阶段、大纲或文章结构都不在范围内。

无论对话哪一方冒出碎片，都追加到同一个 Markdown 文件中。

如果用户没给路径，问一次文档存哪里，之后整个会话记住它。

从用户说的第一句话开始捕捉碎片，包括最初的提示语。

首次写入时，只在顶部放一个 H1 工作标题（以后可改），此外什么都不放：不要元数据、目录或日期。

</what-to-do>

<supporting-info>

## 什么是碎片

碎片是任何可能活到终稿的文字。它必须作者可读（作者能看懂意思），但不需要定义术语，也不需要让冷读者看懂。标准是“这是不是一段好文字”，而不是“这是不是自足的论证”。

碎片刻意保持驳杂多样。举例：

- 一句锋利的话，想用在某处，但还不知道放哪里。
- 一个主张加一句论证。
- 一段小插曲：发生过的事、一段代码、一个场景、一个类比。
- 半个想法：“X 让人感觉像 Y 之类的，以后再理清楚”。
- 一句引文、一段对话、一句无意中听到的话。
- 一组凭感觉放在一起的相关观察。
- 一句抱怨、坦白或妙语。
- 一个**引导词**：一个紧凑的隐喻或新造词，整篇文章可以挂在它上面（一个词命名一个想法，就像 _tracer bullets_（示踪弹）或 _fog of war_（战争迷雾）命名一整类模式）。

其中引导词是最有价值的碎片，它是承重的：在探索阶段定下正确的词，它会塑造后面的结构、过渡和标题，在整个利用阶段持续分红。当对话反复绕着同一个想法打转，就推一把，给它定个词。

范本是小说家的日记：多年无结构的所见所感，日后挖出来做原始素材。碎片就是所见所感。

## 文件格式

```markdown
# Working title

A first fragment lives here.

It can be multiple paragraphs. It can include lists, code, quotes: whatever
shape the fragment naturally takes.

---

A second fragment.

---

> A quoted line that the user wants to keep around.

A reaction to it.

---

- A cluster of related observations
- That hang together by feel
- And want to be near each other
```

碎片之间用水平分隔线（`\n---\n`）隔开。正文内不用标题，不打标签，除了添加顺序外不排序。

## 写作节奏

静默追加，不必每个碎片都请示。顺带提一句加了什么（比如“adding that”），但不要用保存确认打断对话。

每次写入前从磁盘重读文件。用户可能在轮次之间编辑、重排或删除碎片，所以保留他们的改动。绝不覆盖整个文件，只追加（用户要求时才原地编辑特定碎片）。

用户随时可以说“cut the last one”“rewrite that one sharper”“merge those two”，把这些当作一等指令对待。

</supporting-info>
