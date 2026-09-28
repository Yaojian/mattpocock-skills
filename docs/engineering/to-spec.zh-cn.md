## 功能简介

`to-spec` 把你刚刚进行的对话变成一份**[规约 (spec)](https://www.aihero.dev/ai-coding-dictionary/spec)**，并以单个 issue 的形式发布到你的问题跟踪器上。

它不会采访你。走到这一步时，决策已经完成，所以它综合已知信息（来自对话线程、代码库、你的 `CONTEXT.md` 和 ADR），而不是开启新一轮提问。规约是已做决策的记录，不是产生新决策的地方。

## 何时使用

输入 `/to-spec` 即可调用；[智能体 (agent)](https://www.aihero.dev/ai-coding-dictionary/agent)不会自动选用它。

当构建太大，单个智能体[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)装不下，必须拆到多个会话中完成时，就该用它。触发条件就是这一个：

| 你的处境 | 该运行什么 |
| --- | --- |
| 还没有做任何决定 | 先用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 已决定，且工作量适合一个[上下文窗口 (context window)](https://www.aihero.dev/ai-coding-dictionary/context-window) | [implement](https://aihero.dev/skills-implement)：跳过规约 |
| 已决定，且工作量横跨多个会话 | `/to-spec`，然后用 [to-tickets](https://aihero.dev/skills-to-tickets) |
| [wayfinder](https://aihero.dev/skills-wayfinder) 地图已完成 | `/to-spec #<map_issue>` |

## 前置条件

`to-spec` 把规约发布为 issue，所以 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须先为本仓库配置好跟踪器和分诊标签词汇。两种都可以：GitHub 这样的真实跟踪器，或者 `.scratch/` 下的本地 markdown 文件，开箱即用。

## 规约是决策记录

规约存在的原因是上下文窗口会结束。你在[追问 (grilling)](https://www.aihero.dev/ai-coding-dictionary/grilling)期间定下来的所有东西（解决方案的形状、争论过的选择、明确拒绝的内容）都在一个即将被清空的对话里。规约就是活下来的那部分。

所以它不验证任何东西，也不决定任何东西。它用项目自己的词汇记录已决定的事项，让一个全新的会话可以接手工作，而不用你再解释一遍。规约中任何你其实没说过的断言都是缺陷。

## 接缝先于正文

在写下一个字之前，`to-spec` 先勾勒出功能将要测试所在的**接缝 (seam)**，并请你确认。它优先使用已存在的接缝而不是新建，并取尽量高层的接缝：一次改动的理想接缝数量是一个。

这些确认过的接缝会继续传递。[tdd](https://aihero.dev/skills-tdd) 只在预先确认的接缝上工作，[code-review](https://aihero.dev/skills-code-review) 对照规约评审 diff，因此未经确认的接缝会作为评审意见冒出来。这个约束是间接的：它经由本文档传递，这正是接缝对话值得在这里认真对待，而不是推迟到实现阶段的原因。

## 常见问题

**`/to-prd` 去哪了？**
它就是本技能，在 v1.1 改名。“规约 (spec)”现在是贯穿始终的唯一术语，旧的 `to-prd` 别名已废弃；请按新名称重装。取代旧词汇的一对术语是*规约*和*工单*：规约是目标和锁定它的决策，[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)是到达那里的执行步骤。如果方向变了，删掉未完成的工单，留下规约。

**为什么规约会被打上 `ready-for-agent` 标签？我不想让智能体照着它直接开工。**
该标签的意思是“无需进一步分诊”：文档已完整到智能体可以照着工作。它是输入就绪标志，不是开工指令。但如果你运行轮询 `ready-for-agent` 的 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) 智能体，它们看不到这个区别，会愉快地试图一次跑完整个规约，而不是按工单切片领取。这是本技能被反馈最多的粗糙之处。在改动之前，请在 AFK 智能体的提示中明确排除父规约，或者在 `/to-tickets` 跑完后摘掉该标签。

**为什么不从追问直接跳到 `/to-tickets`，跳过规约？**
很多时候就该跳过；规约这一步只在多会话工作中才值回成本。它的价值在于工单用完即弃而规约不是：每个工单按一个全新上下文窗口切分，做完就删除或关闭，而规约留下推理过程的唯一存放处。对单会话改动，这买不到任何东西，你还多付了一次综合步骤，让[模型 (model)](https://www.aihero.dev/ai-coding-dictionary/model)有机会漂移。此时应走追问到 `/implement`。

**我刚完成一张 wayfinder 地图，该喂给它什么？**
主地图 issue：`/to-spec #<map_issue>`，而不是单个决策工单。[wayfinder](https://aihero.dev/skills-wayfinder) 产出的是决策，分散在地图各处，而不是可交付物；`to-spec` 是把它们收敛为一份可构建文档的步骤。把地图直接接到 `/implement` 上，会丢掉这次收敛。

**规约是给我看的，还是只给智能体看的？**
主要是给智能体看的，读起来也像：完整、密集、引用多。值得你看的是接缝和非目标小节，因为这两处的错误决定在早期发现成本最低，事后发现代价最高。从头到尾通读全文确实是大家抱怨的一点，目前没有摘要模式：诚实的答案是，如果规约让你意外，说明追问做得太浅，而不是规约太长。

**工单开工后，我该冻结规约，还是让智能体改写它？**
没有任何机制保持它同步，所以实践中它是你当时所知的快照，实现一旦教会你新东西它就过时了。工作上线后就把它当一次性产物对待。真正要留下的产物是你的 `CONTEXT.md` 和 ADR；实现中学到的值得保留的东西属于那里，而不是改写规约。

**我的工作是重构或模块边界，不是功能，模板还适用吗？**
适配度较差，这是已知局限。模板重度依赖用户故事，而这对架构工作是错误的形状：你会围绕接口和不变式的决策，硬写没人要的故事。应倚重实现决策和测试决策两节，把持久的架构决策经由 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 落为 ADR，而不是硬让规约承载它们。

**它会检查跟踪器中的相关工作，或引用它遵守的 ADR 吗？**
两者都不会。它会阅读并遵守所触及范围的 ADR，但不链接它们，也不先搜索跟踪器中的重叠 issue，因此规约可能悄悄重复别人已建的工作。如果相关区域很繁忙，请自己先搜索跟踪器。

**`/to-tickets` 读不到我的规约：一直在截断。**
很大的规约可能超出跟踪器 issue 一次能干净返回的长度，而且没有本地副本可回退。修复办法是上下文卫生：不要在 `/to-spec` 和 `/to-tickets` 之间[清理 (clearing)](https://www.aihero.dev/ai-coding-dictionary/clearing)或[压缩 (compaction)](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一个窗口内连着运行两者，规约就根本不需要重新取回。

## 生效标志

- 它直接开始写，而不是开启新一轮提问。
- 动笔前先把接缝交给你确认，并尽量提最少的数量。
- 它用你项目的名词写成，而不是通用产品管理套话。
- 其中每个决策都是你记得做过的，没有为填章节而虚构的内容。
- 非目标小节中有实实在在的内容：你拒绝的东西通常是全页最有用的几行。

## 在流程中的位置

`to-spec` 是主构建链中的一步，而且只出现在多会话分支上：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

上游邻居是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它完成本技能只负责记录的决策；[wayfinder](https://aihero.dev/skills-wayfinder) 完成的地图正是在这里汇入链条。下游 [to-tickets](https://aihero.dev/skills-to-tickets) 把规约切成示踪弹工单，供 [implement](https://aihero.dev/skills-implement) 构建。当不确定哪个技能或流程合适时，由 [ask-matt](https://aihero.dev/skills-ask-matt) 指路。
