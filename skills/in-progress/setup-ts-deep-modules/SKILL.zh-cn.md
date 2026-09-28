---
name: setup-ts-deep-modules
description: "将 dependency-cruiser 接入 TypeScript 仓库，让每个包成为深度模块，实现藏在子目录中，只能经由入口文件访问。用户调用。"
disable-model-invocation: true
---

# 配置 TS 深度模块

让仓库中的每个包都成为**深度模块**：小接口后面藏大量行为。包的公开面就是它的**入口文件**（包根目录下的文件），子目录里的一切都对外隐藏。本技能安装 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 并配置规则，让入口文件成为唯一的进入方式，然后证明规则真的会咬人。

涉及词汇（deep module、interface、seam、depth）时，用 "codebase-design" 调用 Skill 工具，并全程使用它的语言。

## 约束出的形状

```
src/packages/
  <name>/
    index.ts        ← an entry point (public). Import this from outside.
    client.ts       ← another entry point. Packages may expose SEVERAL.
    lib/            ← implementation: hidden from outside, free to import each other.
    tests/          ← co-located tests + fixtures (a subfolder, so private).
```

公开面是包的**根目录文件**，而不是指定的某个 `index.ts`。按惯例实现放在 `lib/`、测试放在 `tests/`，让每个包都有相同的两目录形状。但规则本身是通用的：任何子目录里的任何东西都是私有的，所以加目录时永远不用改配置。

四条规则，级别全是 `error`：

1. **入口边界**：包之外的代码（应用代码或其他包）只能导入该包的入口文件（根目录文件），绝不能导入其子目录里的东西。
2. **包内自由**：包自己的文件之间可自由互导。
3. **测试走入口**：`<pkg>/tests/` 下的文件只能导入各包的入口文件和自身 `tests/` 下的 fixture，绝不能导入任何包的子目录内部（包括自己家的）。跨包集成测试可以，深层导入不行。
4. **无循环**：不允许依赖循环。

**要入口文件，不要桶文件。**因为公开面是每个根文件，包可以暴露多个小入口（`index.ts`、`client.ts`、`server.ts`），而不必把一切都汇进一个巨大的 `index.ts`。不鼓励复导出整棵子树的桶文件，保持入口小，实现藏进子目录。

分层（哪些包可以依赖哪些包）是另一回事，在配置里只留注释桩子，由本仓库自行补全。

## 步骤

### 1. 探测环境

- **包管理器**：`pnpm-lock.yaml` 对应 pnpm，`yarn.lock` 对应 yarn，`bun.lockb` 对应 bun，否则用 npm。下面所有命令都用它（`pnpm` / `yarn` / `npm run` / `bunx`）。
- **包根目录**：有 `src/` 就用 `src/packages`，否则用 `packages`。如果仓库已有明显不同的惯例，和用户确认后再定。
- **已有配置**：检查 `.dependency-cruiser.*` 文件。如果已存在，不要覆盖，把四条规则和选项合并进去，并告诉用户你加了什么。

**完成标准：**包管理器、包根目录和已有配置状态都已明确。

### 2. 安装 dependency-cruiser

用探测到的包管理器把 `dependency-cruiser` 装成 devDependency。

**完成标准：**`dependency-cruiser` 已进入 `devDependencies`。

### 3. 写配置

把 [`dependency-cruiser.config.cjs`](./dependency-cruiser.config.cjs) 复制到仓库根目录，命名为 `.dependency-cruiser.cjs`。把 `PACKAGES_ROOT` 设为步骤 1 探测到的根目录。规则基于路径深度、与扩展名无关，其他都不用改。

**完成标准：**`.dependency-cruiser.cjs` 已存在，`PACKAGES_ROOT` 正确，四条禁止规则齐全。

### 4. 接入检查

- 新增 `lint:boundaries` 脚本：`depcruise <packages-root>`（或 `depcruise src`）。
- 把它并入仓库已有的总检查命令，也就是跑类型检查的那个命令（比如 `check` / `ci` / `validate` 脚本）。不要动 `tsconfig`，也不要加路径别名。
- 如果没有总检查脚本，就只加 `lint:boundaries`，并告诉用户把它加入 CI。

**完成标准：**`lint:boundaries` 已存在，且和类型检查在同一条命令里运行。

### 5. 搭建示例包

新建已提交的 `<packages-root>/example/`，作为可复制的模板：

- `index.ts` 是入口文件，导出一个委托给内部文件的函数（让包 visibly 呈现出深度，而不是透传）。
- `lib/impl.ts`：子目录中的内部文件，被 `index.ts` 导入，外部不可达。
- `tests/example.test.ts` **只**导入 `../index`（入口文件），并针对公开函数断言。

告诉用户这是 starter 模板，可复制可删除。

**完成标准：**示例包已存在，经由根入口文件对外暴露行为，`impl` 藏在子目录中。

### 6. 证明规则会咬人

这是整个技能的完成标准：遇到违规不报错的配置毫无价值。

1. 运行 `lint:boundaries`，在干净的示例上必须**通过**。
2. 给 `tests/example.test.ts` 临时加一个深层导入（比如 `import { thing } from "../lib/impl"`），再跑 `lint:boundaries`，必须**失败**，报错 `tests-through-entrypoints`。
3. 撤销深层导入，再跑一次，必须**通过**。

**完成标准：**依次观察到通过、深层导入下失败、撤销后再次通过。如果第 2 步没有失败，说明规则没接对，先修好再收工。

### 7. 记录惯例

在包目录内写 `README.md`（`<packages-root>/README.md`，和它管辖的包放在一起），内容包括：`src/packages/<name>/` 布局（入口文件在根目录，`lib/` 放实现，`tests/` 放测试）、“只能经由包的入口文件（根文件）导入”，以及如何跑 `lint:boundaries`。明确**不鼓励桶文件**：暴露多个小入口，而不是经由一个 index 复导出整棵子树。篇幅控制在可复制片段加四条规则各一段。

然后从仓库的智能体说明文件（有 `CLAUDE.md` 就用它，否则用 `AGENTS.md`，两者都没有就新建 `AGENTS.md`）加一条**上下文指针**指向它。一行就够，比如 `Packages are deep modules: see [src/packages/README.md](./src/packages/README.md) before adding or importing one.` 这条指针让智能体主动发现边界规则，而不是撞上去才知道。

**完成标准：**`<packages-root>/README.md` 已存在且不鼓励桶文件，仓库的 `CLAUDE.md` / `AGENTS.md` 已链接到它。

## 说明

- 配置里的 `$1` 反向引用（dependency-cruiser 的分组匹配）让包能访问自己的内部实现，而外部不行。不要把它展平成按包拆分的规则。
- 公开还是私有由**深度**决定：包根文件是入口，子目录里的一切都是私有的。惯例子目录是 `lib/`（实现）和 `tests/`，但规则不写死它们：任何子目录都是私有的，所以新目录永远不用改配置。新增入口就是新增根文件（不用桶文件）。
- 包是**扁平**的：根目录下一层直接子目录。包内部可以任意嵌套，包里不能再套包。
- 用 `.cjs`（不用 `.js`），这样即使在 `"type": "module"` 的仓库里，配置的 `module.exports` 也能工作。
