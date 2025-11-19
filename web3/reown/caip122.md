# CAIP-122 标准说明

## 概述

CAIP-122 (Sign in With X, SIWx) 是一个**链无关**的身份验证标准，用于区块链账户在链下服务中的身份验证和授权。它是 EIP-4361 (Sign-In with Ethereum) 的通用化版本，支持所有符合 CAIP-10 和 CAIP-2 格式的区块链。

### 相关标准

- **CAIP-2**: 区块链 ID 规范
- **CAIP-10**: 账户 ID 规范  
- **CAIP-74**: CACAO (链无关能力对象)
- **CAIP-104**: 命名空间规范
- **EIP-4361**: Sign-In with Ethereum (CAIP-122 的以太坊实现)

---

## 数据模型

### 必填字段

| 字段名 | 类型 | 说明 | 示例 |
|--------|------|------|------|
| `domain` | string | RFC 4501 dnsauthority，请求签名的域名 | `FTGstake.com` |
| `account_address` | string | 执行签名的区块链地址，只包含 CAIP-10 的 account_address 部分，**不应该**包含 CAIP-2 的 chain_id | `0x1234567890123456789012345678901234567890` |
| `uri` | string | RFC 3986 URI，签名主体资源 | `https://FTGstake.com` |
| `version` | string | 消息的当前版本 | `1` |
| `chain_id` | string | CAIP-10 的 chain_id 部分，即 CAIP-2 标识符 | `eip155:1` |
| `signature` | bytes | 钱包签名的消息 | `0x1234...` |
| `type` | string | 签名类型（由命名空间定义） | `eip191`, `eip1271` |

### 可选字段

| 字段名 | 类型 | 说明 | 示例 |
|--------|------|------|------|
| `statement` | string | 人类可读的 ASCII 声明，**不能**包含 `\n` | `I accept the ServiceOrg Terms of Service: https://service.org/tos` |
| `nonce` | string | 随机令牌，用于防止签名重放攻击 | `32891756` |
| `issued-at` | string | RFC 3339 日期时间，签发时间 | `2021-09-30T16:25:24.000Z` |
| `expiration-time` | string | RFC 3339 日期时间，过期时间 | `2021-09-30T16:30:24.000Z` |
| `not-before` | string | RFC 3339 日期时间，生效时间 | `2021-09-30T16:25:00.000Z` |
| `request-id` | string | 系统特定标识符，唯一标识认证请求 | `req_123456` |
| `resources` | List<string> | RFC 3986 URI 列表，用 `\n` 分隔 | `['ipfs://...', 'https://...']` |

---

## 消息格式

### 标准格式模板

```js
{domain} wants you to sign in with your {blockchain} account:
{account_address}

{statement}

URI: {uri}
Version: {version}
Nonce: {nonce}
Issued At: {issued-at}
Expiration Time: {expiration-time}
Not Before: {not-before}
Request ID: {request-id}
Chain ID: {chain_id}
Resources:
- {resources[0]}
- {resources[1]}
...
- {resources[n]}
```

**格式说明:**

- `{blockchain}`: 人类可读的区块链生态系统名称（如 Ethereum、Solana）
- `{account_address}`: CAIP-10 地址的 account_address 部分
- `{chain_id}`: CAIP-10 地址的 chain_id 部分（CAIP-2 格式）

---

## 示例

### Ethereum 示例

```js
FTGstake.com wants you to sign in with your Ethereum account:
0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2

I accept the ServiceOrg Terms of Service: https://service.org/tos

URI: https://service.org/login
Version: 1
Nonce: 32891756
Issued At: 2021-09-30T16:25:24Z
Chain ID: eip155:1
Resources:
- ipfs://bafybeiemxf5abjwjbikoz4mc3a3dla6ual3jsgpdr4cjr3oz3evfyavhwq/
- https://example.com/my-web2-claim.json
```

**对应的数据:**

- `account_address`: `0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2`
- `chain_id`: `eip155:1`
- `accountAddress` (CAIP-10): `eip155:1:0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2`
- `type`: `eip191` (personal_sign)

---

### Solana 示例

Solana 不支持明文签名，需要签名原始字节。首先生成明文表示，然后转换为字节：

**明文表示:**

```js
service.org wants you to sign in with your Solana account:
GwAF45zjfyGzUbd3i3hXxzGeuchzEZXwpRYHZM5912F1

I accept the ServiceOrg Terms of Service: https://service.org/tos

URI: https://service.org/login
Version: 1
Nonce: 32891757
Issued At: 2021-09-30T16:25:24.000Z
Chain ID: solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d
Resources:
- ipfs://Qme7ss3ARVgxv6rXqVPiikMJ8u2NLgmgszg13pYrDKEoiu
- https://example.com/my-web2-claim.json
```

**对应的数据:**

- `account_address`: `GwAF45zjfyGzUbd3i3hXxzGeuchzEZXwpRYHZM5912F1`
- `chain_id`: `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d`
- `accountAddress` (CAIP-10): `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d:GwAF45zjfyGzUbd3i3hXxzGeuchzEZXwpRYHZM5912F1`
- `type`: `solana:ed25519`

---

## CAIP-122 vs EIP-4361 对比

### 主要差异

| 特性 | EIP-4361 | CAIP-122 |
|------|----------|----------|
| **链支持** | 仅 Ethereum | 所有区块链 |
| **Chain ID 格式** | 数字 (如 `1`) | CAIP-2 格式 (如 `eip155:1`) |
| **字段命名** | 驼峰 (`issuedAt`) | 连字符 (`issued-at`) |
| **地址格式** | 以太坊地址 | CAIP-10 account_address |
| **签名类型** | 隐式 (EIP-191/1271) | 明确的 `type` 字段 |
| **标准化程度** | Ethereum 特定 | 链无关，更通用 |

### 字段名称对照表

| EIP-4361 | CAIP-122 | 说明 |
|----------|----------|------|
| `address` | `account_address` | 仅包含地址部分 |
| - | `chain_id` | 新增必填字段 |
| `chainId` | `chain_id` | 从数字变为 CAIP-2 格式 |
| `issuedAt` | `issued-at` | 字段名改为连字符 |
| `expirationTime` | `expiration-time` | 字段名改为连字符 |
| `notBefore` | `not-before` | 字段名改为连字符 |
| `requestId` | `request-id` | 字段名改为连字符 |
| - | `type` | 新增必填字段 |
| - | `signature` | 新增必填字段（数据模型） |

### 兼容性说明

✅ **向后兼容**: EIP-4361 可以看作是 CAIP-122 的特定实现
✅ **升级路径**: EIP-4361 应用可以升级到 CAIP-122 以支持多链
✅ **格式转换**: 可以轻松将 EIP-4361 消息转换为 CAIP-122 格式

---

## Chain ID (CAIP-2) 格式

CAIP-2 定义了区块链标识符的标准格式：

```js
{namespace}:{reference}
```

### 常见 Chain ID

| 区块链 | Namespace | Chain ID | 说明 |
|--------|-----------|----------|------|
| Ethereum Mainnet | eip155 | `eip155:1` | 以太坊主网 |
| Ethereum Goerli | eip155 | `eip155:5` | Goerli 测试网 |
| Ethereum Sepolia | eip155 | `eip155:11155111` | Sepolia 测试网 |
| Polygon Mainnet | eip155 | `eip155:137` | Polygon 主网 |
| BSC Mainnet | eip155 | `eip155:56` | 币安智能链 |
| Arbitrum One | eip155 | `eip155:42161` | Arbitrum L2 |
| Optimism | eip155 | `eip155:10` | Optimism L2 |
| Solana Mainnet | solana | `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d` | Solana 主网 |
| Solana Devnet | solana | `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1` | Solana 开发网 |
| Cosmos Hub | cosmos | `cosmos:cosmoshub-4` | Cosmos 主网 |
| Polkadot | polkadot | `polkadot:91b171bb158e2d3848fa23a9f1c25182` | Polkadot 中继链 |

### EIP-155 Namespace

对于所有兼容 EVM 的链，使用 `eip155` namespace + 链的 Chain ID：

```typescript
const chainId = 1  // Ethereum Mainnet
const caip2 = `eip155:${chainId}`  // "eip155:1"
```

---

## Account Address (CAIP-10) 格式

CAIP-10 定义了账户地址的标准格式：

```js
{chain_id}:{account_address}
```

### 示例

| 区块链 | CAIP-10 格式 |
|--------|--------------|
| Ethereum | `eip155:1:0x1234567890123456789012345678901234567890` |
| Polygon | `eip155:137:0x1234567890123456789012345678901234567890` |
| Solana | `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d:GwAF45zjfyGzUbd3i3hXxzGeuchzEZXwpRYHZM5912F1` |
| Cosmos | `cosmos:cosmoshub-4:cosmos1t2uflqwqe0fsj0shcfkrvpukewcw40yjj6hdc0` |

**重要**: 在 CAIP-122 消息中，`account_address` 字段**只包含地址部分**，不包含 `chain_id`。

---

## 签名类型 (type 字段)

`type` 字段由各个区块链命名空间定义，指定签名算法。

### Ethereum (eip155 namespace)

| Type | 说明 | 标准 |
|------|------|------|
| `eip191` | Personal sign (钱包签名) | EIP-191 |
| `eip1271` | Contract signature (合约签名) | EIP-1271 |

### Solana (solana namespace)

| Type | 说明 |
|------|------|
| `solana:ed25519` | Ed25519 签名 |

### 其他链

每个链的命名空间规范应该定义其支持的签名类型。

---

## 实现要求

### 命名空间实现

每个支持 CAIP-122 的命名空间必须提供：

1. **签名算法**: 一个或多个签名接口
2. **Type 字符串**: 用于 `type` 字段的标识符
3. **签名输入生成**: 如何从数据模型创建签名输入
4. **签名方法**: 如何签名
5. **验证方法**: 如何验证签名

### 钱包实现要求

#### 域名绑定验证

钱包实现者**必须**通过匹配 `domain` 字段来防止钓鱼攻击：

```typescript
// ✅ 正确：验证域名匹配
if (message.startsWith('service.invalid wants you to sign in')) {
  const actualDomain = window.location.hostname
  if (actualDomain !== 'service.invalid') {
    throw new Error('Domain mismatch! Possible phishing attack.')
  }
}
```

#### 域名来源

域名应该从可信数据源读取，例如：

- 浏览器窗口的 `window.location.hostname`
- 移动应用的已验证应用域
- WebView 的 `document.domain`

#### 用户界面

钱包应该清楚地向用户显示：

1. 请求签名的域名
2. 正在签名的账户地址
3. 签名声明（statement）
4. 链 ID
5. 过期时间

---

## TypeScript 类型定义

```typescript
/**
 * CAIP-122 数据模型
 */
interface CAIP122Message {
  // 必填字段
  domain: string                    // RFC 4501 dnsauthority
  account_address: string           // CAIP-10 account_address 部分
  uri: string                       // RFC 3986 URI
  version: string                   // 消息版本
  chain_id: string                  // CAIP-2 链标识符
  signature: Uint8Array             // 签名字节
  type: string                      // 签名类型
  
  // 可选字段
  statement?: string                // 不能包含 \n
  nonce?: string                    // 随机令牌
  'issued-at'?: string             // RFC 3339 时间
  'expiration-time'?: string       // RFC 3339 时间
  'not-before'?: string            // RFC 3339 时间
  'request-id'?: string            // 请求标识符
  resources?: string[]             // URI 列表
}

/**
 * 前端使用的参数接口（驼峰命名）
 */
interface SIWXMessageParams {
  domain: string
  accountAddress: string            // CAIP-10 完整格式
  uri: string
  version: string
  chainId: string                   // CAIP-2 格式
  statement?: string
  nonce?: string
  issuedAt?: string
  expirationTime?: string
  notBefore?: string
  requestId?: string
  resources?: string[]
}

/**
 * 验证请求接口
 */
interface VerifyRequest {
  message: string                   // 完整的格式化消息
  signature: string                 // Hex 格式签名
  address: string                   // 仅地址部分
  chainId: string                   // CAIP-2 格式
  accountAddress: string            // CAIP-10 格式
  type?: string                     // 签名类型
}
```

---

## 安全考虑

### 1. 防重放攻击

- ✅ **使用 nonce**: 每个签名请求必须使用唯一的 nonce
- ✅ **nonce 过期**: nonce 应该有过期时间（建议 5-10 分钟）
- ✅ **一次性使用**: nonce 验证后应立即失效

### 2. 防钓鱼攻击

- ✅ **域名验证**: 钱包必须验证 domain 字段与实际来源匹配
- ✅ **UI 提示**: 在签名界面清楚显示域名
- ✅ **警告机制**: 对可疑域名给出警告

### 3. 时间验证

- ✅ **issued-at**: 验证签发时间不能是未来时间
- ✅ **expiration-time**: 验证消息未过期
- ✅ **not-before**: 验证消息已生效

### 4. 链验证

- ✅ **链匹配**: 验证 chain_id 与用户当前连接的链一致
- ✅ **支持的链**: 只接受支持的 chain_id

---

## 参考资源

### 官方规范

- [CAIP-122 规范](https://chainagnostic.org/CAIPs/caip-122)
- [CAIP-2: 区块链 ID](https://chainagnostic.org/CAIPs/caip-2)
- [CAIP-10: 账户 ID](https://chainagnostic.org/CAIPs/caip-10)
- [CAIP-74: CACAO](https://chainagnostic.org/CAIPs/caip-74)
- [EIP-4361: Sign-In with Ethereum](https://eips.ethereum.org/EIPS/eip-4361)

### RFC 标准

- [RFC 3339: 日期和时间](https://datatracker.ietf.org/doc/html/rfc3339)
- [RFC 3986: URI 语法](https://www.rfc-editor.org/rfc/rfc3986)
- [RFC 4501: DNS URI](https://www.rfc-editor.org/rfc/rfc4501.html)

### 以太坊标准

- [EIP-191: 签名数据标准](https://eips.ethereum.org/EIPS/eip-191)
- [EIP-1271: 合约签名验证](https://eips.ethereum.org/EIPS/eip-1271)

---

## 版本历史

- **v1.0** (2022-06-23): 初始规范发布
- **当前状态**: Review（审查中）

---

## 许可证

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/)