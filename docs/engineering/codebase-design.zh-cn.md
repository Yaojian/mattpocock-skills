## 功能简介

`codebase-design` 用来校准你设计模块时的用词：**模块 (module)**、**接口 (interface)**、**深度 (depth)**、**接缝 (seam)**、**适配器 (adapter)**、**杠杆 (leverage)**、**内聚 (Locality)**。它给每个词下精确定义，禁用含糊的替代词（“component”、“service”、“API”、“boundary”），并给出由此推出的几条原则。

它是参考资料，不是流程。没有要跑的循环，不产出工件，也没有向你提问的检查点。其他凡是碰设计的技能都借用它的词汇；单独使用时，它只给语言，然后停下。在调用之前要先明白这一点，因为一个没有流程、没有停机规则的技能，如果你把一个[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)指向它说一句“开干”，它会自己现编一个流程出来。下面问题里有实际例子。

## 何时使用

输入 `/codebase-design`，或者在设计任务匹配时由智能体自动调用。

当你已经知道要重设计哪段代码、需要思考它的形状时用它：接缝放哪里，接口能压多小，这次抽取是否划算。当争论某个词到底什么意思时，也用它来定分止争。

有几个技能离它很近。用哪个取决于真正的问题是什么：

| 问题 | 对应的技能 |
|---|---|
| 单个模块的形状：接口、接缝、深度 | `codebase-design` |
| *领域措辞*问题：“account”有三个意思，两个人说的“cancellation”不是一回事 | [domain-modeling](https://aihero.dev/skills-domain-modeling) |
| 还不知道*该重设计哪个*模块 | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture)（负责找出候选的普查） |
| 想要有人跟你的设计辩一辩，而不只是命名 | [grilling](https://aihero.dev/skills-grilling) |
| 有一个要构建的具体行为，想要经得起重构的测试 | [tdd](https://aihero.dev/skills-tdd) |

## 词汇表

词汇表就是这个技能。每个术语都对照其他术语定义，每个都附带它要替换掉的词。

| 术语 | 含义 | 别说 |
|---|---|---|
| **模块 (Module)** | 任何有接口和实现的东西。刻意与规模无关：函数、类、包、跨层的切片都可以是模块。 | unit、component、service |
| **接口 (Interface)** | 调用者为正确使用它必须知道的一切：类型签名，外加不变式、顺序约束、错误模式、必需配置、性能特征。 | API、signature |
| **深度 (Depth)** | 接口处的杠杆：调用者或测试每学会一单位接口，能驱使多少行为。**深 (Deep)**：小接口背后有大量行为。**浅 (Shallow)**：接口几乎和实现一样复杂。 | 无 |
| **接缝 (Seam)** | Michael Feathers 的术语：可以在不改动该处的情况下改变行为的位置。它是接口的*位置*，放哪里是独立的决策，和背后放什么分开。 | boundary |
| **适配器 (Adapter)** | 在接缝处满足接口的具体东西。命名的是角色，不是材料：内存 fake 和 Postgres 仓库都是适配器。 | 无 |
| **杠杆 (Leverage)** | 调用者从深度中得到的东西：每学会一单位接口，能换来更多能力。 | 无 |
| **内聚 (Locality)** | 维护者从深度中得到的东西：改动、缺陷和验证都集中在一处。修一次，到处好。 | 无 |

深度刻意*不*定义为实现行数除以接口行数，也就是 Ousterhout 原书的定义。那个指标奖励的是把实现注水。采用的是深度即杠杆。

## 四条原则

- **深度是接口的属性，不是实现的属性。** 深模块内部完全可以用小而可替换的零件搭建，只要不对调用者暴露。模块可以有只供自己测试用的内部接缝，对外在接口处只有一个接缝。
- **删除测试。** 想象删掉这个模块。如果复杂度消失了，它就是个透传。如果复杂度在 N 个调用方那里冒出来，它就是划算的。
- **接口就是测试面。** 调用者和测试走的是同一条接缝。如果你想绕过接口去测*后面*的东西，说明模块形状错了。
- **一个适配器意味着假设的接缝，两个适配器意味着真实的接缝。** 在真有东西跨接缝变化之前，不要切接缝。只有一个适配器的接缝只是间接层。

另有两个配套文件走得更远，技能按需读取而不是预先加载。[DEEPENING.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DEEPENING.md) 把候选项的依赖分成四类（进程内、可本地替换、远端但自有、真正的外部），因为类别决定了加深后的模块跨接缝怎么测。[DESIGN-IT-TWICE.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DESIGN-IT-TWICE.md) 起并行的[子智能体 (subagent)](https://www.aihero.dev/ai-coding-dictionary/subagent)，为同一模块产出三种以上截然不同的接口，再按深度、内聚和接缝位置比较。

## 常见问题

**用 TypeScript 到底怎么做深模块？**

这是关于本技能被问最多的问题，而技能本身不回答。它定义深模块*是什么*；至于怎么拦住一次越过接口的随意 import，它一个字没说。[议题 #458](https://github.com/mattpocock/skills/issues/458) 问得很直白：“就算我们对接口满意了，它藏住了细节，等等。但怎么强制执行？没有 lint 或清晰护栏的话，人和 LLM 都会慢慢把它搞乱。” Matt 在帖子里的回答是三个选项：包进类或 IIFE，接受类会变得巨大；做成 monorepo 里的包，接受 monorepo 工具链；或者用 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 这类 linter 禁止绕过接口的导入。他在别处说过 Effect 是最好的机制，dependency-cruiser 是第二好。仓库 `in-progress/` 里有个 `setup-ts-deep-modules` 技能，按 `src/packages/<name>/index.ts` 约定搭架子，但它是 beta 通道技能，没有文档页，也没自带 lint 规则。

**我把一个会话指向它，它烧了 10 万 [词元 (token)](https://www.aihero.dev/ai-coding-dictionary/token)，重设计了我根本没问的东西。**

已知问题，记录在[议题 #449](https://github.com/mattpocock/skills/issues/449)。这个技能是模型可调用的，自述是词汇表，但里面没有任何硬性刹车能阻止智能体把它当成可执行的流程。被告知“在 /codebase-design 继续，把悬而未决的决定推进下去”，智能体就会去找它能找到的最像行动的内容：`DESIGN-IT-TWICE.md` 里的并行子智能体。它重新探索了上一个会话已经摸过的代码，跑了很远才想起来问一句。驱动型技能该有的护栏（检查点、一次一问、不自动推进）这里全没有，因为参考资料本来就没有。变通办法是点名一个驱动型技能，让这个技能待在底下做词汇：`/grill-with-docs`、`/improve-codebase-architecture` 或 `/tdd`，`codebase-design` 做词汇。议题还开着。

**`design-an-interface` 去哪了？还有 `/interface-design` 技能吗？**

`design-an-interface` 被移除并合并进了本技能。没有丢失：它的“设计两次”手法（并行子智能体产出截然不同的设计，出自 Ousterhout）以 `DESIGN-IT-TWICE.md` 的形式留在这里。另外，好几个人要过专门讲深模块加薄接口哲学的 `/interface-design` 技能；这套哲学已经在这里了，不打算另起技能。如果你是冲着这两个名字来的，看这页就对了。

**这不就是文件结构约定吗，比如目录、barrel 文件、按功能切片？**

不是，技能在反复被追问下一直守住这条线。[议题 #95](https://github.com/mattpocock/skills/issues/95) 提议把正式的分形树文件结构作为深模块的具体实现；回复是两者正交：“深模块关乎接口设计和经由严格接口访问，跟文件系统长什么样无关。完全可能按那种结构搭出一堆浅模块。” #458 里也出现过同样讨论：“我觉得你把模块概念和文件系统绑得太紧了。文件系统可以是模块形状的有用提示，但构造深模块没必要用文件系统。”词汇表刻意把**模块 (module)** 定义为与规模无关，就是这个原因。

**`tdd` 真的用这套词汇吗？**

现在用了。很长一段时间没用。原来住在 `tdd` 里的深模块内联注记在 v1.0 被删掉，改成引用这个共享技能，但替换上去的指针一直没加，所以 `tdd` 自己定义“seam”，什么也不引用。这个缺口已经补上：当接口形状是悬而未决的问题时，技能里已有指针。`tdd` 仍然拥有“seam”作为你要*测试*的那条边界；本技能拥有的是边界背后的模块形状。

**设计两次模式在 Claude Code 之外能用吗？**

不顺。`DESIGN-IT-TWICE.md` 写的是“用 Agent 工具并行起 3 个以上的子智能体”，这是 Claude Code 的[工具 (tool)](https://www.aihero.dev/ai-coding-dictionary/tool)，按 Claude Code 的名字写的。仓库给其他[宿主框架 (harness)](https://www.aihero.dev/ai-coding-dictionary/harness)（包括 Codex）配了元数据，那些框架底下可能根本没有这个名字的东西，所以并行设计阶段的可移植性没有元数据暗示的那么好。记录在[议题 #564](https://github.com/mattpocock/skills/issues/564)，还开着。

**我能往词汇表里加自己的概念吗，比如 connascence、模块秘密 (module secrets)、[渐进式披露 (progressive disclosure)](https://www.aihero.dev/ai-coding-dictionary/progressive-disclosure)？**

确实有人提过。 [议题 #180](https://github.com/mattpocock/skills/issues/180) 提议用 Parnas 的模块秘密和 Page-Jones 的 connascence 作为命名层，讲清楚*什么*从接缝漏过去了，还附了可用的 diff；[议题 #303](https://github.com/mattpocock/skills/issues/303) 提议在实现内部做渐进式披露，让对外接口很深的模块底下不是一整块未分层的整体。两个都开着，没合并。已发布的词汇表刻意保持小，技能里也写了为什么保持小：一致的语言就是意义所在，没人一致使用的词不如没有。

## 怎样算生效

- 设计讨论里不再冒出“component”、“service”和“boundary”，开始冒出“module”、“interface”和“seam”。
- 有人能指着一个拟议的抽取说清它过不过删除测试，不含糊。
- 拟议的接缝附带第二个具名的适配器，而不只有第一个。
- 对接口的讨论覆盖不变式、顺序和错误模式，而不只有类型签名。
- 调用它不会开启一个会话。如果智能体只凭 `/codebase-design` 就开始读文件、提重构，说明它把参考资料当成了驱动器。

## 在整体中的位置

`codebase-design` 是**随时取用的独立技能**，是工程技能底下的词汇层，而不是任何链条中的一步。它最近的邻居是 [domain-modeling](https://aihero.dev/skills-domain-modeling)，后者是*问题域*措辞的平行参考，而不是模块形状。通常两个一起要，因为给深模块起好名字两边都要。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是另一个：它在代码库里普查加深候选，每一个都用这套词汇写出来，所以它负责找到模块，本技能是设计模块的工作台。拿不准哪个技能或流程合适时，用 [ask-matt](https://aihero.dev/skills-ask-matt) 路由。
