## 什么是 Narrowing？

> **从“宽泛类型” → “更具体类型”**

```ts
let value: string | number;
```

通过判断后，TS 能知道 **此时 value 是什么类型**

------

### 1 `typeof` Narrowing（最基础）

```ts
function format(v: string | number) {
  if (typeof v === "string") {
    v.toUpperCase(); // string
  } else {
    v.toFixed(2); // number
  }	
}
```

### 适用

- `string | number | boolean | symbol | bigint`

------

### 2 `instanceof` Narrowing（类 & 内置对象）

```ts
function handle(e: Error | DOMException) {
  if (e instanceof Error) {
    e.message;
  } else {
    e.code;
  }
}
```

### 常见

- `Date`
- `Error`
- 自定义 class

------

### 3 `in` 操作符（对象联合）

```ts
type EvmTx = { hash: string; chainId: number };
type SolTx = { signature: string };

function handle(tx: EvmTx | SolTx) {
  if ("hash" in tx) {
    tx.hash;
  } else {
    tx.signature;
  }
}
```

✔ 非常适合多链 DApp 数据结构

------

### 4 判等 Narrowing（Literal）

```ts
type Status = "pending" | "success" | "failed";

function render(s: Status) {
  if (s === "success") {
    // s: "success"
  }
}
```

也适用于：

```ts
if (value === null)
if (value === undefined)
```

------

### 5 Discriminated Union（⭐ 最重要）

### 5.1 定义

```ts
type Tx =
  | { type: "evm"; hash: string }
  | { type: "sol"; signature: string };
```

### 5.2 使用

```ts
function handle(tx: Tx) {
  switch (tx.type) {
    case "evm":
      tx.hash;
      break;
    case "sol":
      tx.signature;
      break;
  }
}
```

✔ 大厂最常用
 ✔ switch 自动补全
 ✔ 不会写错字段

------

### 6 自定义 Type Guard（高级）

```ts
function isEvmTx(tx: any): tx is { hash: string } {
  return typeof tx?.hash === "string";
}
if (isEvmTx(tx)) {
  tx.hash; // 安全
}
```

### React / Hook 里很常见

------

### 7 Truthy / Falsy Narrowing（⚠️ 小心）

```ts
function f(v?: string) {
  if (v) {
    v.toUpperCase();
  }
}
```

⚠️ 问题：

```ts
"" // 会被误判
0
false
```

✅ 更安全：

```ts
if (v !== undefined) {}
```

------

### 8 Optional Chaining + Narrowing

```ts
user?.profile?.name;
```

但 ⚠️ **不会自动收窄**

```ts
if (user?.profile) {
  user.profile.name; // ❌ 仍可能为 undefined
}
```

✅ 正确写法：

```ts
if (user && user.profile) {
  user.profile.name;
}
```

------

### 9 `never` 穷尽检查（大厂必备）

```ts
function assertNever(x: never): never {
  throw new Error("Unhandled case");
}
function handle(tx: Tx) {
  switch (tx.type) {
    case "evm":
      break;
    case "sol":
      break;
    default:
      assertNever(tx); // 新类型没处理会报错
  }
}
```

✔ 防止未来改类型忘记处理

### 10 Narrowing 在 React 中的实战

### Props 收窄

```ts
type Props =
  | { loading: true }
  | { loading: false; data: string[] };

function List(p: Props) {
  if (p.loading) {
    return <Spinner />;
  }

  return <ul>{p.data.map(...)}</ul>;
}
```

------

### 11 常见反模式（你可能写过）

### ❌ 强制断言

```
(tx as any).hash;
```

### ❌ 非空断言滥用

```
user!.profile!.name;
```

✅ 用 Narrowing 替代

------

## ✅ 总结一张速查表

| 技巧                | 场景         |
| ------------------- | ------------ |
| `typeof`            | 基础类型     |
| `instanceof`        | class        |
| `in`                | 对象 union   |
| literal 判断        | 状态         |
| discriminated union | **强烈推荐** |
| type guard          | 封装逻辑     |
| `never`             | 穷尽校验     |