## 功能简介

`tdd` 以测试先行的方式构建功能或修复缺陷：先写一个失败的测试，再写刚好能通过的最少量代码，然后进入下一个行为。它承载了让这个循环产出值得保留的测试的标准：什么是好的测试、测试放在哪里、Mock 有什么用，以及会悄悄毁掉整个测试套件的三种反模式。

在你没有先确认接缝之前，它不会在任何接缝处写测试。在任何测试存在之前，它会先说出打算测试的公开边界，请你确认，因为测试精力是有限的，要花在关键路径上，而不是每个边界情况上。另一件要知道的事是，`tdd` 是**参考规范**，不是驱动器。它持有循环规则，而另一些东西（你，或者 [implement](https://aihero.dev/skills-implement)）运行应用这些规则的[会话 (session)](https://www.aihero.dev/ai-coding-dictionary/session)。

## 何时使用

输入 `/tdd` 即可调用；当任务符合条件时，[智能体 (agent)](https://www.aihero.dev/ai-coding-dictionary/agent)会自动选用它：测试先行地构建功能或修复缺陷，或者当你说出“red-green-refactor”（红绿重构）时。

当存在要构建的具体行为，有输入和可观察输出，并且你希望测试在重构后仍然有效时，就该用它。

| 你的处境 | 该去哪里 |
| --- | --- |
| 有输入输出定义明确的行为（业务逻辑、请求/响应契约、转换、校验） | `tdd` |
| 行为还没有定下来 | [to-spec](https://aihero.dev/skills-to-spec)，它同样会在写代码之前确认测试接缝 |
| 问题其实是接口形状，而不是测试 | [codebase-design](https://aihero.dev/skills-codebase-design) |
| 你已有[规约 (spec)](https://www.aihero.dev/ai-coding-dictionary/spec)或[工单 (ticket)](https://www.aihero.dev/ai-coding-dictionary/ticket)，想让它替你跑完整个构建 | [implement](https://aihero.dev/skills-implement)，它会按工单驱动 `tdd` |
| 配置、装配、胶水代码、类型注解、直透的 CRUD 委托 | 这里都不太合适；见下面的开放缺口 |

最后一行是真实的缺口，不是风格偏好。本技能决定接缝 (seam) *放在哪里*；它不决定某个改动*是否值得*走这个循环。对没有独立事实来源可断言的改动运行它，你会得到复述实现的测试：正是本技能自己警告过的同义反复 (tautological) 反模式，只是从另一个方向走到了同样结果。它是[ issue #746](https://github.com/mattpocock/skills/issues/746)，仍未关闭。在它关闭之前，这个判断属于你或你的 `CLAUDE.md`。

## 前置条件

需要安装 [codebase-design](https://aihero.dev/skills-codebase-design)。`tdd` 曾经自带深模块和接口设计说明；在 v1.0 中它们被删除，改为使用共享技能，`tdd` 现在借用它的接口设计词汇。没有其他要求；本技能是[无状态 (stateless)](https://www.aihero.dev/ai-coding-dictionary/stateless) 的，不写自己的文件。

## 循环，以及它运行所在的接缝

三个词承载本技能。

**红灯绿灯 (Red-green)。**先写失败的测试，再写刚好能通过的最少量代码。不要预判下一个测试。没有重构阶段：它在 2026 年 6 月被删除，因为智能体实质上从不执行它，也因为评审和实现放在不同会话中效果更好。重构属于 [code-review](https://aihero.dev/skills-code-review)。

**垂直切片 (Vertical slice)。**一个接缝、一个测试、一份最小实现，然后重复，第一个周期是打通一条端到端路径的**示踪弹 (tracer bullet)**。反面是水平切片：先写全部测试，再写全部代码。批量测试验证的是*想象中的*行为，检查的是形状而不是用户行为，并且在理解实现之前就把测试结构定死了。

**预先确认的接缝 (Pre-agreed seam)。**接缝是你观察行为而不深入内部的公开边界。规则是绝对的：未经确认的接缝上不写测试。在完整链条中，接缝在更早的 [to-spec](https://aihero.dev/skills-to-spec) 阶段确认：“`/tdd` 只在预先确认的测试接缝上工作，`/code-review` 检查是否只用了确认过的测试接缝。”单独调用时，`tdd` 会直接问你。

它要预防的三种反模式：

| 反模式 | 特征 |
| --- | --- |
| 实现耦合 (Implementation-coupled) | 重命名内部函数就会弄坏测试，尽管行为没有变化。Mock 了内部协作者、断言调用次数、用数据库查询代替接口来验证。 |
| 同义反复 (Tautological) | 期望值按代码的计算方式算出来，所以测试按构造一定通过。期望值必须来自别处：已知正确的字面量、手工验算的例子、规约。 |
| 水平切片 (Horizontal slicing) | 在任何实现落地之前先落了一批测试。 |

Mock 只用于系统边界：外部 API、时间、随机性，有时是文件系统或数据库。不要用在自己的模块上。

## 常见问题

**为什么不重构？描述里还写着“red-green-refactor”。**

因为重构步骤已删除，而描述没有改。删除是有意的：智能体实质上从不做它，而且把实现和评审放在不同会话中效果更好。结果是否还算书本意义上的 TDD，不如循环能否产出更好的代码重要。触发语和正文的不一致已记录为 [issue #589](https://github.com/mattpocock/skills/issues/589)，仍未关闭，所以“red-green-refactor”继续作为触发本技能的短语有效。你得到的是红灯到绿灯 (red → green)，重构在 [code-review](https://aihero.dev/skills-code-review) 中。

**它让我选测试接缝，可我完全不知道怎么选。**

这是本技能被反馈最多的摩擦点（[issue #607](https://github.com/mattpocock/skills/issues/607)）。提示只按名称列出候选接缝，没有说明每个接缝能抓住什么、漏掉什么，所以你是在名称之间做选择。目前还没有发布修复。实用的变通办法是先让智能体讲清取舍再回答：组件级接缝会漏掉什么集成接缝能抓住的东西，速度又慢多少。这也是链条把接缝提前到 `to-spec` 中确认的原因，在那里你看到的是整个功能，而不是单个提示。

**明明技能要求红灯先行，它却先写了实现。**

确实会发生。有用户追问[模型 (model)](https://www.aihero.dev/ai-coding-dictionary/model)，得到了一个坦诚得不同寻常的回答：“我知道技能说‘一次一个测试，先看到它以正确的原因失败’。我读了，我只是按平常习惯写了。”本技能正是按接受这种情况来写的。任何指令都不能让智能体 100% 遵守，而把约束收得更紧只会限制智能体的创造力，收益却很小；即使没有严格遵守，跑这个循环的结果整体上仍然更好。如果某个切片必须严格遵守，那就盯着运行过程看，而不要指望技能去强制执行。

**应该先写浏览器测试或端到端测试吗？**

通常不该，但技能不会阻止。有用户报告智能体先写了 Playwright 测试，然后陷入漫长的重跑循环，最后得出结论说功能还没实现所以是*测试*坏了。请在你的 `CLAUDE.md` 中配置这一点。浏览器测试慢到红绿反馈循环不再划算；在仓库的 `CLAUDE.md` 中声明它们要在行为跑通之后再写。

**`/tdd` 能代替 `/implement` 或课程中的 `/do-work` 吗？**

不能。`/tdd` 记录方法论；`/implement` 是非常简单的工作到反馈到提交 (work→feedback→commit) 循环，是 `/do-work` 的直接对应物。课程中单个 `/do-work` 步骤现在拆到了 `/implement`、`tdd` 和 `/code-review` 上。如果你问针对工单该运行哪一个，答案几乎总是 `/implement`。

**深模块和接口设计指引去哪了？**

在 v1.0 进入了 [codebase-design](https://aihero.dev/skills-codebase-design)，泛化为多个技能共享的一套词汇。`refactoring.md` 同时搬走；重构现在是 [code-review](https://aihero.dev/skills-code-review) 的工作，那个技能带有 Fowler 坏味道基线。

**它知道我的其他工单吗？**

不知道。针对单个工单运行时，它会愉快地提出属于兄弟工单的工作，因为它看不到 issue 图谱的其余部分（[issue #129](https://github.com/mattpocock/skills/issues/129)）。Matt 的立场是这不是 `tdd` 的工作。把规约和工单一起传过去会有帮助；一开始就把工单切到合适大小，帮助更大。

## 生效标志

- 在任何测试文件出现之前，它停下来，说出打算测试的接缝，并等待确认。
- 先出现一个测试，变红，再写刚好能通过的代码，然后才出现下一个测试，而不是先批量写测试再批量写代码。
- 测试名称读起来是能力（“用户可以用有效购物车结算”），而不是内部细节（“checkout 调用了 paymentService.process”）。
- 断言中的期望值是可追溯到规约的字面量，而不是按代码的计算方式重新算出来的值。
- 重命名内部函数不会弄坏测试套件中的任何用例。
- Mock 只出现在外部边界（支付 API、时钟），从不包住自己的模块。

## 在流程中的位置

`tdd` 是主链条中构建步骤里的引擎，而不是独立的一步：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

[to-spec](https://aihero.dev/skills-to-spec) 在前面确认测试接缝，[implement](https://aihero.dev/skills-implement) 按工单驱动 `tdd`，[code-review](https://aihero.dev/skills-code-review) 在事后检查是否只用了确认过的接缝，并接管 `tdd` 不再做的重构。另一个邻居是 [codebase-design](https://aihero.dev/skills-codebase-design)，它是 `tdd` 所用接缝和深模块词汇的共享来源。你也可以单独使用它，只要有要构建的具体行为，即使没有完整规约也行。当不确定哪种情况适合哪个技能时，由 [ask-matt](https://aihero.dev/skills-ask-matt) 指路。
