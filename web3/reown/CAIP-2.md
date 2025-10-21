## 一、CAIP 是什么？

**CAIP** 是 “**Chain Agnostic Improvement Proposal**”（链无关改进提案）的缩写，
 是由 *ChainAgnostic.org* 发起的一系列标准，
 目标是让不同区块链生态（EVM、Solana、Cosmos、Bitcoin、Tron、Aptos...）之间能通过统一标识互通。

> 就像每个人都有一个身份证号，CAIP 定义了「链」的身份证号。

------

##  二、CAIP-2 是哪一部分？

CAIP-2 定义了 **「链的唯一标识符（Chain ID）」的命名规范**：

```
<namespace>:<reference>
```

例子：

```
eip155:1       → Ethereum 主网
eip155:137     → Polygon 主网
solana:mainnet → Solana 主网
bip122:000000000019d6689c085ae165831e93 → 比特币主网
tron:0x2b6653dc → Tron 主网
```

------

## 三、结构拆解

| 字段          | 含义                         | 示例                                                    |
| ------------- | ---------------------------- | ------------------------------------------------------- |
| **namespace** | 技术体系（定义链的类型）     | `eip155`、`solana`、`bip122`、`cosmos`、`tron`、`aptos` |
| **reference** | 在该体系中的链 ID 或唯一标识 | `1`、`137`、`mainnet`、`000000...e93`                   |

------

### 常见 namespace 对照

| Namespace  | 来源                       | 示例                                      | 说明          |
| ---------- | -------------------------- | ----------------------------------------- | ------------- |
| `eip155`   | EIP-155 (EVM 生态)         | `eip155:1`                                | Ethereum 主网 |
| `bip122`   | 比特币类链（区块哈希标识） | `bip122:000000000019d6689c085ae165831e93` | Bitcoin 主网  |
| `cosmos`   | Cosmos SDK                 | `cosmos:cosmoshub-4`                      | Cosmos Hub    |
| `solana`   | Solana 链                  | `solana:mainnet`                          | Solana 主网   |
| `tron`     | Tron 链                    | `tron:0x2b6653dc`                         | Tron 主网     |
| `aptos`    | Aptos 链                   | `aptos:mainnet`                           | Aptos 主网    |
| `starknet` | StarkNet                   | `starknet:mainnet`                        | StarkNet 主网 |

------

## 四、为什么需要 CAIP-2？

### 1. 跨链统一识别

不同生态的链 ID 命名方式各不相同：

- EVM 链用 `chainId`
- Solana 用 `cluster`
- Bitcoin 用 `genesis block hash`

CAIP-2 让它们都有相同的语法结构。

------

### 2. 多链 DApp 的关键基础

在 WalletConnect v2、Reown AppKit、wagmi、LI.FI API、RainbowKit 等系统中，
 当你声明支持多链时，都是用 CAIP-2 格式：

```js
namespaces: {
  eip155: {
    chains: ['eip155:1', 'eip155:137', 'eip155:56'],
    methods: ['eth_sendTransaction', 'personal_sign'],
    events: ['accountsChanged', 'chainChanged']
  },
  solana: {
    chains: ['solana:mainnet']
  }
}
```

------

### 3. 跨链签名与身份系统的基础

在 DID、跨链消息协议（如 CAIP-10、CAIP-25、Chainlink CCIP、IBC 桥）中，
 CAIP-2 的链标识是身份与资源地址的前缀。

例如 CAIP-10 账户标识：

```js
eip155:1:0x1234...abcd
```

> = EIP-155 主网上的账户 0x1234...abcd

------

## 五、与 EIP-155 的关系

| 对比点   | EIP-155                      | CAIP-2                                      |
| -------- | ---------------------------- | ------------------------------------------- |
| 来源     | Ethereum 改进提案            | ChainAgnostic 标准                          |
| 作用     | 给 EVM 链引入 chainId 防重放 | 用统一语法标识任意链                        |
| 使用范围 | 仅 EVM 生态                  | 全链（EVM、Solana、Bitcoin、Cosmos、Tron…） |
| 示例     | `chainId = 1`                | `eip155:1`                                  |

------

## 六、现代生态中的应用

| 平台 / 库               | 使用场景                               |
| ----------------------- | -------------------------------------- |
| **WalletConnect v2**    | 所有会话均基于 CAIP-2 链标识           |
| **Reown AppKit**        | 连接配置用 eip155:1、solana:mainnet 等 |
| **wagmi / viem**        | 内部 chainId 对应 CAIP-2 reference     |
| **DID / NFT 跨链**      | 使用 CAIP-2 + CAIP-10 统一账户标识     |
| **跨链桥 / Aggregator** | 解析链来源、合并交易路径               |

------

## 七、实际使用示例（WalletConnect）

```js
import { mainnet, polygon } from 'wagmi/chains'

const projectId = 'YOUR_PROJECT_ID'

const metadata = {
  name: 'My MultiChain DApp',
  description: 'Demo using CAIP-2',
  url: 'https://mydapp.io',
  icons: ['https://mydapp.io/icon.png'],
}

const wagmiAdapter = new WagmiAdapter({
  projectId,
  networks: [mainnet, polygon],
  namespaces: {
    eip155: {
      chains: ['eip155:1', 'eip155:137'],
      methods: ['eth_sendTransaction', 'personal_sign'],
      events: ['chainChanged', 'accountsChanged'],
    },
  },
})
```

------

## 八、扩展标准

CAIP-2 只是 CAIP 系列的起点。后续相关：

| 标准        | 功能                          |
| ----------- | ----------------------------- |
| **CAIP-2**  | 链标识符（Chain Identifier）  |
| **CAIP-10** | 账户标识符（Chain + Address） |
| **CAIP-19** | 资产标识符（Chain + Token）   |
| **CAIP-50** | 通用资源定位（跨链 URI）      |

举例：

```
CAIP-10: eip155:1:0x1234...
CAIP-19: eip155:1/erc20:0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48
```

------

## 九、总结

| 项           | 内容                                       |
| ------------ | ------------------------------------------ |
| **名称**     | Chain Agnostic Improvement Proposal 2      |
| **格式**     | `<namespace>:<reference>`                  |
| **作用**     | 为所有链定义统一标识                       |
| **典型示例** | `eip155:1`、`solana:mainnet`、`bip122:...` |
| **核心意义** | 跨链互操作、钱包连接、统一身份体系的基石   |