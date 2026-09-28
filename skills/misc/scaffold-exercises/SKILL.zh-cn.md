---
name: scaffold-exercises
description: 创建带章节、习题、解答和讲解材料的练习目录结构，并确保通过 lint。当用户想要搭建练习脚手架、创建练习存根或新建课程章节时使用。
---

# 搭建练习脚手架

创建能通过 `pnpm ai-hero-cli internal lint` 的练习目录结构，然后用 `git commit` 提交。

## 目录命名

- **章节（Section）**：放在 `exercises/` 内的 `XX-section-name/`（例如 `01-retrieval-skill-building`）
- **练习（Exercise）**：放在章节内的 `XX.YY-exercise-name/`（例如 `01.03-retrieval-with-bm25`）
- 章节号为 `XX`，练习号为 `XX.YY`
- 名称使用短横线命名（小写加连字符）

## 练习变体

每个练习至少需要以下子目录之一：

- `problem/`：学生工作区，带 TODO
- `solution/`：参考实现
- `explainer/`：概念讲解材料，无 TODO

打存根时，除非计划另有说明，默认使用 `explainer/`。

## 必需文件

每个子目录（`problem/`、`solution/`、`explainer/`）都需要一个 `readme.md`，要求：

- **非空**（必须有实质内容，哪怕只有一行标题也可以）
- 没有坏链

打存根时，创建一个只有标题和描述的最小 readme：

```md
# Exercise Title

Description here
```

如果子目录中有代码，还需要一个 `main.ts`（多于 1 行）。但对存根而言，只有 readme 的练习也可以。

## 工作流

1. **解析计划**：提取章节名、练习名和变体类型
2. **创建目录**：对每条路径执行 `mkdir -p`
3. **创建存根 readme**：每个变体目录一个带标题的 `readme.md`
4. **运行 lint**：执行 `pnpm ai-hero-cli internal lint` 验证
5. **修复错误**：迭代直到 lint 通过

## Lint 规则摘要

该检查器（`pnpm ai-hero-cli internal lint`）检查：

- 每个练习都有子目录（`problem/`、`solution/`、`explainer/`）
- `problem/`、`explainer/` 或 `explainer.1/` 中至少存在一个
- 主子目录中存在非空 `readme.md`
- 没有 `.gitkeep` 文件
- 没有 `speaker-notes.md` 文件
- readme 中没有坏链
- readme 中没有 `pnpm run exercise` 命令
- 每个子目录都需要 `main.ts`，纯 readme 的除外

## 移动/重命名练习

重编号或移动练习时：

1. 使用 `git mv`（而不是 `mv`）重命名目录，保留 git 历史
2. 更新数字前缀以维持顺序
3. 移动后重新运行 lint

示例：

```bash
git mv exercises/01-retrieval/01.03-embeddings exercises/01-retrieval/01.04-embeddings
```

## 示例：按计划打存根

给定如下计划：

```
Section 05: Memory Skill Building
- 05.01 Introduction to Memory
- 05.02 Short-term Memory (explainer + problem + solution)
- 05.03 Long-term Memory
```

创建：

```bash
mkdir -p exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer
mkdir -p exercises/05-memory-skill-building/05.02-short-term-memory/{explainer,problem,solution}
mkdir -p exercises/05-memory-skill-building/05.03-long-term-memory/explainer
```

然后创建 readme 存根：

```
exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer/readme.md -> "# Introduction to Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/explainer/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/problem/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/solution/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.03-long-term-memory/explainer/readme.md -> "# Long-term Memory"
```
