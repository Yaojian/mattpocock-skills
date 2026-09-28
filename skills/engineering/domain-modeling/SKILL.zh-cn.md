---
name: domain-modeling
description: 构建并打磨项目的领域模型。当讨论代码库术语、编写或编辑 CONTEXT.md，或记录或编辑 ADR 时使用。
---

# 领域建模（Domain Modeling）

在设计过程中主动构建并打磨项目的领域模型。这是*主动*的纪律：质疑术语、发明边界场景、术语表和决策一旦成形立刻落笔。（仅仅*阅读* `CONTEXT.md` 借用词汇不算本技能：那是任何技能都能做的一行习惯。本技能用于改变模型，而不只是消费模型。）

## 文件结构

大多数仓库只有一个上下文：

```
/
├── CONTEXT.md
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

如果根目录存在 `CONTEXT-MAP.md`，说明仓库有多个上下文。地图指向每个上下文的位置：

```
/
├── CONTEXT-MAP.md
├── docs/
│   └── adr/                          ← system-wide decisions
├── src/
│   ├── ordering/
│   │   ├── CONTEXT.md
│   │   └── docs/adr/                 ← context-specific decisions
│   └── billing/
│       ├── CONTEXT.md
│       └── docs/adr/
```

懒创建文件：有东西可写时才创建。还没有 `CONTEXT.md` 时，在第一个术语敲定时创建；还没有 `docs/adr/` 时，在需要第一份 ADR 时创建。

## 会话期间

### 对照术语表质疑

当用户的用词与 `CONTEXT.md` 中的既有语言冲突时，立刻指出。“术语表把 cancellation 定义为 X，但你似乎在说 Y，到底是哪个？”

### 打磨模糊语言

当用户使用含糊或一词多义的说法时，给出精确的标准术语（canonical term）。“你说的 account 到底指 Customer 还是 User？这是两个不同的东西。”

### 讨论具体场景

讨论领域关系时，用具体场景做压力测试。发明能探测边界情况的场景，迫使用户把概念之间的边界说精确。

### 对照代码交叉验证

当用户描述某处如何工作时，检查代码是否同意。如果发现矛盾，摆出来：“你的代码会取消整个 Order，但你刚说支持部分取消，哪个是对的？”

### 行内更新 CONTEXT.md

术语一旦敲定，就地更新 `CONTEXT.md`。不要攒批：敲定一个记录一个。格式见 [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md)。

`CONTEXT.md` 不得包含任何实现细节。不要把它当规格说明、草稿纸或实现决策的收容所。它只是一份术语表，仅此而已。

### 克制地提议 ADR

只有三条全中时，才提议创建 ADR：

1. **难以逆转**：将来改主意的代价实实在在
2. **缺少上下文会令人意外**：未来的读者会问“当时为什么这样做？”
3. **真实权衡的结果**：确实存在备选方案，你因具体理由选中了这一个

缺任何一条都不写 ADR。格式见 [ADR-FORMAT.md](./ADR-FORMAT.md)。
