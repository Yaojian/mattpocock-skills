## 功能简介

`to-tickets` 把一份计划、一份[规约 (spec)](https://www.aihero.dev/ai-coding-dictionary/spec) 或当前对话拆成问题跟踪器上的一组**[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)**。每张工单声明它的**阻塞边 (blocking edge)**：必须先完成才能启动它的其他工单。

每张工单都是一颗**示踪弹 (tracer bullet)**：穿过改动每一层（Schema、API、UI、测试）的窄而完整的路径，一落地就能独立演示。这正是它与直观切分方式的不同之处，直观做法是一次切一层，最后再集成。它还把每张工单切到一个全新的[上下文窗口 (context window)](https://www.aihero.dev/ai-coding-dictionary/context-window)能装下的大小，因为领取工单的是从没见过你的规约的[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)。

## 何时使用

输入 `/to-tickets` 即可调用。[智能体 (agent)](https://www.aihero.dev/ai-coding-dictionary/agent)不会自动选用它。

| 你的处境 | 该运行什么 |
| --- | --- |
| 你有规约 issue，且构建横跨多个会话 | `/to-tickets`，或 `/to-tickets #<spec_issue>` |
| 计划只在对话中，从未写下来 | `/to-tickets` 直接读对话，不需要规约 |
| 整个改动适合一个上下文窗口 | [implement](https://aihero.dev/skills-implement)，跳过工单 |
| 还没有决定任何事 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，然后用 [to-spec](https://aihero.dev/skills-to-spec) |
| [wayfinder](https://aihero.dev/skills-wayfinder) 地图已完成 | 先用 [to-spec](https://aihero.dev/skills-to-spec) 收敛地图，再用 `/to-tickets` |

`to-tickets` 产出的工单按构造就是智能体可执行的。不要对它们运行 [triage](https://aihero.dev/skills-triage)。分诊处理的是别人送来的工作。

## 前置条件

`to-tickets` 发布到跟踪器中，所以 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须先为本仓库配置好跟踪器和分诊标签词汇。两种都可以：GitHub 或 Linear 这样的真实跟踪器，或者 `.scratch/` 下的本地 markdown 文件，开箱即用。

## 示踪弹，而不是分层

**水平 (horizontal)** 切片一次交付改动的一层。在每一层都落地之前，没有任何东西可用，而且每张工单的验收标准不得不伸手去够别的工单拥有的工作。**垂直 (vertical)** 切片（示踪弹）一次交付穿过所有层的薄路径，因此可以独立验证，评分所依赖的东西都归自己所有。

这是大家最常违反的规则，后果也有充分记录。一个团队曾跑过一个按层切分（语料、生产者、聚合器、选择器）的 26 张工单栈，平均每关闭一张工单约花二十次智能体运行，其中约四分之三是返工。他们自己的事后复盘把每个失败类别都追溯到水平切片，而不是实现本身。

在发布之前还有两件事。`to-tickets` 先寻找预重构机会（原则是“先让改动变容易，再做容易的改动”），并把这类工作排在最前。然后它把拆分结果以编号列表交给你，并就此追问你：粒度对吗，阻塞边真实吗，有没有该合并或拆分的地方。未经你批准，任何内容都不会进入跟踪器，而这次追问正是表达反对意见的位置。

## 阻塞边

阻塞边的存放位置取决于跟踪器，读法有两种：

| 跟踪器 | 阻塞边存在哪里 | 如何推进 |
| --- | --- | --- |
| 本地 markdown | `.scratch/<feature>/issues/<NN>-<slug>.md` 下每张工单一个文件中的文字，按阻塞优先编号 | 自上而下手工推进 |
| 真实跟踪器（GitHub、Linear） | 原生阻塞链接，或跟踪器支持时的子 issue | 阻塞项已完成的工单都在**前沿 (frontier)** 上，可以领取 |

无论如何，阻塞边都在工单里。介质只决定能否并行推进。`to-tickets` 只产出工件；运行它们（一次一个会话，或一个机群）是你的工作，不是技能的工作。

## 宽重构例外

有一种形状打破示踪弹规则。**宽重构 (wide refactor)** 是单个机械改动（改名一列、改一个共享符号的类型），其**爆炸半径 (blast radius)** 铺满整个代码库，一处编辑弄坏数千调用点，没有垂直切片能独立变绿落地。

`to-tickets` 改用**扩展收缩 (expand-contract)** 来排序：

- **扩展 (Expand)**：把新形式加在旧形式旁边，保证什么都不坏。
- **迁移 (Migrate)**：按爆炸半径分批搬调用点（按包、按目录），一批一张工单，每张都阻塞于扩展。因为旧形式还在，CI 保持绿色。
- **收缩 (Contract)**：等调用者清空后删除旧形式，工单阻塞于每个迁移批次。
- 其中连分批都无法独立变绿时，它们共享一个集成（integration）分支，并全部阻塞一个最终的集成验证工单。绿色只在那里承诺。

## 常见问题

**三行改动，它产出了十二张工单。**
过度拆分是本技能被反馈最多的摩擦点，而且在不同实践者之间表现一致：[模型 (model)](https://www.aihero.dev/ai-coding-dictionary/model)默认切成原子单元，丢掉了本可赋予意义的分组。追问步骤正是为此而设：让它合并，它就会合并。更深层的答案是工单有下限：如果整个改动适合一个上下文窗口，你根本不需要本技能，直接用 [implement](https://aihero.dev/skills-implement)。

**工单一层一张：Schema 全在一张，API 全在另一张。**
这正是垂直切片规则要防范的失败，而技能有时还是会产出它。在追问步骤抓住它，逐张工单问一个问题：做完这张，我能演示什么？答不上来的工单就是水平切片。有人因此给每张工单加一行“演示路径”，据说这能把模型推向垂直拆分。

**在 GitHub 上，工单没有建成规约 issue 的子 issue。**
已知，未修复。十几次运行、多个模型都报告过，[issue #554 里的报告最完整](https://github.com/mattpocock/skills/issues/554)，在 Codex 上比在 Claude 上更严重。`gh` 从 v2.94 起已原生支持：`gh issue create --parent <n>`，事后可用 `gh issue edit <parent> --add-sub-issue <n>` 补链。在跟踪器模板优先支持它们之前，跑完后手工连父子链接是最可靠的做法。

**“Blocked by”写进了 issue 正文，而不是真正的阻塞链接。**
同类问题，[记录在 issue #513](https://github.com/mattpocock/skills/issues/513)，当时智能体甚至断言 GitHub 根本没有原生阻塞关系。其实有：`gh issue create --blocked-by 12,15`。因为阻塞项先发布，创建时它们的编号一定已知。正文文字本是给没有原生阻塞边的跟踪器准备的回退方案，不该是默认。

**本地工单去哪里了？v1.1 说明里说根目录有个 `tickets.md`。**
说明确实这么写过，那是个 bug：并行智能体同时写入单个共享文件会竞态。本地模式现在按依赖顺序，每张工单一文件，写在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md` 下，与本地跟踪器模板描述的布局一致。`NN` 前缀是真实工单 ID，所以 `/implement 03` 可用，不用重敲长标题。

**它读我的规约时一直在截断。**
很大的规约可能超出跟踪器 issue 一次能干净返回的长度，而且没有本地副本可回退，智能体随后把[工具调用 (tool call)](https://www.aihero.dev/ai-coding-dictionary/tool-call)烧在分块重取上，永远走不到结尾。不要在 `/to-spec` 和 `/to-tickets` 之间[清理 (clearing)](https://www.aihero.dev/ai-coding-dictionary/clearing)或[压缩 (compaction)](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一个上下文窗口内连着运行两者，规约就根本不需要取回。

**验收标准什么都没测：有些在开工前就通过了。**
模板只要求写标准，没说它们能否失败，所以这种情况会发生。三种形状反复出现：在基准提交上已为真的标准、只能由别的工单拥有的工作来满足的标准、复述需求而不是从产物推导的标准。垂直切片能防住大半（交付此前不存在的行为的切片，按构造在基准提交上就是红的），但仍值得手工检查。对每条标准，说出能证明它为假的观察，并确认它在实现者出发的提交上确实失败。

**工单已发布，我实际怎么跑？**
技能止于工件，没有自动派发模式。派发是手工的：看板，找到没有未完成阻塞项的工单，数一数，开同样数量的智能体会话。一张工单一会话，会话之间清空上下文。注意 [implement](https://aihero.dev/skills-implement) 在 GitHub 或本地 markdown 上都不可靠地关闭或勾选工单，所以工单状态由你更新。

## 生效标志

- 每张工单都能回答“做完这张，我能演示什么？”，答案是行为，而不是一层。
- 列表以编号形式回到你面前，每张带有“Blocked by”行，然后才发布任何内容。
- 最上面的工单没有阻塞项，可以立即开工。
- 工单正文中没有任何文件路径或行号，原型产出的片段除外。
- 每张工单读起来都像全新会话在你不在场时也能做完。
- 找到的预重构排在顺序最前，而不是混在功能工单里。

## 在流程中的位置

`to-tickets` 是主构建链中的一步：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

上游是 [to-spec](https://aihero.dev/skills-to-spec)，它交来一份已稳定的规约供切片；两者保持在同一个不间断的上下文窗口内。下游是 [implement](https://aihero.dev/skills-implement)，它按全新会话一张工单地构建，用 [tdd](https://aihero.dev/skills-tdd) 驱动测试，以 [code-review](https://aihero.dev/skills-code-review) 收尾。当不确定哪个技能或流程合适时，由 [ask-matt](https://aihero.dev/skills-ask-matt) 指路。
