数字枚举

1. 支持反向映射
2. 会生成运行时代码

```ts
enum Status {
  Pending,
  Success,
  Failed,
}

```



字符串枚举

- 可读性好
- 不会有 `0 / 1 / 2` 这种魔法值
- 和 **接口 / 后端字段 / 链上状态** 非常契合

```ts
enum OrderStatus {
  Pending = "PENDING",
  Paid = "PAID",
  Canceled = "CANCELED",
}

```

常量枚举（性能）

不能被 `Object.values` 遍历

不适合 SDK / 库对外导出（可能被 tsconfig 影响）

```ts
const enum ChainType {
  EVM = "EVM",
  Solana = "SOLANA",
}

```

枚举反向映射（只有数字枚举有）

```ts
enum Role {
  Admin,
  User,
}

```

## enum vs union type（⚠️ 很重要）

### union type（更现代）

```ts
type WalletType = "MetaMask" | "OKX" | "Trust";
```

### enum

```ts
enum WalletTypeEnum {
  MetaMask = "MetaMask",
  OKX = "OKX",
  Trust = "Trust",
}
```

### 大厂 & React 项目推荐

| 场景                 | 推荐         |
| -------------------- | ------------ |
| 仅做类型约束         | ✅ union type |
| 需要运行时对象       | ✅ enum       |
| 表单 / 映射 / switch | ✅ enum       |
| SDK / 配置常量       | ✅ const enum |

```ts
enum TxStatus {
  Pending = "pending",
  Success = "success",
  Failed = "failed",
}

const TxStatusText: Record<TxStatus, string> = {
  [TxStatus.Pending]: "处理中",
  [TxStatus.Success]: "成功",
  [TxStatus.Failed]: "失败",
};

```

**状态 / 后端字段 / 显示映射** → `string enum`

**参数校验 / props** → `union type`

**高频常量 / SDK 内部** → `const enum`

**不要滥用数字 enum**