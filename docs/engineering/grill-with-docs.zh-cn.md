## 它的作用

`grill-with-docs` 会围绕某个计划或设计对你进行访谈，直到你和[智能体 (agent)](https://www.aihero.dev/ai-coding-dictionary/agent)对它形成共识，并在访谈过程中把词汇和关键决策写进仓库。它运行的是与 [grill-me](https://aihero.dev/skills-grill-me) 相同的访谈流程（一轮提问，然后等待，再进入下一轮），只是访谈对象是一个代码库。

它是[**有状态的 (stateful)**](https://www.aihero.dev/ai-coding-dictionary/stateful)。其他追问类技能只在你的头脑中留下[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)；而这个技能会在磁盘上留下文件。一个术语一旦确定，就会立刻写入 `CONTEXT.md`，而不是攒到最后批量写入。一项决策通过三道门槛，就会立刻记为一条 ADR。这就是它的全部区别，也是人们使用这个技能时遇到的大多数麻烦的来源：产物是真实仓库中的真实文件，所以它们可能在你以为存在时缺席，也可能在多人同时写入时产生分歧。

## 什么时候用它

由你输入 `/grill-with-docs` 来调用；智能体不会自行调用它。

当变更刚开始，计划还很模糊、描述事物的词汇尚未确定，并且你身处某个仓库中时，就用它。它是单会话工具。具体该用哪个追问类技能，取决于你面前的状况：

| 你手头的情况 | 该用 |
| --- | --- |
| 完全不在工作目录里工作 | [grill-me](https://aihero.dev/skills-grill-me) |
| 有一个仓库，且变更可以在一个会话内敲定 | `grill-with-docs` |
| 工作量太大，一个会话装不下（全新构建、大型功能） | [wayfinder](https://aihero.dev/skills-wayfinder) |
| 有一个仓库，但完全没有领域文档，也没有具体想做的功能 | `grill-with-docs`，访谈对象是仓库本身而非某次变更 |
| 某项决策卡在别人头脑里的知识上 | [to-questionnaire](https://aihero.dev/skills-to-questionnaire) |

它和 wayfinder 的区分在于会话数量：`/grill-with-docs` 用于单会话规划，`/wayfinder` 用于多会话规划。

## 前提条件

这个技能会向你的仓库写入内容，所以你需要处在一个可以安全写入的位置。确定的术语会写入根目录的 `CONTEXT.md` 词汇表；如果根目录的 `CONTEXT-MAP.md` 把该仓库标记为多上下文 (multi-context)，则写入对应上下文的 `CONTEXT.md`。决策写入 `docs/adr/`。两者都是按需创建的；在第一个术语或决策成形之前，什么都不存在，所以前期不需要搭建任何脚手架。

它还需要另外两个技能同时存在，因为它自己的 `SKILL.md` 只有一行委托逻辑：[grilling](https://aihero.dev/skills-grilling) 提供访谈能力，[domain-modeling](https://aihero.dev/skills-domain-modeling) 提供写作能力。只安装 `grill-with-docs` 会得到一个无法工作的技能。

## 书面产出

一次会话会产出三样东西，它们并不等价。

| 确定了什么 | 落到哪里 |
| --- | --- |
| 一个术语：项目内部对某事物的叫法 | `CONTEXT.md`，确定时立刻内联写入 |
| 一项难以逆转、缺少上下文会令人意外、且确有权衡的决策 | `docs/adr/` 下的一条 ADR |
| 其他所有你做出的决定 | 只存在于对话中，别处没有 |

第三行最容易让人踩坑。`CONTEXT.md` 是词汇表，并且有意只保留词汇：没有实现细节，没有[规格说明 (spec)](https://www.aihero.dev/ai-coding-dictionary/spec)，没有草稿笔记。ADR 要求三个条件同时满足，所以大多数决策都不符合，大多数会话都不会产生 ADR。一次只产出更清晰的词汇表、零 ADR 的会话是符合设计的，但这意味着你们达成的大部分共识只存在于达成它的那个[上下文窗口 (context window)](https://www.aihero.dev/ai-coding-dictionary/context-window)里。把同一段对话交给 [to-spec](https://aihero.dev/skills-to-spec)，而不是[清空 (clearing)](https://www.aihero.dev/ai-coding-dictionary/clearing)它。

词汇表才是重点。这个技能真正构建的是领域语言：项目自己的词汇，约定一次，之后你、智能体和同事都不用再反复推导。需要说明的是，并非所有人都认同这能提升智能体表现：最尖锐的公开反驳是，术语和它的白话展开从[模型 (model)](https://www.aihero.dev/ai-coding-dictionary/model)那里得到的结果是一样的，词汇真正压缩的是共享它的人类之间的沟通成本。即便按这种理解，词汇表依然有价值，只是价值的位置变了。

## 常见问题

**该用它还是用 `/wayfinder`？**
范围决定一切。在一个会话能敲定的事情上用它；当工作量太大、一个会话装不下时用 [wayfinder](https://aihero.dev/skills-wayfinder)，它会先把工作绘制成决策[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)地图。Wayfinder 更慢、更密，在范围清晰的功能上用它是常见错误。它不能替代本技能：它可以把地图中适合的部分交回给追问会话来处理。

**运行了，但没有出现 `CONTEXT.md`，也没有 ADR。**
两个已知原因。平常的原因：没有够格的内容。ADR 需要三道门槛全过，一次没有新词汇的变更会话确实没什么可写。真正的问题：当这个技能运行在另一层编排之下（spec 驱动开发封装、多智能体框架、把它当作流水线中一步来调用的规则）时，有报告称访谈照常运行，但写文件那一半会静默地没有发生。这个问题已记录，尚未修复。如果你处在这种配置里，先检查工作目录，再相信会话的输出。

**它一次性问完所有问题，不给建议，也从没提过 `CONTEXT.md`。**
那是它的两个依赖没有加载。因为 `SKILL.md` 是一行委托，智能体如果没有接上 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling)，就会自己猜测追问是什么意思，于是丢给你一堆无差别的提问。部分加载的情况更让人困惑：`grilling` 加载了而 `domain-modeling` 没加载，你会得到一场不错的访谈，但没有任何书面产出。这与模型和[投入档位 (effort)](https://www.aihero.dev/ai-coding-dictionary/effort)相关，也是这个技能被报告最多的问题。如果你怀疑是这个问题，直接问智能体它加载了哪些技能。

**我的其他决策都去哪了？**
只留在了对话里。这是关于这个技能最有实质内容的公开抱怨：词汇表不是规格说明，大多数回答够不上 ADR，而且没有任何台账把每个已确定的回答串到规格说明、工单和测试。精确的回答（顺序保证、否定性需求、数字默认值）在下游会被弱化成更含糊的文字，结果看似完整，却丢了你真正定下来的东西。目前可用的缓解办法是保留会话，把它直接喂给 [to-spec](https://aihero.dev/skills-to-spec)，并对照你自己的回答重读规格说明，不要默认它已经完整记录。

**能把它指向一个完全没有文档的已有仓库吗？**
可以。面对没有 ADR、没有领域语言、没有设计原则的代码库，这正是合适的技能：调用它，说“帮我给仓库写文档”。社区里的常见搭配是把它和 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 一起用，来构建或修复 `CONTEXT.md`。注意要主动引导：它会读代码，并就读到的内容向你提问，而由你来判定代码库里已有的词里哪些是正确的。

**会话结束时我该做什么？**
这个技能的结束语往往比较开放，这是已知的粗糙之处。在主流程里，答案是留在同一段对话里接 [to-spec](https://aihero.dev/skills-to-spec)。如果变更小到可以立刻动手，就直接去 [implement](https://aihero.dev/skills-implement)。

**为什么叫这个名字？**
没人对这个名字满意。有一个公开建议是改名为 `grill-domain-model`，更能诚实地描述它的行为。目前没有任何进展。如果改名真的落地，文档页会跟着搬家，URL 也会变化。

## 生效的标志

- `CONTEXT.md` 在会话**过程中**逐条变化，而不是在结束时一次性出现。
- 词汇表读起来是纯词汇（项目自己的词加紧凑定义），不含实现细节或规格说明式的文字。
- 代码库能回答的问题都通过读代码库来回答，而不是来问你。
- ADR 很少或没有，出现的 ADR 都是那种“要我重新争一遍会很烦”的决策。
- 它会因为既有词汇表的定义不同，而挑战你使用的某个词。

## 它在流程中的位置

`grill-with-docs` 是主构建链的起点：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它位于任何成文规格说明之前：它产出共识和确定的词汇，然后 [to-spec](https://aihero.dev/skills-to-spec) 在此基础上综合成文，不再重复访谈你。它的近邻是 [grill-me](https://aihero.dev/skills-grill-me)（同样的访谈，没有仓库也没有文件）和 [domain-modeling](https://aihero.dev/skills-domain-modeling)（它所驱动的词汇表加 ADR 规范）；两者都建立在 [grilling](https://aihero.dev/skills-grilling) 这一原语之上。它的上游是 [wayfinder](https://aihero.dev/skills-wayfinder)，负责绘制单会话装不下的工作，再把地图中合适的部分交回给它。当你不确定哪个技能或流程合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
