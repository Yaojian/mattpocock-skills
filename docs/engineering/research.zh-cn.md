## 它的作用

`research` 通过阅读拥有答案的来源来回答问题，然后在仓库里留下一份带引用的 Markdown 文件。它只使用[**一手来源 (primary source)**](https://www.aihero.dev/ai-coding-dictionary/primary-source)：官方文档、源代码、规格说明、第一方 API。它把每个断言一路追到拥有它的来源，所以只要官方文档可达，它就不会复述某篇博客对 API 的转述。

它不在对话里回答你。产出是一个文件，写在仓库本来就放这类笔记的地方，每个断言带链接。重点正在于此：得到一份可以反应、可以丢给另一个智能体、也可以扔掉的文档，而不是一段随[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)结束就消失的回答。

## 什么时候用它

输入 `/research`，或者[智能体 (agent)](https://www.aihero.dev/ai-coding-dictionary/agent)在任务变成阅读体力活时自动调用它。

当下一步是向工作目录之外*打听某件事*（第三方 API 的行为、规格说明到底怎么说、某个版本断言是否成立），而你不想让自己的主线程卡在阅读上时，就用它。你需要什么，决定了该用哪个技能：

| 你需要 | 该用 |
| --- | --- |
| 一项决策等着要的外部事实 | `research` |
| 和你一起*通过访谈*做出的决策 | [grilling](https://aihero.dev/skills-grilling) |
| 写进 `CONTEXT.md` 和 ADR 的持久架构决策 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 验证某个方案在你的代码库里是否可行 | [prototype](https://aihero.dev/skills-prototype) |
| 单会话装不下的大计划 | [wayfinder](https://aihero.dev/skills-wayfinder) |

`research` 和 `grill-with-docs` 的界线在于**产出的保质期**。Research 产出短命资产：这个库的鉴权机制截至本周是什么样。ADR 记录你要留下的决策。如果你产出的是决策而不是事实，那你是在[追问 (grilling)](https://www.aihero.dev/ai-coding-dictionary/grilling)，不是在研究。

## 委托出去的体力活

标志性动作是阅读以**后台智能体**方式运行。你继续干你的；它去把每个断言追到一手来源，写一个 Markdown 文件，回来汇报。Research 是委托出去的体力活，不是外包出去的思考：你拿到的是一份可以追问、规划、设计的文档，决断还是你来做。

委托是无防护的，后台智能体还能再开自己的后台智能体。这是这个技能记录最充分的粗糙之处。

文件落在哪里由仓库决定，不由技能决定：它沿用已有放笔记的约定，没有约定就选个合理位置并告诉你。一次运行写一个文件。

## 常见问题

**它又开了一个 research 智能体，这是设计好的吗？**

不是。这是已知的 bug (open bug)，[issue #530](https://github.com/mattpocock/skills/issues/530)。技能让调用方开一个后台智能体，但不限智能体类型，于是开出来的 `general-purpose` 智能体手握 `Agent` 工具和同一份指令，又照做了一遍。一位报告者测到单次 research 任务横跨三次重叠运行烧了约 45 万 [token](https://www.aihero.dev/ai-coding-dictionary/token)，重复的那次在视线之外半小时后才结束。在 Claude Code 之外也能复现；同样的嵌套在 Codex 加 GPT-5.6-sol 上也确认过。目前没有随附修复。用户给自己装的副本打了补丁，加一行告诉已是[子智能体 (subagent)](https://www.aihero.dev/ai-coding-dictionary/subagent)的智能体自己动手，这有帮助，但只是指令层，不是结构层。调用后盯一下后台任务列表，把重复的停掉。

反方向的失败也存在：如果你的全局指令禁止智能体转委托，后台智能体会礼貌拒绝任务，技能就悄无声息地什么都不做。

**文件该放哪，要提交吗？**

技能把文件放在仓库本来放笔记的地方，除此之外没有主张。社区共识比较稳定：ADR 留，research 文件不留。针对这个问题的 Discord 讨论里最尖锐的版本：“ADR 留，其他做完就归档或删除，不然会变成工作残渣，偏离规格和研究之后还会污染未来的仓库读取。”research 文件记录的是成文当天的事实，所以过期的文件比没有更糟。总的来说这些产物并不真正属于 git，人们用 Obsidian、单独的知识库或 issue 跟踪器放它们，也没有公认的归处 (canonical home)。

**什么算“高可信”一手来源，谁说了算？**

[模型 (model)](https://www.aihero.dev/ai-coding-dictionary/model)说了算。技能只命名够格的来源*种类*（官方文档、源代码、规格说明、第一方 API），没有允许清单 (allowlist)，没有域名门槛，也没有核验环节。这是技能刚提出时最响亮的反对，至今没有公开回应：“五个 research 子智能体对着垃圾来源，只会更快产出五个自信的错误答案，怎么把关 (gate) 高可信来源？”你真正有的缓解办法是每个断言上的引用。随手跟两三个，如果落到对东西的转述而不是东西本身，这次运行就没干好它唯一的工作。

**之后的会话会复用之前跑出的结果吗？**

不会。过去的 research 文件不会自动加载；它就是躺在仓库里的一份文档，直到人或技能指向它。这是最早提出的最强设计质疑：“价值在于 markdown 变成智能体之后重读的上下文，而不是抓取本身，只写一次的死文件就是花哨的搜索。”随附的技能没有解决它。实践中文件要靠主动投喂到下一步才有价值：附到规格说明上、引用进追问会话、让[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)指向它。

**为什么不直接让智能体去读文档？**

可以，两行提示说清要求就是这个技能所替代的老做法。相比随手提示，技能多买到两样东西：后台运行让你的会话[上下文 (context)](https://www.aihero.dev/ai-coding-dictionary/context)保持干净，一手来源约束和带引用的文件产出每次都一样，而不是随你的措辞浮动。对上[执行环境 (harness)](https://www.aihero.dev/ai-coding-dictionary/harness)自带的深度研究模式，差别在产物和来源纪律，不在搜索。小问题上两行提示能解决，就用两行提示。

**它什么时候停下阅读？**

技能里没有停止标准，表现为两个看似相反、实为同源的抱怨：挖得太深的智能体，和铺得很宽却漏掉唯一关键细节的智能体。一位实践者的话是“深度研究技能有时太深了，而让智能体去研究常常漏掉关键细节。”范围由你来定。窄而可答的问题（一个 API、一个行为、一个版本断言）远比“研究一下 X”效果好。

**`/wayfinder` 建了 research 工单，我自己去解决吗？**

不用，现在它替你开火。v1.1 之后未发布的改动里，绘制会话会按 research 工单各开一个 `/research` 子智能体，并行烧完，把发现记在抛弃型的 `research/<name>` 分支上，由工单上的[上下文指针 (context pointer)](https://www.aihero.dev/ai-coding-dictionary/context-pointer)引用。research 工单是 wayfinder 单工单单会话规则的唯一例外，因为它们是 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk)：不需要你守着。这些分支有两个已知坑：子智能体曾被看到从永不合并的分支开 draft PR（[issue #576](https://github.com/mattpocock/skills/issues/576)），以及之后删掉分支会弄断工单上的上下文指针。

## 生效的标志

- 你自己的会话不停。如果你在坐看它阅读，委托就没发生。
- 只出现一个新增后台任务。第二个名字相近的就是嵌套 bug。
- 只新增一个 Markdown 文件，落在仓库本来放笔记的文件夹，智能体告诉你路径。
- 每个断言都带链接，随手跟两个，落到官方文档、规格说明或源文件本身，而不是别人对它的转述。
- 只凭这份文件就能做出卡住你的决策，不用自己回去翻来源。

## 它在流程中的位置

随时取用的独立技能，投喂思考类技能而不坐在构建链里。它的文件是带*进*流程的东西：手头有了事实，[grilling](https://aihero.dev/skills-grilling) 和 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 的问题更准，[to-spec](https://aihero.dev/skills-to-spec) 可以对照它综合。唯一直接调用它的技能是 [wayfinder](https://aihero.dev/skills-wayfinder)，用一个 `/research` 子智能体解决地图上的每个 research 工单。全图见 [ask-matt](https://aihero.dev/skills-ask-matt)。
