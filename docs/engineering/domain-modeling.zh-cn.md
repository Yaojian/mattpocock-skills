## 功能简介

`domain-modeling` 在设计过程中构建并打磨项目的**通用语言 (ubiquitous language)**：挑战与词汇表冲突的术语，在你用词含糊处逼出精确的词，用具体场景压测关系直到边界精确。

它是**主动**的纪律，不是被动的。读 `CONTEXT.md` 借用词汇是任何技能都能做的一行习惯；这个技能管的是你*改*模型的时候。这就是它打断你的原因。它在术语敲定的那一刻就把决议写进 `CONTEXT.md`，在对话中间写，而不是最后产出一份整齐的词汇表，因为批量版是[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)的总结，而内联版是会话的实际产出。

## 何时使用

输入 `/domain-modeling`，任务匹配时智能体也会自动调用。实际中自动调用是这个技能最弱的一环：当 `grill-with-docs` 或 `wayfinder` 说加载它时，[模型 (model)](https://www.aihero.dev/ai-coding-dictionary/model)经常加载 `grilling` 而跳过它。如果一场[追问 (grilling)](https://www.aihero.dev/ai-coding-dictionary/grilling)跑完 `CONTEXT.md` 纹丝不动，那就是发生了这种情况；把本技能和另一个技能一起点名调用。

当*措辞*就是问题时用它：

| 处境 | 走法 |
| --- | --- |
| 两个人说的“cancellation”不是一回事 | `domain-modeling`：定一个标准术语，其他的列进 `_Avoid_` |
| “Account”在三个文件里干三份活 | `domain-modeling`：拆成 Customer 和 User |
| 你刚做了难逆转的架构选择 | `domain-modeling`：够格的话它会建议写 ADR |
| 问题在模块*形状*：接缝放哪，接口多深 | [codebase-design](https://aihero.dev/skills-codebase-design) |
| 想在开工前把整个计划盘问一遍 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它在底下驱动本技能 |
| 只想查词，不想改 | 什么都不用。读 `CONTEXT.md` 就行，它就是个文件。 |

## 前置条件

开工不需要任何东西。技能写两个地方，都是懒创建：

- 仓库根的 **`CONTEXT.md`**，由第一个敲定的术语创建。根下有 `CONTEXT-MAP.md` 的仓库，术语写进地图指向的各上下文 `CONTEXT.md`。
- **`docs/adr/`**，由第一份过线的 ADR 创建。

开始前什么都不必存在，也不会预先创建任何东西。

## 两份工件，两道线

词汇表和 ADR 是两套标准，混为一谈是这个技能大多数麻烦的来源。

| | `CONTEXT.md` | `docs/adr/NNNN-slug.md` |
| --- | --- | --- |
| 装什么 | 术语。一个东西**是什么**，一到两句话，被否掉的同义词放 `_Avoid_` | 一个决定，一到三句话：背景、选择、理由 |
| 动笔的线 | 含糊的词变成了标准词 | **三条全中**：难逆转、缺上下文会意外、是真实权衡的结果 |
| 什么时候写 | 内联，敲定的那一刻 | 建议写，不擅自写 |
| 永远不装 | 实现细节、[规格说明 (spec)](https://www.aihero.dev/ai-coding-dictionary/spec)、草稿纸、通用编程概念 | 本会话每个选择的流水账 |

ADR 三条测试缺一条就没有 ADR。容易逆转的决定反正会被逆转；不意外的决定没人会问；没有真实备选项的记录只是在记你做了显而易见的事。

`CONTEXT.md` 那条线才是真正要守住的，因为它在实战里最容易破。**它只是词汇表，别的什么都不是。** 不盯着的话，模型会把“写进 `CONTEXT.md`”当成持久化你每个回答的许可，文件很快变成连载的规格说明。这是本技能被上报最多的问题，跨多个模型都一样。

## 交叉核对，以及到哪里停

让这个技能成立的动作：当你说清某样东西怎么运作，它去查代码并把矛盾摆出来。*“你的代码取消的是整个 Order，但你刚说支持部分取消，哪个对？”* 语言和代码先当面达成一致，再改任何一边。

上限值得知道。它只交叉核对**代码**和已提交的 `CONTEXT.md`/ADR，别的都不查。它不搜你的议题追踪器，所以几个月前在已关闭议题里争过并已定论的命名冲突，会被当成新问题重新摆出来。修它的[公开请求](https://github.com/mattpocock/skills/issues/717)还在；在此之前，变通办法是把指令写进你自己的 `docs/agents/domain.md`，各技能本来就读它。

## 常见问题

**我的 `CONTEXT.md` 500 行了。1000 行。3000 行。怎么办？**
大小是症状，不是病：文件吸进了实现细节和根本不是词汇的决定。修法是一句直接指令：`/grill-with-docs make my CONTEXT.md more concise and remove any implementation details from it`。拿它去打臃肿文件，大半都会被删掉。只有当文件真正精简、仍然覆盖两个读者不想同时装脑子里的域时，才考虑 `CONTEXT-MAP.md` 拆分；拿臃肿文件去拆，只会得到几个臃肿文件。技能目前防增生的指导还不够强，追踪的议题还开着。

**为什么叫 `CONTEXT.md` 而不叫 `GLOSSARY.md`？**
这是整套技能里吵得最凶的命名问题，而且没有定论。反对方的理由很硬：如果它“只是词汇表，别的什么都不是”，`GLOSSARY.md` 才名副其实，用一位读者的话说，“跟 AI 智能体在一起，什么都是[上下文 (context)](https://www.aihero.dev/ai-coding-dictionary/context)”。正方的理由是地图：`CONTEXT-MAP.md` 指向几个 `CONTEXT.md` 读起来顺，`GLOSSARY-MAP.md` 就不顺，而且 `context` 是 DDD 里 bounded area of the model 的现成词。至少有一个人为了改名维护本地 fork。你也可以照做，但全套其他技能找的都是 `CONTEXT.md`，改名意味着全部打补丁。

**`/ubiquitous-language` 去哪了？**
移除了，不是废弃。它的活搬进了 `domain-modeling`，后者持续维护整个模型，而不是从一次对话里倒出一份词汇表。词汇强制从此分量更重，而不是更弱：它在追问、分诊和映射之下跑，而不是等你记得才做的一次单独检查。

**代码库还没有词汇表，怎么从零起一份？**
显式要，而不是等它慢慢攒。`/grill-with-docs help me scaffold my existing repo with a CONTEXT.md` 是文档化的路线；预期是一场漫长的盘问：有用户报告问了 50 多个问题文件才成形。靠零散使用在存量仓库里攒词汇表太慢了。

**我能留着领域模型，用自己的 ADR 格式吗？**
目前不顺。词汇表一半和 ADR 一半装在一个技能里，有既定 ADR 规范（不同模板、不同位置、不同命名）的团队拿到的指令会跟自家风格打架。目前的选项是本地复制技能自己改，或者在仓库自己的智能体文档里覆盖 ADR 约定。把两者拆开是[公开请求](https://github.com/mattpocock/skills/issues/557)。

**词汇表真划算吗？又多一份要评审的东西，还会过期。**
有时不划算，诚实点说清边界。DDD 越靠近实现越没用：回报在上游，在命名和概念对齐，不在聚合和分层仪式。同义词管控在命名边界重要：模块名、表名、状态枚举、议题标题、CLI 命令。在普通正文里重要得多收敛。还有个活着的异议：领域术语压缩的是*人跟人*之间已有共识的沟通，智能体对白话描述的反应是一样的。按这个理解，词汇表的价值是让你和评审人跟智能体在干什么对齐，而不是让智能体更强。一天的构建就跳过它。没人评审、智能体写的词汇表不如没有：它会变成听起来自信的传说，后面的会话都拿它当真。

**它能把我的含糊提示翻成领域语言吗？**
不能，也没打算为此做技能。你自己都不懂的领域语言写下来就是没意义的套话。这个技能在你有理解之后强制精确，不替你发明你没有的词汇。相关的坑是用了领域词却没做建模：名词对了，概念结构错了，产出读着正确其实不对。

## 怎样算生效

- 它中途打断你，问你两个意思指哪一个，而不是替你选一个往下走。
- `CONTEXT.md` 在对话**过程中**改，而不是最后集中一次性写入。
- 明天就能撤销的东西，它拒绝写 ADR，并说三条测试挂了哪条。
- 新条目用一到两句话定义一个东西*是什么*，并在 `_Avoid_` 下写清你放弃的词。
- 你的代码和你的话对不上时，它把你的代码引回来摆在你面前。
- `CONTEXT.md` 和变长一样经常变短。

## 在整体中的位置

`domain-modeling` 是**模型可调用的参考型技能**，跑在*其他技能底下*的时候多过单独跑。[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 在追问会话里驱动它，[wayfinder](https://aihero.dev/skills-wayfinder) 在画地图时加载它，[triage](https://aihero.dev/skills-triage) 用它让[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)说项目自己的话，[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 在决定成形时调用它。它最近的兄弟是 [codebase-design](https://aihero.dev/skills-codebase-design)：两者是其他一切之下的词汇层，这个管*领域*，那个管模块*形状*。也可以直接调用，想要这份纪律又不想接管它通常附着的那个技能的步骤时。拿不准哪个技能合适，用 [ask-matt](https://aihero.dev/skills-ask-matt) 路由。
