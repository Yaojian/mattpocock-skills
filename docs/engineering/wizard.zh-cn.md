## 它能做什么

`wizard` 会生成一个交互式 bash 脚本，一步一步带真人走完手动流程：接入第三方服务、跑一次性迁移、把项目从状态 A 搬到状态 B。它逐个打开 URL，告诉你点哪里、复制什么，收回返回结果，并写入 `.env` 文件和 GitHub Actions secrets。

[Agent](https://www.aihero.dev/ai-coding-dictionary/agent) 只负责写脚本，从不运行它。运行的人是你，在你自己的机器上。所以 wizard 不是一份照着做的说明清单；它是一个驱动流程并持有状态的程序，你的任务是点击、粘贴、按回车。

## 何时使用它

你可以直接输入 `/wizard`，Agent 也可以主动调用它。当它撞上必须由你出手的步骤（它签发不了的 key、它点不了的控制台），它会为你构建一个 wizard，而不是把说明写进聊天里任其滚走。

当挡住你的下一步是去控制台跑一趟时，就用它：

| 场景 | wizard 做什么 |
| --- | --- |
| 新人入职，应用启动前要配六个服务 | 按顺序打开每个控制台，收集 key，写入 `.env` 和 CI |
| 一次性迁移要按特定顺序拨开关 | 把不可逆步骤排好，并在确认门后执行 |
| 项目要一次性从状态 A 搬到状态 B | 带你走完迁移，并报告哪些没能完成 |
| 你正准备把这些步骤写进 README | 改为写一个可执行版本，它不会像文档那样悄悄腐烂 |

不要用它来*决定*构建什么；那是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 和 [to-spec](https://aihero.dev/skills-to-spec) 的活。

## 前提条件

生成 wizard 没有前提条件。它写出的向导跑在 bash 上，某个阶段要设置 GitHub secret 或变量时会用到 `gh`。如果 `gh` 缺失或未登录，该阶段变成警告，结束汇总会告诉你哪些要手工补，而不是让整个运行失败。

## 阶段（Stages）

**阶段（stage）**是在一个屏幕上完成的一项聚焦任务。脚本在阶段之间清屏，所以溢出屏幕的阶段会丢掉滚走的部分。你按依赖顺序编写阶段，并设置 `TOTAL_STAGES`，它驱动进度显示。

动手写第一行之前先定范围。[技能 skill](https://www.aihero.dev/ai-coding-dictionary/skill) 会先读仓库而不是上来就问：`.env*`、`docker-compose*`、框架配置、`.github/workflows/` 里每个 `secrets.*` / `vars.*` 引用：这些都是 wizard 必须产出的值。然后它把排好序的阶段列表给你确认，之后才把每个阶段映射到真人要走的精确路径（“控制台 → Developers → API keys → Reveal test key → 复制”）。凡是它不知道的当前界面，它会问你或查文档，而不是编造点击步骤。

每个收集到的值，在定范围时就要定好落点：

| 落点 | 何时 |
| --- | --- |
| 只写 `.env` | 本地开发需要，CI 不需要 |
| GitHub secret | CI 要读，且敏感 |
| GitHub variable | CI 要读，且公开 |
| `.env` 和 secret 都写 | 本地开发和 CI 都要 |
| 哪里都不写 | 该阶段是纯动作：拨了个开关、升了个套餐 |

## 模板已经解决了 UX

[模板 template](https://github.com/mattpocock/skills/blob/main/skills/engineering/wizard/template.sh)自带完整体验：带剩余时间的进度、确认门、含 WSL 在内的跨平台 URL 打开、secret 隐藏输入、幂等的 `.env` 更新、`gh secret` / `gh variable` 写入，以及最后汇总所有跳过项。`STAGES` 标记之上的部分是固定库，每个 wizard 都一样，永远不要手工改。一致性正是目的。你的工作只有定范围和写阶段。

写 wizard 的 Agent 从不端到端运行它，因为它要开浏览器并等待真人输入。它只做静态校验：`bash -n`、有条件时跑 `shellcheck`，并追踪每个值都落到了定范围时说好的位置，每个 `set_secret` 名字都能在 CI 里找到真实的 `secrets.*` 引用。请据此设定期望：第一次运行是你跑的，那次运行就是测试。

## 默认用完即弃

| 你的情况 | 脚本怎么处理 |
| --- | --- |
| 一次性迁移、个人配置、再也不会重复的搬迁 | 存到 scratch 或 `scripts/` 路径，跑完删掉 |
| 仓库里下一个人也需要的配置路径 | 提交进仓库并在 README 里链接，让后来人直接跑脚本，而不是再问一遍 Agent |

## 常见问题

**我的 API key 会进模型的上下文吗？**

不会。Agent 只写脚本，不运行它。脚本由你自己运行，它用隐藏的终端输入收集 key，直接写入 `.env` 或 `gh secret`。wizard 是 CLI，模型连不上它。一个提醒：这只适用于 wizard 在运行时收集的值。如果你在定范围时把 key 粘进了聊天，它就和任何粘贴文本一样进了[上下文 context](https://www.aihero.dev/ai-coding-dictionary/context)。

**输错的值能回去改吗？**

中途不行。没有后退键：阶段只往前跑，第 3 步答错了就 Ctrl-C 后重跑。重跑按便宜设计：已写入 `.env` 的值会作为默认值回填，你一路回车跳过做对的阶段，只重输错的那一个。这是上线第一周就提出的问题，至今未关闭：“太喜欢了！但有个问题，有办法回去改已输入的内容吗？”

还有个相关的未修复 bug。`ask` 提示符里按方向键会插入 `^[[D` / `^[[C` 而不是移动光标，因为提示符用的是 `read -r` 而不是 Readline（[issue #741](https://github.com/mattpocock/skills/issues/741)）。退格可用，方向键不可用。删到错处重输，别想把光标移进去。

**它知道我已经配好哪些了吗？**

部分知道，比上线时的第一反应以为的要少。它动手问之前先读仓库（你的 `.env` 文件、`docker-compose`、框架配置、CI 里的 `secrets.*` 引用），所以它只定位真正缺失的值，不会像 README 那样从零开始。它不做的是检查第三方服务。如果你 `.env` 里已有 key，wizard 会回填并让你回车保留；如果你当年建了 Stripe 账号但没存 key，wizard 还是会送你去控制台取。

**它在追问和规格之后处于什么位置？**

没有固定位置。它是独立件，不是链条上的一环。常见的猜测是 `/grill-with-docs → /to-spec → /wizard`，这个顺序没问题，但触发条件是出现了手动流程，它可以在任何时刻出现：开工前、构建中、上线很久后之后。它还能当发现工具：定范围会翻出任务隐藏的前置条件，比如你没想到的三个 API key，让你在投入之前先看见。

**Claude Code 之外能用吗？**

产物无条件能用：它是纯 bash 脚本，不关心哪套[运行环境 harness](https://www.aihero.dev/ai-coding-dictionary/harness)生成的它。技能本身是模型可调用的，所以处处可列出：在 Claude Code 里输入 `/wizard`，在 Codex 里输入 `$wizard`，或直接描述你卡住的配置。模型可调用也让它避开了 [#693](https://github.com/mattpocock/skills/issues/693)，在 Claude 桌面端和网页端，用户可调用的技能会从[模型 model](https://www.aihero.dev/ai-coding-dictionary/model)的列表里掉出来并显示为未安装。

**它以前不是用户可调用的吗？**

是的。它现在是模型可调用的，所以 Agent 在撞上必须由你出手的步骤时会主动拿起它。以前能做的事都没丢：模型可调用*增加*了 Agent 的主动使用，从不拿走你的使用权，`/wizard` 的行为和以前完全一样。变化的是退役了一种失败模式：Agent 在构建中途撞上凭证墙，只能往聊天里甩六个编号步骤让你手工照做。

**它以前在 `in-progress/` 里：现在去哪了？**

`engineering/`，v1.2 起。从 beta 分组毕业，现在随插件一起发布，不用单独安装就能拿到。毕业时行为没变。

## 达到这些就说明它正常工作

- 在任何脚本存在之前，你先看到排好序的阶段列表和每个阶段的产出值，并被要求确认。
- 每个 URL 都是先打开，再问你要那一页的值。你永远不会被要求粘贴一个还没被送去取的东西。
- Secret 盲输。敏感内容不会回显进你的滚动历史。
- 每个阶段占一屏。你还需要的内容不会滚走。
- Ctrl-C 后重跑能接着来，已保存的值作为默认值回填。
- 最后的屏幕列出写了什么，并单独列出没能做什么、需要你手工收尾。

## 它在整体中的位置

`wizard` 是随时可用的独立件，站在自动化结束、真人必须点鼠标的那条线上。最近的邻居是 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills)，因为两者都是为了让仓库进入可用状态：那个配置这套技能，`wizard` 为其他一切生成配置路径。它也和 [implement](https://aihero.dev/skills-implement) 搭档：构建落地了需要凭证或手工切换的功能，真人那一半就靠 wizard 完成。拿不准当下该用哪个技能时，[ask-matt](https://aihero.dev/skills-ask-matt) 会帮你分流。
