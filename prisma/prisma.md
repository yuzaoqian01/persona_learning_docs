# 一、Prisma 是什么 & 适合用在什么场景

## 1. Prisma 是什么

Prisma 是一个 **现代 ORM（对象关系映射）**，核心目标：

- 用 **Schema 文件** 描述数据库
- 自动生成 **强类型 Client**
- 用 **TypeScript 方式操作数据库**
- 极少 SQL，但不限制你写 SQL

# 二、Prisma 快速上手（10 分钟跑起来）

## 1️⃣ 安装

```cmd
npm init -y
npm install prisma --save-dev
npm install @prisma/client
```

初始化：

```cmd
npx prisma init
```

生成结构：

```css
prisma/
 └─ schema.prisma
.env
```

------

## 2️⃣ 配置数据库（以 PostgreSQL 为例）

```yaml
.env
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

------

## 3️⃣ 定义第一个 Model

```js
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
}
```

执行迁移：

```
npx prisma migrate dev --name init
```

👉 会发生三件事：

1. 创建数据库表
2. 生成 migration
3. 生成 Prisma Client

------

## 4️⃣ 使用 Prisma Client

```cmd
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const user = await prisma.user.create({
    data: {
      email: 'test@example.com',
      name: 'Alice',
    },
  })

  console.log(user)
}

main()
```

------

# 三、核心概念（必须吃透）

## 1️⃣ Prisma Schema

### 基础字段类型

```js
String
Int
BigInt
Boolean
Float
Decimal
DateTime
Json
Bytes
```

### 可选字段

```
name String?
```

### 默认值

```
createdAt DateTime @default(now())
```

------

## 2️⃣ 主键 & 唯一索引

```
id    Int    @id @default(autoincrement())
uuid  String @unique
```

复合唯一索引（很常见）：

```
@@unique([chainId, address])
```

👉 **钱包地址 + 链 ID** 场景非常常用

------

## 3️⃣ 关系（重点）

### 一对多

```ts
model User {
  id     Int     @id @default(autoincrement())
  orders Order[]
}

model Order {
  id     Int  @id @default(autoincrement())
  userId Int
  user   User @relation(fields: [userId], references: [id])
}
```

查询：

```js
prisma.user.findMany({
  include: {
    orders: true,
  },
})
```

------

### 多对多（中间表）

```
model User {
  id    Int    @id
  roles Role[]
}

model Role {
  id    Int    @id
  users User[]
}
```

Prisma 会自动生成中间表 👍

------

# 四、CRUD 精通（90% 时间都在用）

## 1️⃣ Create

```js
prisma.user.create({
  data: {
    email: 'a@a.com',
  },
})
```

嵌套创建：

```
prisma.user.create({
  data: {
    email: 'a@a.com',
    orders: {
      create: [{ amount: 100 }],
    },
  },
})
```

------

## 2️⃣ Read（查询）

### 常用查询

```
prisma.user.findMany()
prisma.user.findUnique({ where: { id: 1 } })
prisma.user.findFirst({ where: { email: { contains: '@' } } })
```

### 条件查询

```
where: {
  AND: [
    { chainId: 1 },
    { amount: { gt: 100 } },
  ],
}
```

### 分页

```
prisma.order.findMany({
  skip: 0,
  take: 20,
  orderBy: { createdAt: 'desc' },
})
```

------

## 3️⃣ Update

```
prisma.user.update({
  where: { id: 1 },
  data: { name: 'Bob' },
})
```

------

## 4️⃣ Delete

```
prisma.user.delete({
  where: { id: 1 },
})
```

------

# 五、事务（DApp 场景非常重要）

## 1️⃣ $transaction

```
await prisma.$transaction(async (tx) => {
  await tx.user.update(...)
  await tx.order.create(...)
})
```

👉 适合：

- 转账记录
- 余额变更
- 状态机更新

------

# 六、进阶必会

## 1️⃣ 枚举（非常推荐）

```
enum ChainType {
  EVM
  SOLANA
  BITCOIN
  TRON
}
chain ChainType
```

------

## 2️⃣ JSON 字段（存交易原始数据）

```
rawTx Json
```

------

## 3️⃣ 原生 SQL

```
await prisma.$queryRaw`
  SELECT * FROM "User" WHERE email LIKE '%@%'
`
```

------

## 4️⃣ Prisma Studio（神器）

```
npx prisma studio
```

👉 类似一个数据库 Admin UI，调试神器

------

# 七、工程最佳实践（很重要）

## 1️⃣ Prisma Client 单例

```
const globalForPrisma = global as any

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient()

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma
}
```

------

## 2️⃣ Model 设计建议（钱包/DApp）

```
model Wallet {
  id      Int    @id
  address String
  chain   ChainType

  @@unique([address, chain])
}
```

------

## 3️⃣ 不要滥用 include

- 列表页：select 必要字段
- 详情页：include 关系

------

# 八、学习路线总结（建议顺序）

### 🚀 入门（1–2 天）

- schema
- migrate
- CRUD
- Prisma Studio

### 🚀 熟练（3–5 天）

- 关系
- 事务
- 枚举
- JSON

### 🚀 精通（项目级）

- 复杂查询
- 性能优化
- SQL 混用
- 数据建模