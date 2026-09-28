## 它的作用

`improve-codebase-architecture` 会在代码库中巡检**加深机会 (deepening opportunities)**：那些接口几乎和它隐藏的东西一样复杂、本可以从浅模块变成深模块的地方。它把结果写成一份自包含的 HTML 报告，然后[追问 (grill)](https://www.aihero.dev/ai-coding-dictionary/grilling)你，带你深入你选中的那一条。

它从不改代码。整次运行只产出一个放在操作系统临时目录里的 HTML 文件加一段对话；真正的重构发生在之后，在另一个[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)里走正常构建流程。这正是它被称为巡检而不是重构工具的原因，也是为什么在你还没准备好动手时也值得跑一次。

有两个过滤器防止报告变成泛泛的清理建议。每个候选项都必须通过**删除测试 (deletion test)**：删掉这个模块，是把复杂度收敛到一个更小的接口后面，还是仅仅把它摊给调用方？只有“收敛”的情形才配拥有一张卡片。除非你把它指向某个具体区域，否则它会先读最近的提交历史，把扫描偏向正在活跃变化的路径，理由是在没人碰的代码里做加深，是永远无法兑现的重构。

## 什么时候用它

由你输入 `/improve-codebase-architecture` 来调用；[智能体 (agent)](https://www.aihero.dev/ai-coding-dictionary/agent)不会自行调用它。

它在构建循环之外：它不是主循环中的一步，而是你定期运行、为改善代码库排好更多工作的东西。它常用的四种情形：

| 情形 | 用法 |
| --- | --- |
| 日常维护 | 每隔几天跑一次，或一有空就跑，防止结构在功能之间腐烂。 |
| 大型构建之前 | 把[规格说明 (spec)](https://www.aihero.dev/ai-coding-dictionary/spec)指给它：“怎样才能让这次变更更容易？”这是对它最有效的提示。 |
| 存量审计 | 在大型、无结构或 [vibe coding](https://www.aihero.dev/ai-coding-dictionary/vibe-coding) 的仓库上跑，看清它实际是什么形状。 |
| 存量测试工作 | 先用它找到缺失的接缝，再对不可测试的代码写测试。 |

容易和兄弟技能混淆的地方：

- 如果是设计一个你已经选定的模块，用 [codebase-design](https://aihero.dev/skills-codebase-design)：那是工作台，这个技能是巡检，负责找出该放上工作台的东西。
- 如果是单会话装不下的整体工作，用 [wayfinder](https://aihero.dev/skills-wayfinder)。
- 如果是“某个具体的东西坏了”，用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)。当真正的发现是没有合适的接缝来锁定 bug 时，它会把你交回这里。

## 前提条件

运行它不需要前提。它会读 `CONTEXT.md` 和 `docs/adr/` 下已有的 ADR（如果存在），并用你领域自己的名词说话：候选项读起来是“加深 Order 接入模块”，而不是“重构 FooBarHandler”。

它写两个地方。报告写到 `<tmpdir>/architecture-review-<timestamp>.html`，在仓库外面。在追问循环中，它会向 `CONTEXT.md` 新增或打磨术语（文件不存在就创建），并提议把一条被否决的候选项记为 ADR，这样未来的运行不会重复推荐它。

## 深度，以及寻找深度的报告

这个技能围绕一个概念：**深度 (depth)**。深模块把大量行为藏在一个小而稳定的接口后面。浅模块把实现透过一个几乎和底下代码一样宽的接口漏出来。报告寻找三种形态的浅：只为可测试性抽出的纯函数，而真正的 bug 藏在调用方式里（没有**内聚 locality**）；跨越**接缝 (seam)** 漏水的模块；以及不打开五个文件就看不懂的概念。最后它会为修复方案提出一个加深建议。

每个候选项是一张卡片：涉及的文件、摩擦点、白话解决方案、用**内聚 (locality)** 和**杠杆 (leverage)** 表述的收益、前后对比图，以及一个强度徽标。

| 徽标 | 对你的含义 |
| --- | --- |
| `Strong` | 明确通过删除测试，摩擦真实存在。认真对待。 |
| `Worth exploring` | 看似合理的加深，但回报取决于代码下一步往哪走。 |
| `Speculative` | 为完整性列出，大多可以安全忽略。 |

报告以一条**首要推荐 (Top recommendation)** 结尾（它会先啃哪一条），然后技能停下来，问你想深入哪条。到这一步什么都没定，也没有代码被搬动。

## 选中一条之后发生什么

选中一条候选项，就会开启一场围绕它的[追问 (grilling)](https://www.aihero.dev/ai-coding-dictionary/grilling)会话：约束、接缝后面是什么、哪些测试能留、加深后的接口应该长什么样。这场会话的产出是一项决策，而不是一个 diff。之后走正常流程：把决策带进 [to-spec](https://aihero.dev/skills-to-spec)，再到 [to-tickets](https://aihero.dev/skills-to-tickets)，再到 [implement](https://aihero.dev/skills-implement)。

## 常见问题

**它围着一个想法追问了我一小时，而不是给我选项，能关掉吗？**

可以：调用时就说（“别追问我，只给我看报告”）。这是这个技能最响亮的抱怨。一位用户直言：他们喜欢把它当作“拿到改进项彻底分析的方便办法”，加上追问循环后觉得它“几乎没法用”，报告说有会话只提一个方案，然后问了“几十上百个问题”。设计意图是报告先行，只对你选中的候选项开始追问，但较弱的[模型 (model)](https://www.aihero.dev/ai-coding-dictionary/model)会跳过报告，直接围着自己的第一个想法采访你。同一帖子里的体验随模型差异很大，这是未解决的问题 (open issue)：技能目前还没有书面化的免追问模式。

**报告打开是无样式的原始 HTML，没有图，发生了什么？**

报告从 CDN 加载 Tailwind 和 Mermaid，所以打开时需要联网，拦截脚本的东西会让它静默损坏。已记录的案例是一个安全钩子要求 SRI 哈希：智能体加上了哈希，但 CDN 给浏览器和给计算哈希用的 `curl` 返回了不同字节，浏览器拦截了脚本。离线和锁网环境会撞上同一堵墙。智能体看不到这个问题，因为它从不渲染页面。变通办法是要求内联 CSS 和手写 SVG 图，替代 CDN 脚手架。这是未解决的问题 (open issue)，也是真正的粗糙之处。

**它给了我十二条候选项，在同一个会话里逐条过，还是开新会话？**

一条候选项一个会话。在一段对话里连做几条，会把报告、追问、领域模型修改和代码改动全塞进一个[上下文窗口 (context window)](https://www.aihero.dev/ai-coding-dictionary/context-window)。报告只住在临时文件里，所以带走候选项本身而不是文件：选一条，追问它，把决策带进 `/to-spec`，其余变成[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)，以后独立认领。把选中的改进放进规格说明，而不是直达实现。这是反复被问、但技能自身没有书面流程的问题。

**该怎么提示它？**

带着下一步要构建的东西去提示。如果有大型构建在前，把规格说明指给它，问“怎样才能让这次变更更容易？”不带方向直接跑，它会自己扫描热点，日常维护够用，但指明方向才能让报告可行动。

**在大型存量代码库上好用吗？**

部分好用。它擅长处理缺少一致结构的大型现存代码库，也是在一次性结构治理之后推荐的维护机制。诚实的另一面：项目真正失控的用户报告它“有点帮助，但还是不够”，一位有八年存量代码库的开发者报告模型原地打转，而同样的技能在整洁仓库上能产出干净的图。那种情形目前还没有专用的 `/refactor` 技能。如果代码库完全没有共享词汇，先用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 建立词汇，往往能让这个技能的输出好得多。

**它和 `/codebase-design` 有什么区别？**

`/codebase-design` 是参考资料，不是会话驱动器。它提供词汇（模块、接口、深度、接缝、适配器、杠杆、内聚），本技能借用它。把全新智能体指向 `/codebase-design` 当作要“做”的事，是已知失败模式：没有自己的流程可跟，智能体会自创流程，重新探索代码，跑很久才问你一句。用本技能来驱动，拿那个当资料消费。

**它会告诉我代码库没问题吗？**

很少，事先要有这个心理准备。这个技能就是为了产出发现而构建的，所以框架导向会推着它产出候选项，而不是得出没问题的结论。强度徽标就是防线：一份全是 `Speculative` 的报告，就是它用自己唯一会的方式告诉你没发现问题。

**在 Codex 或其他执行环境里好用吗？**

部分好用。探索步骤直接点名了 Claude Code 的 `Agent` 工具（`subagent_type=Explore`），所以没有这个工具的[执行环境 (harness)](https://www.aihero.dev/ai-coding-dictionary/harness)可能跳过并行探索，而不是用自己的工具替代。技能照常运行，只是扫描没那么彻底。已有提议做一次执行环境中立的重写，但还没合并。

**在 TypeScript 里具体怎么做深模块？**

目前没有随技能提供的好答案。反复出现的需求是一份 `TYPESCRIPT.md`，给出对应这些原则的具体文件和模块布局，但它不存在。技能会告诉你加深属于哪里、接缝后面该放什么；把它翻译成包或目录结构目前靠你自己。

## 生效的标志

- 候选项用你领域的概念命名，而不是编造的类名：“Order 接入模块”，而不是“FooBarHandler”。
- 候选项聚集在你最近改过的文件里，而不是仓库里沉睡的角落。
- 运行期间没有代码变化。唯一的新文件是临时目录里的 HTML 报告。
- 它在报告后停下来，问你想选哪条，而不是自己继续往下走。
- 每张卡片用内聚或杠杆解释回报，并说明哪些测试会变简单，而不只是说“这样更干净”。
- 因可持续的理由否决一条候选项时，它会提议记一条 ADR，这样下次运行不再重复推荐。

## 它在流程中的位置

`improve-codebase-architecture` 是**定期维护**：每隔几天运行一次，在任何链条之外，负责排工作而不是做工作。它的邻居是 [codebase-design](https://aihero.dev/skills-codebase-design)（拥有每张卡片所用的深度与接缝词汇）、[grilling](https://aihero.dev/skills-grilling)（选中候选项后走决策树）和 [domain-modeling](https://aihero.dev/skills-domain-modeling)（在决策落定时维护 `CONTEXT.md` 和 ADR）。它的产出是一个想法，从 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 或 [to-spec](https://aihero.dev/skills-to-spec) 重新进入主构建流程。判断哪个技能适合哪种情形，[ask-matt](https://aihero.dev/skills-ask-matt) 是覆盖全套的路由器。
