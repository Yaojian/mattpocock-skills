## 功能简介

`ask-matt` 是本仓库中各技能之上的路由器。你描述自己所处的处境（有一个无法启动的想法，一堆涌入的缺陷报告，一个已经跑了很久的[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)），它会给出匹配的技能或技能序列，并指出该序列中需要由人做决定的位置。

它只负责推荐，然后停下。它不会追问 (grill)，不会写[规格说明 (spec)](https://www.aihero.dev/ai-coding-dictionary/spec)，不会打开文件，也不会直接执行它刚推荐的技能；你得到的只是下一步要输入的内容，然后由你亲自输入。它也是手写维护的本仓库技能地图，而不是扫描你已安装内容的产物，所以它不会基于你自己的技能或其他作者的技能来做路由。

## 何时使用

输入 `/ask-matt` 即可调用；智能体不会主动调用它。

| 你的处境 | 路由器会返回什么 |
| --- | --- |
| 有一个想法，但不知道从哪里开始 | 主流程的起点，以及当前的构建是否足够小、可以跳过规格说明 |
| 来自他人的缺陷和需求不断涌入 | [分诊 (triage)](https://aihero.dev/skills-triage) 入口，以及为什么你自己生成的[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)不属于这条入口 |
| 有两个看起来可以互换的技能 | 它们之间的分界线，通常是一个具体的判断标准，而不是品味问题。[grill-me](https://aihero.dev/skills-grill-me) 还是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，取决于你是否在工作目录中；[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 还是 [wayfinder](https://aihero.dev/skills-wayfinder)，取决于工作量能否装进一个会话 |
| 会话已经很长，需要决定[上下文 (context)](https://www.aihero.dev/ai-coding-dictionary/context)怎么处理 | 阶段边界上五个选项的有序决策树 |
| 你已经选好了技能 | 它帮不上忙。直接调用那个技能。 |

## 前置条件

路由器只负责点名技能，不负责安装。要让推荐可以真正执行，它指向的所有技能都必须已经安装，而且它只认识本仓库中已推广的技能。

依赖任务追踪器的路由（triage、`to-spec`、`to-tickets`、`implement`）都假设 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 已经在仓库里配置好了议题追踪器。在这件事完成之前，路由器仍然会照常推荐它们。

## 流程，而非单个技能

这个技能交给你的思考用词是**流程 (flow)**：一条*穿过*多个技能的路径，而不是某一个技能。说出你的处境，就等于把你放到某条流程的某一步上，这和“按关键词给你一个技能”是不同的答案。共有四类路由，技能本身带有完整说明：

- **主流程**，从想法到上线。追问 (grill)、规格说明、工单、实现、评审，中间有两个分支：当某个问题需要用可运行代码才能定论时的原型旁路；以及规格和工单拆分，只有当构建跨越多个会话时才值得付出这份成本。
- **入口 (On-ramps)**，针对先产生工作、再并入主流程的处境：收到的缺陷报告、已经出故障的东西，或者模糊且庞大、一个会话装不下的工作。
- **独立技能 (Standalones)**，在所有流程之外，按自身条件单独使用：原型、问卷、你已经身处其中的合并冲突。
- **底层的词汇层**，另外两个参考型技能，当问题出在措辞而不是流程上时，其他技能会引用它们。

## 阶段边界

它交给你的另一个概念是**阶段边界**。阶段是会话内部的一段工作（[追问 (grilling)](https://www.aihero.dev/ai-coding-dictionary/grilling)、实现、QA），两个阶段之间的边界是唯一适合问“这个上下文怎么办”的地方。在阶段中途没有什么可决定的：要么继续，要么把剩下的工作拆给[子智能体 (subagent)](https://www.aihero.dev/ai-coding-dictionary/subagent)。

| 选项 | 选用时机 |
| --- | --- |
| **继续 (Continue)** | 下一阶段需要原样使用当前上下文，或者你还剩有[高效区 (smart zone)](https://www.aihero.dev/ai-coding-dictionary/smart-zone)。它是唯一能让当前会话保持为[一手来源 (primary source)](https://www.aihero.dev/ai-coding-dictionary/primary-source)的走法，所以先排除它 |
| **`/clear`** | 背后的所有内容都可以丢弃。棋盘上最便宜的一步，但如果判断错了就无法回头 |
| **[交接 (handoff)](https://aihero.dev/skills-handoff)** | 有东西必须带走：新的[宿主框架 (harness)](https://www.aihero.dev/ai-coding-dictionary/harness)、新的目录、一位同事、在阶段中途分叉出去的支线任务 |
| **子智能体 (Subagent)** | 任务范围足够收敛，可以在你[离开键盘 (away from the keyboard)](https://www.aihero.dev/ai-coding-dictionary/afk)期间独立运行 |
| **`/compact`** | 以上都不符合。默认选项，实际也经常落到这里 |

其中两个选项经常被用错，所以路由器给出的是顺序而不是列表。`/handoff` 看起来像是窗口之间的通用桥梁，其实不是：可移植性是它带来的全部价值。`/compact` 是决策树的底部而不是首选，因为它前面的四个问题各自更便宜或更精确。

## 常见问题

**难道就没有一份按正确顺序排列的技能清单吗？**

README 里一直有人要这份清单。这个技能就是那份清单，它为此而存在。静态表格会写成 `wayfinder → to-spec → to-tickets → implement → code-review`，但对大多数情况都是错的，因为有意思的部分全在分支里：有没有代码库，构建是否跨会话，这个问题靠聊能不能聊定。诚实的代价是路由器靠手工维护，会落后于仓库。`/grilling` 和 `/resolving-merge-conflicts` 都是发布了很久之后路由器才收录它们。

**它说有一半技能没安装。**

这是已知的缺陷，还没修。路由器要经过的大多数技能都设置了 `disable-model-invocation: true`，这意味着宿主框架在注入给智能体的技能列表时会略过它们。智能体会把这份列表当成全量，然后报告它们缺失。有一次上报的会话里，它宣布整个规格和工单流程都不存在，改道去用裸 `/grilling` 和 `/tdd`。插件 22 个技能里有 13 个带这个标记，所以这是常态而不是边角情况。它们其实都装着。直接输入斜杠命令就行，也可以查 `.claude-plugin/plugin.json`，它才是记录装了什么的权威来源。

**它描述的某个技能行为，和技能实际行为对不上。**

这也是真的，也没修。路由器是根据自己手里对每个技能的一句话摘要作答，而不是根据技能原文。有一份详细报告追踪了单次会话里的三个例子，包括根据“把讨论串变成规格说明”这句简介建议跳过 [to-spec](https://aihero.dev/skills-to-spec)：那次根本没打开过 `to-spec/SKILL.md`。每次都是用户质疑之后它才去核实，从不会主动核实。那次跳过 `to-spec` 付出过真实代价，漏掉了一处真正的接缝检查，产出的工单也低估了工作量。当路由器断言另一个技能的关键行为时，先让它打开那个 `SKILL.md`。地图完全没覆盖的问题也一样，比如是否使用[计划模式 (plan mode)](https://www.aihero.dev/ai-coding-dictionary/agent-mode)：那个答案是[模型 (model)](https://www.aihero.dev/ai-coding-dictionary/model)的推断，不是在这里写下的内容。

**为什么是散文，而不是带编号的检查清单？**

这是合理的抱怨，已经作为公开议题提出，理由是大部分路由是确定性的，叙述体不好扫读。你完全可以直接要压缩版：“只给我序列”就能拿到序列。散文承载的是条件那一半：分支在哪里，哪里需要人做决定，步骤之间哪里该清理或压缩上下文。扁平的检查清单恰恰会丢掉这些。

**它能基于我自己的技能或其他作者的技能做路由吗？**

不能。已经有三个不同的提案希望要一个能读取本地 `skills/` 目录、基于已安装内容推荐的路由器。`ask-matt` 不是那个。它是一份手写维护的单一技能集合地图，对你自己写或从别处安装的技能一无所知。

**它让我去改一个 SKILL.md。**

这个建议常常是对的，但很少能持久。有人问怎么让 [implement](https://aihero.dev/skills-implement) 自动关闭工单，被告知在技能里加一行，对方立刻发现了问题：`npx skills update` 会覆盖这个文件，而且插件安装是只读的。把长期行为放进你自己的 `CLAUDE.md` 或 `AGENTS.md`，或者写在调用语里。调用层的适配经得起更新：把流程指向 Linear 而不是 GitHub，或者问它哪些未完成工单可以并行，都是人们常用的做法。

**它点名了一个我没有的技能，或者漏掉了我有的技能。**

先查更新日志里的改名记录，再认定它没了。`writing-great-skills` 改名为 [writing-for-agents](https://aihero.dev/skills-writing-for-agents)，没有别名，`to-prd` 改名为 [to-spec](https://aihero.dev/skills-to-spec)，`pathfinder` 改名为 [wayfinder](https://aihero.dev/skills-wayfinder)。有四个技能被直接退役，合并进了吸收它们的技能：`ubiquitous-language`、`design-an-interface`、`qa` 和 `request-refactor-plan`。反过来，路由器点名缺失则属于上面说的路由器滞后。

## 怎样算生效

- 它以点名要输入什么收尾，并且停在那里，而不是自己动手开干。
- 它给出的路线会说明哪里清理或压缩上下文、哪里需要人来评审，而不只是列一串技能名。
- 遇到两个相近的技能，它会说选哪一个，以及为什么另一个不适合你。
- 它对另一个技能行为的任何断言，在执行轨迹里都能看到它读过那个技能的 `SKILL.md`。
- 你在它返回的内容里能认出自己的处境，而不是最接近的通用场景。

## 在整体中的位置

`ask-matt` 是覆盖全套技能的**独立路由器**。它从不是链条中的一步；它指向每条链条，其他文档页都会链回这个节点，这样谁都不用重画整张图。从这里出发，最常落到 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)（主流程的起点）或 [triage](https://aihero.dev/skills-triage)（处理别人交过来的工作的入口）。

它是其所描述技能之上的[二手来源 (secondary source)](https://www.aihero.dev/ai-coding-dictionary/secondary-source)。路由器和某个 `SKILL.md` 不一致时，以 `SKILL.md` 为准。
