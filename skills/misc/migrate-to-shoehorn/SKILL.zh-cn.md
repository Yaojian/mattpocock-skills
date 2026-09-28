---
name: migrate-to-shoehorn
description: 将测试文件从 `as` 类型断言迁移到 @total-typescript/shoehorn。当用户提到 shoehorn、想替换测试中的 `as`，或需要构造部分测试数据时使用。
---

# 迁移到 Shoehorn

## 为什么用 shoehorn？

`shoehorn` 让你在测试中传入部分数据，同时让 TypeScript 满意。它用类型安全的替代方案取代 `as` 断言。

**仅限测试代码。** 绝不在生产代码中使用 shoehorn。

测试中 `as` 的问题：

- 一直被教导不要用它
- 必须手动指定目标类型
- 故意构造错误数据时要用双重 `as`（`as unknown as Type`）

## 安装

```bash
npm i @total-typescript/shoehorn
```

## 迁移模式

### 只需要少数属性的大对象

改动前：

```ts
type Request = {
  body: { id: string };
  headers: Record<string, string>;
  cookies: Record<string, string>;
  // ...20 more properties
};

it("gets user by id", () => {
  // Only care about body.id but must fake entire Request
  getUser({
    body: { id: "123" },
    headers: {},
    cookies: {},
    // ...fake all 20 properties
  });
});
```

改动后：

```ts
import { fromPartial } from "@total-typescript/shoehorn";

it("gets user by id", () => {
  getUser(
    fromPartial({
      body: { id: "123" },
    }),
  );
});
```

### `as Type` 改为 `fromPartial()`

改动前：

```ts
getUser({ body: { id: "123" } } as Request);
```

改动后：

```ts
import { fromPartial } from "@total-typescript/shoehorn";

getUser(fromPartial({ body: { id: "123" } }));
```

### `as unknown as Type` 改为 `fromAny()`

改动前：

```ts
getUser({ body: { id: 123 } } as unknown as Request); // wrong type on purpose
```

改动后：

```ts
import { fromAny } from "@total-typescript/shoehorn";

getUser(fromAny({ body: { id: 123 } }));
```

## 各函数用法

| 函数 | 适用场景 |
| --------------- | -------------------------------------------------- |
| `fromPartial()` | 传入仍能通过类型检查的部分数据 |
| `fromAny()` | 传入故意写错的数据（保留自动补全） |
| `fromExact()` | 强制要求完整对象（之后再换回 fromPartial） |

## 工作流

1. **收集需求**，询问用户：
    - 哪些测试文件中的 `as` 断言造成了困扰？
    - 是否在处理大对象，而其中只有部分属性重要？
    - 是否需要传入故意写错的数据来做错误测试？

2. **安装并迁移**：
    - [ ] 安装：`npm i @total-typescript/shoehorn`
    - [ ] 找出带 `as` 断言的测试文件：`grep -r " as [A-Z]" --include="*.test.ts" --include="*.spec.ts"`
    - [ ] 把 `as Type` 替换为 `fromPartial()`
    - [ ] 把 `as unknown as Type` 替换为 `fromAny()`
    - [ ] 从 `@total-typescript/shoehorn` 添加导入
    - [ ] 运行类型检查验证
