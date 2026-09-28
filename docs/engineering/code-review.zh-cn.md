## 功能简介

`code-review` 沿两个轴评审 `HEAD` 与你指定的固定点（某个提交、分支、标签、`main`、`HEAD~5`）之间的差异。**规范 (Standards)** 问的是代码是否符合本仓库的写法。**规格 (Spec)** 问的是代码是否做了源头议题或[规格说明 (spec)](https://www.aihero.dev/ai-coding-dictionary/spec)要求的事。每个轴跑在各自的[子智能体 (sub-agent)](https://www.aihero.dev/ai-coding-dictionary/subagent)里，互不可见对方的推理过程。

两个轴的结果永远不合并，也永远不重新排序。报告最后给出每个轴各自的最严重问题，拒绝在两个轴之间评出唯一的“冠军”，因为一次变更可能通过一轴而挂掉另一轴：完全遵守约定却实现错东西的代码，能过规范轴，挂规格轴；完全按[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)做了、却破坏仓库约定的代码则反过来。混合成一个总分，只会让通过的那一轴掩盖挂掉的那一轴。

## 何时使用

输入 `/code-review`，或者当你要求评审分支、PR、进行中的工作或任何“自从 X 以来”的内容时，智能体会自动调用它。

| 你的处境 | 该用哪个 |
| --- | --- |
| 差异已经存在，想知道它是不是“做对的事、又把事做对” | `code-review` |
| 想在差异里猎捕缺陷：空指针路径、竞态、差一错误 | 用 Claude Code 自带的评审，而不是这个（见下面的重名问题） |
| 还没写代码，想用测试先行的方式写出来 | [tdd](https://aihero.dev/skills-tdd) |
| 要按完整规格构建，评审包含在内 | [implement](https://aihero.dev/skills-implement)，它自己就会调用本技能 |
| 是整个代码库走形了，而不是某一个差异 | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) |
| 有东西坏了，但不知道原因 | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

你必须给出固定点。不给的话，技能会向你要，而不是猜；然后它会先检查引用能否解析、差异是否非空，再派生子智能体，所以写错分支名会当面报错，而不是在两个子智能体内部失败。

## 前置条件

规范轴不需要任何东西。它读取仓库里已有的文档（`CODING_STANDARDS.md`、`CONTRIBUTING.md` 之类），仓库什么都没写时就回退到内置基线。

规格轴需要规格说明真实存在且找得到。查找顺序如下：

1. 提交信息里的议题引用（`#123`、`Closes #45`、GitLab 的 `!67`），经由 `docs/agents/issue-tracker.md` 获取。
2. 你作为参数传进来的路径。
3. `docs/`、`specs/` 或 `.scratch/` 下面与分支或功能名匹配的规格文件。
4. 直接问你。

第 1 步依赖 `docs/agents/issue-tracker.md`，这个文件由 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 生成。没有它，只要你手工给路径，这一轴照样能跑。完全没有规格时，就跳过规格子智能体，报告里写“没有可用规格”，而不会凭空编造需求。

## 两个轴

| | 规范 (Standards) | 规格 (Spec) |
| --- | --- | --- |
| 问题 | 做得对不对？ | 是不是该做的事？ |
| 读取 | 仓库里写下的规范，外加坏味道基线 | 源头议题或规格说明 |
| 报告 | 违反成文规范的问题（可以判硬伤），以及坏味道（永远是判断题） | 缺失或只做了一半的需求、范围蔓延、需求实现错误 |
| 每条发现必须引用 | 仓库规范文件加规则，或具名的坏味道加代码块 | 规格说明中的对应行 |

这个设计要避开的，正是那种不了解你家规范的通用评审技能：它会把你代码库里故意的写法当错，把你代码库真正依赖的不变量漏掉。所以在规范轴上，仓库自己的文档是[一手来源 (primary source)](https://www.aihero.dev/ai-coding-dictionary/primary-source)，**永远以仓库为准**。

**坏味道基线**是它下面的地板，取自《重构》第 3 章的十二种 Fowler 代码坏味道：Mysterious Name（名不副实）、Duplicated Code（重复代码）、Feature Envy（依恋情结）、Data Clumps（数据泥团）、Primitive Obsession（基本类型偏执）、Repeated Switches（重复的 switch）、Shotgun Surgery（散弹式修改）、Divergent Change（发散式变化）、Speculative Generality（过度保守的通用性）、Message Chains（消息链）、Middle Man（中间人）、Refused Bequest（被拒绝的遗赠）。每一种都是标注好的启发式判断（“疑似 Feature Envy”），永远不是硬性违规，每一条都按“*是什么* → *怎么改*”表述，所以每条发现到你手里都附带走法，而不是一句抱怨。两个轴都会跳过你的 linter 已经强制的东西。

## 常见问题

**它和 Claude Code 自带的 `/code-review` 撞名了，怎么办？**

这是本技能被上报最多的问题，还没修。Claude Code 自带的 `/code-review` 做的是另一件事：在差异里猎捕缺陷，而这个技能检查的是规格符合度和仓库规范。装了这套技能库之后，总有一个会赢，哪个赢取决于安装方式。经插件市场安装，所有技能都会加上 `mattpocock-skills:` 前缀，不带前缀的内置版反而难够到；经普通技能安装，本地文件胜出，这个技能会遮掉内置版。一个干净的解法是把 Claude Code 自带的技能整个删掉：省下一大块[上下文 (context)](https://www.aihero.dev/ai-coding-dictionary/context)，撞名问题也就不重要了。遮蔽本身可以说是 Claude Code [宿主框架 (harness)](https://www.aihero.dev/ai-coding-dictionary/harness) 的缺陷（技能作者应该可以自由取名），所以另一个解法是给本地副本改名。改 frontmatter 或改目录名会被 `npx skills update` 还原；用户上报过的持久 workaround 是 fork 出一个新名字的技能，把托管集合里的 `code-review` 拿掉，并记下 fork 时的提交，方便以后手工同步。

**它的子智能体老是再调用 `/code-review`，然后生出更多智能体。**

已知的公开缺陷，多人复现过，也不止一种宿主框架。规范和规格两个子智能体提示词都没禁止委派，所以子智能体可能重新发现这个技能再扇出一次：有一份报告跑到了 50 多个智能体。fork 上人们实际采用的修复，是在两个子智能体简报末尾各加一行：“不要调用 `/code-review`，也不要派生更多智能体，直接执行这次评审。”也有人更愿意在宿主框架层统一加防护，让所有技能继承。两种都没进 shipped 的技能。如果你无人值守跑它，盯一下智能体数量。

**应该在写代码的同一个[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)里跑它吗？**

建议开新会话。用一位读者的话说：“同一个上下文自己审自己不是评审，是带斜杠命令的确认偏误。”写代码会话里的评审智能体，带着塑造这份代码的全部假设，而独立评审人恰恰没有这些上下文。这也是有人要求 [implement](https://aihero.dev/skills-implement) 去掉内置评审步骤的原因：它在刚写完差异的会话内部跑评审。你自己从干净会话调用 `/code-review` 才是诚实的版本。

**每个工单后审一次，还是一批做完统一审？**

两种都行，技能不替你选。按工单审能让每个差异足够小，规格轴每次只对一份清晰的规格，这也是 `implement` 采用的模式。攒到分支末尾批量审，能抓住单张工单各自通过、合起来却互相打架的问题。拿不准就按工单审，最后再对分支起点跑一遍全量。

**它的发现可信吗？**

不核实就不能信。子智能体的输出是假设，不是证据：有一个团队报告过，基于正文的评审放行了十几个破坏性变更。本技能只是把两份报告原文或轻度整理后汇总，不会逐条回文件里复核，所以某条发现可能引用错位置或夸大影响。处理每条发现之前，先读它的引用。好在每条发现都被强制要求带引用（成文规范的规则、坏味道加代码块，或规格行），这才让核查成为可能。

**为什么每次跑它都找出新问题？**

因为修的地方会制造新的表面，也因为规范轴里判断题那一半在多次运行之间本来就不确定。一位读者描述得很直白：“/code-review 和 /improve-code-architecture 每次都找出新东西。我修完再跑，修完再跑，没完没了。”这里没有收敛保证。把一次通过当成线索清单处理，有成文规则背书的先改，然后停下：不要循环跑到它干净为止，因为它永远不会干净。

**它审我没提交的工作吗？**

不审。它 diff 的是 `<fixed-point>...HEAD`，三点式，从合并基点算起，不含暂存区和工作区改动。如果 `implement` 没做中间提交，那么正要提交的工作对评审是不可见的。先提交，再评审，然后 amend 或加 fixup。

## 怎样算生效

- 引用写错或差异为空时，它在派生任何子智能体之前就拒绝启动。
- 报告是 `## Standards` 和 `## Spec` 之下两个独立区块，而不是合并后的一个清单。
- 每条规范发现要么点名你仓库某文件的某条规则，要么点名十二种坏味道之一，并引用代码块；每条规格发现都引用规格中的一行。
- 结尾总结给出每个轴的最严重问题，并拒绝评出总冠军。
- 没有可用规格时，规格区块如实说明，而不是从代码反推需求列出来。

## 在整体中的位置

`code-review` 是构建链条尾部的评审步骤：`grill-with-docs → to-spec → to-tickets → implement → code-review`。它也可以独立用在你指向的任何分支或 PR 上。

- [implement](https://aihero.dev/skills-implement) 是最近的邻居：它驱动构建，并在提交前把本技能作为收尾评审来调用。
- [to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets) 产出规格轴要对照的文档；规格含糊，这一轴就含糊。
- [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是整个代码库层面的对应物：本技能永远只看一个差异。

拿不准当前处境该用哪个技能时，用 [ask-matt](https://aihero.dev/skills-ask-matt) 做全套路由。
