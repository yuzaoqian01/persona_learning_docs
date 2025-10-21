## 一、CAIP-122 是什么？

CAIP-122 是由 ChainAgnostic 提出的改进提案之一，目标是提供一个 **区块链无关 (“chain-agnostic”) 的 “登录/签名” 消息格式**，用于验证用户拥有某条链上的账户。换言之，它扩展了 “Sign-In With Ethereum” (EIP-4361) 的思想，使其可以适用于不仅仅是以太坊，还包括 Solana、Stacks、其他链。

------

## 二、背景与缘由

- 在 Web3 应用中，一个常见需求：用户用钱包签名以证明「我控制这个账户」。以太坊生态中使用 EIP-4361 (“Sign-In With Ethereum”)。
- 但随着多链生态发展，需要一个能够通用多链／多签名算法的登录格式。
- CAIP-122 提出：让不同链都使用一个统一格式的 “Sign In With X（SIWX）” 消息结构。
  - 例如链为以太坊：`eip155:1:0xabc...`
  - 链为 Solana：`solana:mainnet:...`  等等。
  - 这样 DApp、钱包、身份系统能够用同一套机制实现“用户签名登录”流程。 [Phantom 开发者文档+2Chain Agnostic Namespaces+2](https://docs.phantom.com/developer-powertools/signing-a-message?utm_source=chatgpt.com)
- 也使得钱包连接、DID、授权、公钥注册（如 CACAO 消息格式）等机制能够具备一致基础。 [WalletConnect 规范+1](https://specs.walletconnect.com/2.0/specs/clients/core/identity/identity-keys?utm_source=chatgpt.com)

------

## 三、规范结构与关键字段

### 消息格式（参考 EIP-4361 风格）

CAIP-122 消息通常形式为：

```js
${domain} wants you to sign in with your ${blockchain} account:
${account}

${statement}

URI: ${uri}
Version: ${version}
Chain ID: ${chain-id}
Nonce: ${nonce}
Issued At: ${issued-at}
Expiration Time: ${expiration-time}   # 可选
Not Before: ${not-before}             # 可选
Request ID: ${request-id}             # 可选
Resources:
- ${resources[0]}
- ${resources[1]}
...
- ${resources[n]}
```

示例（以太坊）：

```js
magicapp.io wants you to sign in with your Ethereum account:
eip155:1:0xb9c5714089478a327f09197987f16f9e5d936e8a

Click Sign or Approve only means you have proved this wallet is owned by you.

URI: https://magicapp.io
Version: 1
Chain ID: eip155:1
Nonce: bZQJ0SL6gJ
Issued At: 2022-10-25T16:52:02.748Z
Resources:
- https://foo.com
- https://bar.com
```

[Phantom 开发者文档+1](https://docs.phantom.com/developer-powertools/signing-a-message?utm_source=chatgpt.com)

### 主要字段解释

- `domain`: 请求方域名／应用名
- `account`: 用户的区块链账户标识，通常采用 CAIP-10 格式（如 `eip155:1:0x…`）
- `statement`: 用户附加确认语句，如 “Click Sign or Approve only means you have proved this wallet is owned by you.”
- `URI`: 应用的 URI
- `Version`: 消息规范版本
- `Chain ID`: 链标识，使用 CAIP-2 规范（如 `eip155:1`）
- `Nonce`: 随机值，用做防重放
- `Issued At`: 发行时间
- `Expiration Time`, `Not Before`, `Request ID`, `Resources`: 可选扩展字段

### 签名算法／验证

- 根据具体链的签名机制，例如以太坊使用 ECDSA/secp256k1 + EIP-191 验签，Solana 使用 Ed25519 等。 [Chain Agnostic Namespaces+2Chain Agnostic Namespaces+2](https://namespaces.chainagnostic.org/solana/caip122?utm_source=chatgpt.com)
- 验证流程通常包括：将以上格式转换为标准字符串、编码为字节、用户钱包签名；随后验证签名恢复出的公钥或地址是否匹配提供的 account。

------

## 四、与其他规范的关系

| 规范     | 作用                                       | 与 CAIP-122 的关系                                           |
| -------- | ------------------------------------------ | ------------------------------------------------------------ |
| EIP‑4361 | “Sign-In With Ethereum” 登录格式，仅以太坊 | CAIP-122 可视为其多链通用版。 [Chain Agnostic Namespaces+1](https://namespaces.chainagnostic.org/eip155/caip122?utm_source=chatgpt.com) |
| CAIP‑10  | 多链账户标识格式（链＋地址）               | CAIP-122 中 `account` 字段通常用 CAIP-10 表示。 [WalletConnect 规范+1](https://specs.walletconnect.com/2.0/specs/clients/core/identity/identity-keys?utm_source=chatgpt.com) |
| CAIP‑2   | 多链标识格式（namespace:reference）        | CAIP-122 中 `Chain ID` 使用该格式（如 `eip155:1`） [Chain Agnostic Namespaces](https://namespaces.chainagnostic.org/eip155/caip122?utm_source=chatgpt.com) |

------

##  五、开发者使用场景与示例

### 使用场景

- 用户通过钱包在 dApp 中 “连接 + 登录” 检查自己地址的所有权。
- 钱包或协议注册 “身份公钥（Identity Key）” 时，使用 CAIP-122 消息来授权。 （例如 WalletConnect 的 Identity Keys 部分） [WalletConnect 规范](https://specs.walletconnect.com/2.0/specs/clients/core/identity/identity-keys?utm_source=chatgpt.com)
- 多链应用希望支持以太坊、Solana、Stacks 等不同链用户登录，使用统一格式便于处理。

###  示例代码（以 Web3 + EVM 为例）

```js
const message = `
myapp.io wants you to sign in with your Ethereum account:
eip155:1:0xAbc123...

Click “Sign” only means you agree to login.

URI: https://myapp.io
Version: 1
Chain ID: eip155:1
Nonce: 123ABC
Issued At: ${new Date().toISOString()}
`;

// 请求钱包签名
const signature = await provider.request({
  method: "personal_sign",
  params: [message, accountAddress]
});

// 在后端验证签名：恢复 address 是否等于 accountAddress。
// 若验证通过，则用户所有权证明有效。
```

### 多链变体（Solana）

```js
magicapp.io wants you to sign in with your Solana account:
solana:mainnet:FYpB58cLw5...

URI: https://magicapp.io
Version: 1
Chain ID: solana:mainnet
Nonce: 32891757
Issued At: 2021-09-30T16:25:24.000Z
```

[Chain Agnostic Namespaces](https://namespaces.chainagnostic.org/solana/caip122?utm_source=chatgpt.com)

------

## 六、优势 &注意事项

### 优势

- 多链统一格式，降低开发多链登录流程复杂度。
- 与现有登录标准（如 SIWE/EIP-4361）兼容，且扩展至非 EVM 链。
- 支持钱包/协议实现可通用的身份验证接口。

###  注意事项

- 虽为草稿状态（Draft），部分链或钱包尚未完全实现。 [hackmd.io](https://hackmd.io/@chainagnostic/rJKQMx37i?utm_source=chatgpt.com)
- 在具体链上需指定正确的签名算法／地址格式（EVM vs Solana vs Stacks 等）。
- DApp 后端验证时必须严格解析消息格式、验证 nonce、防重放、校验时间戳等安全事项。
- 多链时需确保 `chain-id` 字段与你期望的链一致，避免用户在错误链登录。

------

## 七、总结

CAIP-122 是一个强大的、面向未来的 “多链登录/签名”规范。它将登录消息结构标准化、链标识标准化、账户标识标准化，可帮助 Web3 应用在多链环境下更简单、更安全地处理用户身份。