## 它能做什么

`wayfinder` 处理的是单个 Agent [会话 session](https://www.aihero.dev/ai-coding-dictionary/session)装不下的工作：一个你能说出**目的地（destination）**、但还看不清路线的主意，把它画成一张共享的**地图（map）**，地图由 issue 跟踪器上的**决策工单（decision tickets）**组成，然后逐个解决，直到路线清晰。

它只规划，不动手。每张工单承载的问题，其解决方式都是一个决策，而不是一段待执行的构建切片；当地图上再也没有开工前必须做的决策时，地图就算完成。这条规则是 wayfinder 工单和普通实现[工单 ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 的分界线，也是 Agent 最常违反的规则。地图走通后 wayfinder 就交接，它不会继续写代码。

## 何时使用它

输入 `/wayfinder` 来调用；[Agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自动调用它。

它是整套技能里最重、最密的流程，所以触发条件很窄：工作量必须确实超过一个 Agent 会话能装下的规模，而且通往目的地的路线必须还不清楚。划分很干净：单会话规划用 `/grill-with-docs`，多会话规划用 `/wayfinder`。

| 你面前的东西 | 该运行什么 |
| --- | --- |
| 一次坐下来就能定好的、范围清晰的功能 | [grill-me](https://aihero.dev/skills-grill-me)，有代码库时用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 路线尚不清晰的绿地项目，或跨多个会话的构建 | `/wayfinder` |
| 决策已经做完的讨论串 | [to-spec](https://aihero.dev/skills-to-spec)：直接跳过地图 |
| 已走通的 wayfinder 地图 | [to-spec](https://aihero.dev/skills-to-spec)，然后是 [to-tickets](https://aihero.dev/skills-to-tickets) 和 [implement](https://aihero.dev/skills-implement) |
| 已经膨胀过大的现有会话 | 说“交接给 `/wayfinder`”（[handoff](https://aihero.dev/skills-handoff) 既能接进地图也能接出地图） |

绿地项目不是必要条件。Wayfinder 经常用在遗留和半成品代码库上，甚至在那里更锋利，因为很多迷雾是“这里已经有什么是真的”，而不是“我们该做什么”。

## 前提条件

地图和工单都放在仓库的 issue 跟踪器上，所以 wayfinder 需要 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 打好的跟踪器配置。那一步会写出“Wayfinding operations”一节，说明针对 GitHub、GitLab 或本地 markdown，地图、子工单、阻塞边和前沿查询如何表达。Wayfinder 通过 `CLAUDE.md` / `AGENTS.md` 中的指针找到那份文档，而不是写死路径；如果完全没有配置跟踪器，就回退到本地 markdown 文件。

跟踪器不是装饰。阻塞关系决定了前沿能否直接在跟踪器自己的界面里可视化；没有原生依赖链接的跟踪器（比如自建 Gitea），wayfinder 只能从地图文本推断阻塞关系，能用，但需要更密切的人工监督。

## 地图、迷雾和前沿

**地图（map）**是一个打着 `wayfinder:map` 标签的 issue，它的工单是它的子 issue。它是**索引，不是仓库**：一个决策只存在于一个地方，即它的工单，地图只写摘要并链接。会话以低分辨率加载地图，按需放大查看单个工单，这样地图可以一直变大，而每个会话不用为全部历史付费。

地图上有四样东西：

- **目的地（Destination）**：走到这张地图终点时是什么样子。在任何工单存在之前先定目的地，这是绘制地图的第一个动作，因为每张工单的范围都要对照目的地来衡量。
- **已做决策（Decisions so far）**：每个已关闭工单一行，每行链接到细节真正存放的位置。
- **尚未明确（Not yet specified）**：**战争迷雾（fog of war）**。你能看出迟早要面对、但现在还无法精确表述的决策。区分迷雾和工单的标准是，你*现在*能否把问题精确地讲出来，而不是你能否回答它。解决一张工单会驱散它前方的迷雾，把现在能讲清楚的内容升级为新工单。
- **范围之外（Out of scope）**：被判定超出目的地的工作。迷雾只朝目的地聚拢，所以范围之外的工作直接关闭，永远不会升级。

**前沿（frontier）**是开放、未被阻塞、无人认领的工单（已知世界的边缘）。一个会话认领工单的方式是在动手前把它指派给自己，所以负责人（assignee）*就是*认领标记，并发会话会自动跳过。叙述中一律用名字指代工单，绝不用光秃秃的 `#42`；一墙的 issue 编号没法读。

## 四种决策工单类型

每张工单带一个 `wayfinder:<type>` 标签，并且是 **[HITL](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)**（与真人一起工作，真人为自己发言）或 **[AFK](https://www.aihero.dev/ai-coding-dictionary/afk)**（由 Agent 独立驱动）之一。HITL 工单只能通过实时交流解决；自己回答自己的[追问 grilling](https://www.aihero.dev/ai-coding-dictionary/grilling)问题的 Agent 已经违规。

| 类型 | 模式 | 何时使用 | 如何解决 |
| --- | --- | --- | --- |
| `grilling` | HITL | 默认选项。问题可以通过讨论解决。 | 在全新会话中跑 [grilling](https://aihero.dev/skills-grilling) 加 [domain-modeling](https://aihero.dev/skills-domain-modeling) |
| `prototype` | HITL | “应该长什么样”或“应该有什么行为”：光靠聊解决不了的问题。 | 跑 [prototype](https://aihero.dev/skills-prototype)，构建产物作为附件链接在工单上 |
| `research` | AFK | 工作目录之外的某个事实阻塞了决策。 | 在绘制地图时派出 [research](https://aihero.dev/skills-research) [子智能体 subagent](https://www.aihero.dev/ai-coding-dictionary/subagent)，在 `research/<name>` 分支上并行消化 |
| `task` | 都可以 | 无需决策，但有手工活阻塞了决策，比如开通权限、注册服务、搬运数据以便看清数据形状。 | Agent 能做就自己做，否则给真人一份精确的检查清单 |

`task` 是唯一*动手*而不*决策*的类型，它的位置是靠“为决策扫清阻塞”挣来的，绝不是靠交付目的地的一部分。在实践中这也是最容易出错的类型：Agent 常把它理解成实现步骤，在地图里直接写起产品代码。

Research 是*单工单单会话*规则的唯一例外。

## 常见问题

**它和 `/grill-with-docs` 有什么区别？我该从哪个开始？**
看会话数量，不看项目大小。`/grill-with-docs` 是单会话规划；wayfinder 是多会话规划。如果整件事能在一个对话里装下，追问是更便宜更好的工具，这时用 wayfinder 又慢又重。社区总结的顺口溜是：只有装不进单个会话的工作才值得上 wayfinder。这是被问得最多的 wayfinder 问题，而且会一直被问下去，因为光看描述你判断不出自己的任务落在这条线的哪一段。会话数量只能你自己判断。

**它问“目的地（destination）”时，是指这次会话的终点还是整件事的终点？**
指整张地图。也就是整个地图的目的地，不只是初始会话。这个问题读起来有歧义，是因为 wayfinder 按定义就是多会话工具，会话级别的答案永远说不通。典型的目的地有：要交接出去的[规格文档 spec](https://www.aihero.dev/ai-coding-dictionary/spec)、规划开始前要锁定的决策、概念验证，或数据迁移这类就地完成的变更。

**地图走通了。难道 wayfinder 还没写规格、建工单吗？为什么还要 `/to-spec` 和 `/to-tickets`？**
没有。Wayfinder 的工单是决策工单，地图关闭时它们也全部关闭了。剩下的是一张挂满已链接决策的地图，这不是构建计划。[to-spec](https://aihero.dev/skills-to-spec) 把这些已链接的决策收拢成一份规格（`/to-spec #<map_issue>`），[to-tickets](https://aihero.dev/skills-to-tickets) 再把它切成子弹轨迹式的实现工单。把地图直接连到 [implement](https://aihero.dev/skills-implement) 会跳过收拢，把已链接的细节扔掉。只有当工作量确实很小时才直接去实现。有人跑过缩写版流程并说能用；多出的两步换来一份评审人或同事能读的显式规格 artifact，你越不是单打独斗，它越值钱。

**我的 Agent 在 wayfinder 会话中途开始写生产代码了。**
这是该技能被报告最多的故障，背后有个真缺口。Wayfinder 默认“只规划、不动手”，但可以在地图的 **Notes** 中覆盖，而 Notes 是 Agent 自己写的，所以约束和豁免住在同一个文件里，而那个文件归被约束的一方所有。有用户亲眼看到 Agent 在自己的 Notes 里写下“这张地图包含执行”，然后在后续会话中把它当作自己的许可证，在线上服务器上构建。技能里目前没有针对“我指的是默认情况”的硬拦截。在此之前：凡是不是你亲手绘制的地图，先读 Notes；实现放到独立会话里；任何看起来像构建切片的 `wayfinder:task` 都当作类型标错来处理。

**我画了 27 张工单，做到第 13 张时，剩下的已经对不上了。**
真实且被反复报告的结果，逐字引自一线反馈。Wayfinder 的默认冲动是全面规划，而后排工单建立在会被前排结论推翻的假设上，这正是它被指责的瀑布陷阱。有两件事能对冲它。把地图范围收敛到有界的目的地，而不是整个产品。实践者一致反馈，收敛到单个明确 epic 的地图，表现远好于铺满的“实现 V1”，而规划超大事项本来也不是目标：小步交付才是。还要激进地 [prototype](https://aihero.dev/skills-prototype)：让路线保持新鲜的办法，是在实现依赖它们之前，用便宜的具体产物把不确定性提前暴露掉。Wayfinder 是“原型最大化（prototypemaxxing）”，不是“规划最大化（planmaxxing）”。

**我能并行做几张工单吗？**
前沿本来就是展示可接工作的，阻塞边也是为了让并行在纸面上安全。但实践中一次一张是更安全的默认。并行跑两张追问工单的用户，会在一个会话里被问到刚在另一个会话里回答过的问题，因为会话之间不共享[上下文 context](https://www.aihero.dev/ai-coding-dictionary/context)。原型工单上还有个已知缺口：有报告说 Agent 做了三个 UI 变体，自己选了一个，然后关了工单。选择权在你手里，而技能目前对此说得不够响亮。如果你确实要并行，请先自己检查一遍依赖图。

**必须用 GitHub Issues 吗？**
不用。任何 issue 跟踪器都行。GitHub 是支持最好的路径，因为它的原生子 issue 和阻塞关系让前沿无需打开地图也能看见；GitLab、Linear、Jira 和本地 markdown 都有人在用。两个诚实的提醒。没有原生阻塞关系的跟踪器，依赖图要从文本推断，需要手工纠正。本地 markdown 会把产物放进你的仓库，而这并不推荐：把这类材料存进仓库容易导致意外持久化。开源维护者遇到的是反向问题（公开跟踪器被 Agent 生成的规划工单淹没），反而倾向于选本地 markdown。

**追问太累了。每个问题都有三段那么长。**
这是目前对 wayfinder 最尖锐的抱怨，尚未解决。有用户拆解过：冗长本身造成决策疲劳，而且长度吃掉了*为什么问这个问题*，当地图变长，你就丢了决策之间的链条。这种冗长看起来是当前一批[模型 model](https://www.aihero.dev/ai-coding-dictionary/model) 的属性，而不是技能的属性，目前没有修复落地。流传中的实践缓解办法：用更低的[推理投入 reasoning effort](https://www.aihero.dev/ai-coding-dictionary/effort)，并在全局 `CLAUDE.md` 里加一句大白话指令。但无论如何都要准备花真心思，因为 wayfinder 要求你投入的大量思考不是缺陷，那正是它的大部分价值所在。

**已经关闭的决策后来发现错了。我改旧工单还是建新工单？**
没有官方指引，Agent 的直觉还帮倒忙：它倾向于绕着错误决策打补丁，而不是挑战它。有效的做法是直接告诉 wayfinder 发生了什么变化；它会更新地图，修订受影响的工单，并在已关闭的工单下评论。地图中途改范围是可恢复的。但一张*设计成会变*的地图是范围没定好的气味。

**`decision-mapping` 去哪了？**
就是这个技能，在 v1.1 改名为 `wayfinder`，用 `/wayfinder` 调用。“Decision map”是黑话，而且不准确，因为四种工单里只有一种真正算决策。改名后技能有了一套连贯词汇（目的地、战争迷雾、前沿、地图），而不是叠在上面的生造词。但单元名称保留了“decision”一词：wayfinder 工单就叫**决策工单（decision ticket）**，正是为了防止人们把它当成实现工单。

## 达到这些就说明它正常工作

- 在第一张工单存在之前，目的地已经写下来并达成一致。
- 每个开放工单读起来都是一个问题。任何读起来像“构建 X”的工单，要么类型标错，要么属于地图下游。
- 不打开地图，只看跟踪器就知道哪些工单可接，因为前沿已经通过原生阻塞关系自己渲染出来。
- 一个会话解决一张工单，把答案作为解决评论发出，关闭它，并在地图的*已做决策（Decisions so far）*上留一行。然后停下。
- **尚未明确（Not yet specified）**随时间缩小。从迷雾区毕业成工单的内容会从该节消失，而不是两头并存。
- 当开头的广度优先追问完全没有翻出迷雾，技能停下并告诉你：工作量小到可以跳过地图。
- 走完地图的会话把你送往规格文档，而不是 pull request。

## 它在整体中的位置

`wayfinder` 是**特定场景的入口（situational on-ramp）**，不是默认前门。以追问为起点的想法到上线主链仍然是大多数工作的起点；wayfinder 是想法大到单个会话装不下时才爬上去的岔路，它在 [to-spec](https://aihero.dev/skills-to-spec) 处汇回主链，因为走通的地图负责交接，不负责构建。

往下看，它大多是穿着 wayfinder 调度外衣的其他技能：[追问 grilling](https://aihero.dev/skills-grilling)和[领域建模 domain-modeling](https://aihero.dev/skills-domain-modeling)解决默认工单类型，[原型 prototype](https://aihero.dev/skills-prototype)解决光靠聊解决不了的工单，[研究 research](https://aihero.dev/skills-research)作为子智能体运行，以免它的阅读材料落进你的会话。[handoff](https://aihero.dev/skills-handoff)是进出地图的桥：从膨胀的对话进入地图，会话中出现支线时离开地图。其他情况由 [ask-matt](https://aihero.dev/skills-ask-matt) 在全套技能中分流。
