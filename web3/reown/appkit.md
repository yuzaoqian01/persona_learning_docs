# Appkit 的熟悉与使用

## 一、安装

```cmd
pnpm add @reown/appkit @reown/appkit-universal-connector @reown/appkit-common ethers
```

为了快速集成 Appkit Core，您可以使用 `UniversalConnector` 类。它通过为所有区块链协议提供单一接口来简化 Appkit Core 的集成。

您可以使用要支持的网络配置通用连接器。有关更多信息，请访问我们文档中的 [RPC 参考](https://docs.reown.com/advanced/multichain/rpc-reference/cosmos-rpc)部分。

源码配置

provider.ts

```ts
import type { AppKitNetwork } from '@reown/appkit/networks'
import type { CustomCaipNetwork } from '@reown/appkit-common'
import { UniversalConnector } from '@reown/appkit-universal-connector'

// Get projectId from https://dashboard.reown.com 申请的renown的项目id
export const projectId = import.meta.env.VITE_PROJECT_ID || "b56e18d47c72ab683b10814fe9495694" // this is a public projectId only to use on localhost

if (!projectId) {
  throw new Error('Project ID is not defined')
}

// you can configure your own network // 自定义网络配置 配置区块链的provider
const suiMainnet: CustomCaipNetwork<'sui'> = {
  id: 784,
  chainNamespace: 'sui' as const,
  caipNetworkId: 'sui:mainnet',
  name: 'Sui',
  nativeCurrency: { name: 'SUI', symbol: 'SUI', decimals: 9 },
  rpcUrls: { default: { http: ['https://fullnode.mainnet.sui.io:443'] } }
}

// 初始化appkit 的网络适配器
export async function getUniversalConnector() {
  const universalConnector = await UniversalConnector.init({
    projectId,
    metadata: { // 项目的Meta 信息
      name: 'Universal Connector', // 项目名称
      description: 'Universal Connector', // 项目描述
      url: 'https://appkit.reown.com', // 项目域名
      icons: ['https://appkit.reown.com/icon.png'] // 项目图标logo 
    },
    networks: [
      {
        methods: ['sui_signPersonalMessage'], // 支持的方法
        chains: [suiMainnet as CustomCaipNetwork], // 支持的链
        events: [],//支持的事件
        namespace: 'sui'// 命名空间
      }
    ]
  })

  return universalConnector
}
```

在入口文件

```tsx
// app.tsx
import { useState, useEffect } from 'react'
import { getUniversalConnector } from './config' // previous config file
import { UniversalConnector } from '@reown/appkit-universal-connector'

export function App() {
  const [universalConnector, setUniversalConnector] = useState<UniversalConnector>()
  const [session, setSession] = useState<any>()

  
  // Initialize the Universal Connector on component mount
  useEffect(() => {
    getUniversalConnector().then(setUniversalConnector)
  }, [])

  // Set the session state in case it changes 回话更新
  useEffect(() => {
    setSession(universalConnector?.provider.session)
  }, [universalConnector?.provider.session])
```



### 基础概念

`wagmi` 是一个用于 **React + EVM 区块链交互** 的非常流行的库。
 它相当于 Web3.js / ethers.js 的 **React 封装层**，让你能在 React 应用中 **优雅、安全、响应式地操作钱包和合约**。

**`viem` 是一个现代、类型安全、轻量级的以太坊与 EVM 链交互库**，
 由 wagmi 作者团队（[@tmm](https://github.com/tmm)）开发，用来取代 `ethers.js`。是t s版的ethers.js

`appkit`是一个钱包sdk 用于创建无缝的链上用户体验，提供登录 多链支持 等钱包功能

# Trigger the modal

```tsx
// 从通用连接器获取会话
// get the session from the universal connector
    const handleConnect = async () => {
      if (!universalConnector) {
        return
      }
  
      const { session: providerSession } = await universalConnector.connect()
      setSession(providerSession)
    };

 // 断开会话连接
    // disconnect the universal connector
    const handleDisconnect = async () => {
      if (!universalConnector) {
        return
      }
      await universalConnector.disconnect()
      setSession(null)
    };

    ...

    return (
    (
    <div>      
        <button onClick={handleConnect}>Open AppKit Core</button>
        <button onClick={handleDisconnect}>Disconnect</button>
    </div>
    )
```

### Hooks

AppKit 提供了一套全面的 React 钩子，它们协同工作以提供完整的钱包连接和区块链交互体验。这些钩子可以分为几个功能组：

#### Connection Hooks

**管理钱包连接和用户身份验证**

1. useAppKit		
2. useAppKitAccount
3. useAppKitProvider
4. useDisconnect

#### Network Hooks

**处理区块链网络选择和信息**

1. useAppKitNetwork

#### **UI Control Hooks**

**控制模式和 UI 元素**

1. useAppKitState
2. useAppKitTheme

#### **Data Access Hooks**

**访问钱包和区块链数据**

1. useAppKitBalance
2. useWalletInfo

#### **Event Hooks**

**订阅钱包和连接事件**

1. useAppKitEvents

#### **Multi-Wallet Hooks**

**管理多个钱包连接**

1. useAppKitConnections
2. useAppKitConnection

#### **Authentication Hooks**

**处理身份验证和用户管理**

1. useAppKitUpdateEmail
2. useAppKitSIWX

#### **Solana-Specific Hooks**

**Solana 区块链交互**

1. useAppKitConnection

### 连接视图的命名空间

`solana` 

`bip122`

`eip155`

### 视图控制(web3modal Scuffold UI)

`Connect`

`Account`

`ALLWallets`

`Networks`

`WhatIsANetwork`

`WhatIsAWallet`

`OnRampProviders`

`WalletSend`

`Swap`

处于多链时账户访问

```ts
import { useAppKitAccount } from "@reown/appkit/react";

const eip155Account = useAppKitAccount({ namespace: "eip155" }); // for EVM chains
const solanaAccount = useAppKitAccount({ namespace: "solana" }); // solana
const bip122Account = useAppKitAccount({ namespace: "bip122" }); // for bitcoin
```

#### 获取比特币公钥

```tsx
import { useAppKitAccount } from "@reown/appkit/react";

function BitcoinComponent() {
  const { allAccounts } = useAppKitAccount({ chainNamespace: 'bip122' });
  const publicKeys = allAccounts.map(acc => acc.publicKey);
  
  return (
    <div>
      {publicKeys.map((key, index) => (
        <div key={index}>Public Key: {key}</div>
      ))}
    </div>
  );
}
```

#### 多链连接示例

```tsx
// Connect to Ethereum/EVM chains
const { connect: connectEVM } = useAppKitWallet({
  namespace: 'eip155',
  onSuccess: (address) => console.log('Connected to EVM:', address)
})

// Connect to Solana
const { connect: connectSolana } = useAppKitWallet({
  namespace: 'solana',
  onSuccess: (address) => console.log('Connected to Solana:', address)
})

// Connect to Bitcoin
const { connect: connectBitcoin } = useAppKitWallet({
  namespace: 'bip122',
  onSuccess: (address) => console.log('Connected to Bitcoin:', address)
})

// Usage
<Button onClick={() => connectEVM("metamask")}>Connect MetaMask (EVM)</Button>
<Button onClick={() => connectSolana("phantom")}>Connect Phantom (Solana)</Button>
<Button onClick={() => connectBitcoin("leather")}>Connect Leather (Bitcoin)</Button>
```

## 二 、SIWX

<p style="color:red;">签名消息格式 消息格式（参考 EIP-4361 风格）遵循CAIP-122 而不是简单的自定义字符串拼接</p>

使用 X 登录标准由 [CAIP-122](https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-122.md) 定义。它对`地址`字段使用 [CAIP-10](https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-10) 标识符，对 `chain-id` 使用 [CAIP-2](https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2)。

**SIWX** 用作 AppKit 的插件系统，您将在 AppKit 配置中添加插件。有一些方法可以实现 **SIWX** 功能：

- 使用 Reown Authentication 管理 Reown Dashboard 中的会话
- 使用 AppKit 提供的 `DefaultSIWX` 类
- 创建自定义实施以满足您的特定要求。

### SIWX Expected Behavior SIWX 预期行为

- 每次连接发生时，**SIWX** 都会提示获取用户签名并验证其身份;
- 如果已存储 SIWX 会话，用户将自动登录并跳过提示步骤;
- 如果用户更改了连接的网络，**SIWX** 将提示获取用户签名并再次验证其身份;
- 如果用户断开与 Dapp 的连接，**SIWX** 将撤销会话，用户需要重新登录。

### 1. DefaultSIWX 实现

#### 安装

```bash
pnpm add @reown/appkit-siwx
```

#### 用法

```ts
import { createAppKit } from '@reown/appkit'
// Add the following code line
import { DefaultSIWX } from '@reown/appkit-siwx'

const appkit = createAppKit({
  projectId,
  networks,
  metadata,
  // Add the following code line
  siwx: new DefaultSIWX() // add this line to enable SIWX
})
```

### 2. 自定义 DefaultSIWX 类

`DefaultSIWX` 类分为三个主要组件：`SIWXMessenger`、`SIWXVerifier` 和 `SIWXStorage`。`@reown/appkit-siwx` 包定义了在初始化 `DefaultSIWX` 配置时实现部件的选项，您还可以根据需要设置自己的部件。

`@reown/appkit-siwx` 包提供了一些预定义的组件，您可以使用它们来快速设置 `DefaultSIWX` 配置。Check the latest components over the [SIWX repository](https://github.com/reown-com/appkit/tree/main/packages/siwx)
通过 [SIWX 存储库](https://github.com/reown-com/appkit/tree/main/packages/siwx)检查最新组件

```ts
import {
  DefaultSIWX,
  InformalMessenger,
  EIP155Verifier,
  SolanaVerifier,
  LocalStorage
} from '@reown/appkit-siwx'

const siwx = new DefaultSIWX({
  messenger: new InformalMessenger({
    domain: 'reown.com',
    uri: 'https://reown.com',
    getNonce: async () => Math.round(Math.random() * 10000).toString()
  }), // 消息生产器
  verifiers: [new EIP155Verifier(), new SolanaVerifier()],//消息验证
  storage: new LocalStorage({ key: '@appkit/siwx' }) // 存储管理
})
```

### 3. 改造DefaultSIWX需要的自定义实现

#### `SIWXMessage`  消息生成器

是个抽象类 用于生成要签名的消息的方法

可以继承并添加公共属性

.  `version` 消息的版本

.  `stringify` 处理消息数据并返回签名字符串的方法

```js
import { SIWXMessenger } from '@reown/appkit-siwx'
import type { SIWXMessage } from '@reown/appkit-core'

// 继承这个类并实现对应方法和属性
export class MyMessenger extends SIWXMessenger {
  protected readonly version = '1'

  protected override stringify(params: SIWXMessage.Data): string {
    // Implement your message format here
    return `My message for ${params.accountAddress} on ${params.chainId}`
  }
}
```

#### `SIWXVerifier`消息验证器

是抽象类 用于定义已签名消息的验证逻辑

可以继承并添加公共属性

.  `chainNamespace` 表示验证者的链命名空间的字符串;

.  `verify`：接收会话数据并返回布尔值的方法，指示会话是否有效。

```js
import { SIWXVerifier } from '@reown/appkit-siwx'
import type { SIWXSession } from '@reown/appkit-core'


export class MyVerifier extends SIWXVerifier {
  public readonly chainNamespace = 'eip155' // set the chain namespace for your verifier

  public async verify(session: SIWXSession): Promise<boolean> {
    // Implement your verification logic here
    return true
  }
}
```

#### [`SIWXStorage`](https://github.com/reown-com/appkit/blob/main/packages/siwx/src/core/SIWXStorage.ts) 是一个接口，用于定义会话数据的存储方式。

- `add`：将调用此方法来存储已验证的新单个会话;
- `set`：此方法必须将存储中的所有会话替换为新会话;
- `get`：此方法必须返回为特定链和地址存储的所有会话。预计会话已经过验证;
- `delete`：此方法必须删除为特定链和地址存储的所有会话。

```js
import type { SIWXSession } from '@reown/appkit-core'
import type { SIWXStorage } from '@reown/appkit-siwx'

export class MyStorage implements SIWXStorage {
  add(session: SIWXSession): Promise<void> {
    // Implement your logic to add a session
  }

  set(sessions: SIWXSession[]): Promise<void> {
    // Implement your logic to set sessions
  }

  get(chainId: CaipNetworkId, address: string): Promise<SIWXSession[]> {
    // Implement your logic to get sessions
    return []
  }

  delete(chainId: string, address: string): Promise<void> {
    // Implement your logic to delete a session
  }
}
```

<p style="color:red;">如果省略任何组件，则将使用默认实现</p>

所以完全自定义实现需要的接口有

```js
interface SIWXConfig {
  createMessage: (input: SIWXMessage.Input) => Promise<SIWXMessage>
  addSession: (session: SIWXSession) => Promise<void>
  revokeSession: (chainId: CaipNetworkId, address: string) => Promise<void>
  setSessions: (sessions: SIWXSession[]) => Promise<void>
  getSessions: (chainId: CaipNetworkId, address: string) => Promise<SIWXSession[]>
  getRequired?: () => boolean
  signOutOnDisconnect?: boolean
}
  
  /**
 * 此接口表示 SIWX 配置插件，用于创建和管理 SIWX 消息和会话。
 * AppKit 通过 `@reown/appkit-siwx` 提供了此接口的预定义实现。
 * 你可以根据需要创建自定义实现，但请注意方法的要求。
 */
export interface SIWXConfig {
  /**
   * 此方法将在创建用户签名的新消息时调用。
   *
   * 约束条件：
   * - 消息必须唯一，并包含验证用户身份所需的所有信息。
   * - 必须实现 SIWXMessage.toString() 方法以返回消息字符串。
   *
   * @param input SIWXMessage.Input
   * @returns SIWXMessage
   */
  createMessage: (input: SIWXMessage.Input) => Promise<SIWXMessage>

  /**
   * 此方法将在存储新的单个会话时调用。
   *
   * 约束条件：
   * - 此方法必须验证会话是否有效，并成功将其存储到存储中。
   *
   * @param session SIWXSession
   */
  addSession: (session: SIWXSession) => Promise<void>

  /**
   * 此方法将在撤销特定链和地址存储的所有会话时调用。
   *
   * 约束条件：
   * - 此方法必须成功删除特定链和地址的所有会话。
   *
   * @param chainId CaipNetworkId
   * @param address string
   */
  revokeSession: (chainId: CaipNetworkId, address: string) => Promise<void>

  /**
   * 此方法将在用新的会话替换存储中的所有会话时调用。
   *
   * 约束条件：
   * - 此方法必须在存储之前验证所有会话；
   * - 此方法必须成功替换存储中的所有会话，否则必须抛出错误。
   *
   * @param sessions SIWXSession[]
   */
  setSessions: (sessions: SIWXSession[]) => Promise<void>

  /**
   * 此方法将在获取特定链和地址存储的所有会话时调用。
   *
   * 约束条件：
   * - 此方法必须只返回经过验证且有效的会话；
   * - 此方法不得返回已过期的会话。
   *
   * @param chainId CaipNetworkId
   * @param address string
   * @returns
   */
  getSessions: (chainId: CaipNetworkId, address: string) => Promise<SIWXSession[]>

  /**
   * 此方法决定当用户拒绝签名请求时，钱包是否保持连接。
   *
   * @returns {boolean}
   */
  getRequired?: () => boolean

  /**
   * 此方法决定当用户断开连接时，会话是否应被清除。
   *
   * @default true
   * @returns {boolean}
   */
  signOutOnDisconnect?: boolean
}

/**
 * 此接口表示 SIWX 会话，用于存储用户身份信息。
 */
export interface SIWXSession {
  data: SIWXMessage.Data
  message: string
  signature: string
  cacao?: Cacao
}

/**
 * 此接口表示 SIWX 消息，用于创建用户签名的消息。
 * 必须包含验证用户身份所需的信息以及生成消息字符串的方法。
 */
export interface SIWXMessage extends SIWXMessage.Data, SIWXMessage.Methods {}

export namespace SIWXMessage {
  /**
   * 此接口表示 SIWX 消息数据，用于创建用户签名的消息。
   */
  export interface Data extends Input, Metadata, Identifier {}

  /**
   * 此接口表示 SIWX 消息输入。
   * 此处必须包含应用中每个用户不同的内容。
   */
  export interface Input {
    accountAddress: string
    chainId: CaipNetworkId
    notBefore?: Timestamp
  }

  /**
   * 此接口表示 SIWX 消息元数据。
   * 此处必须包含与应用相关的主要数据。
   */
  export interface Metadata {
    domain: string
    uri: string
    version: string
    nonce: string
    statement?: string
    resources?: string[]
  }

  /**
   * 此接口表示 SIWX 消息标识符。
   * 此处必须包含请求 ID 和时间戳。
   */
  export interface Identifier {
    requestId?: string
    issuedAt?: Timestamp
    expirationTime?: Timestamp
  }

  /**
   * 此接口表示 SIWX 消息方法。
   * 此处必须包含生成消息字符串的方法以及 SIWX 消息执行的其他方法。
   */
  export interface Methods {
    toString: () => string
  }

  /**
   * 时间戳是表示时间的 ISO 8601 格式的 UTC 字符串。
   */
  export type Timestamp = string
}

/**
 * Cacao 接口参考了 CAIP-74，表示链无关的对象能力（OCAP）。
 * https://chainagnostic.org/CAIPs/caip-74
 */
export interface Cacao {
  h: Cacao.Header
  p: Cacao.Payload
  s: {
    t: 'eip191' | 'eip1271'
    s: string
    m?: string
  }
}

export namespace Cacao {
  export interface Header {
    t: 'caip122'
  }

  export interface Payload {
    domain: string
    aud: string
    nonce: string
    iss: string
    version?: string
    iat?: string
    nbf?: string
    exp?: string
    statement?: string
    requestId?: string
    resources?: string[]
    type?: string
  }
}

```

#### example

```js
import { createAppKit, type SIWXConfig } from '@reown/appkit'

const siwx: SIWXConfig = {
  createMessage: async (input) => {
    // Implement your logic to create a message
    return 'my message'
  }
  addSession: async (session) => {
    // Implement your logic to add a session
  },
  revokeSession: async (chainId, address) => {
    // Implement your logic to revoke a session
  },
  setSessions: async (sessions) => {
    // Implement your logic to set sessions
  },
  getSessions: async (chainId, address) => {
    // Implement your logic to get sessions
    return []
  },
  getRequired: () => {
    // Return whether the wallet should stay connected when user denies signature
    return false
  },
  signOutOnDisconnect: true // Whether to clear sessions when user disconnects
}

createAppKit({
  // ... your configuration
  siwx
})
```

## 三 Adapter

### Wagmi适配器

WagmiAdapter 使用 Wagmi 库与基于以太坊的网络集成。

**主要特点：**

- 与 Wagmi 的配置系统完全集成
- 支持符合 EIP-1193 标准的钱包
- ENS 名称解析和头像获取
- 交易建立和发送
- 合约交互
- 帐户和网络变化的事件监听器

```js
// Connect to a wallet
async connect(params: AdapterBlueprint.ConnectParams): Promise<AdapterBlueprint.ConnectResult>

// Sign a message
async signMessage(params: AdapterBlueprint.SignMessageParams): Promise<AdapterBlueprint.SignMessageResult>

// Send a transaction
async sendTransaction(params: AdapterBlueprint.SendTransactionParams): Promise<AdapterBlue
```

### EthersAdapter 和 Ethers5Adapter

这些适配器使用 ethers.js（分别为 v6+ 和 v5）与以太坊网络交互。

**主要特点：**

- 与 ethers.js 提供商直接集成
- 支持 EIP-6963 钱包发现
- ENS 解析
- Gas 估算和交易处理
- 钱包连接状态的事件监听器

```js
// Create ethers configuration
private async createEthersConfig(options: AppKitOptions)

// Listen for provider events
private listenProviderEvents(provider: Provider | CombinedProvider)

// Switch networks
public override async switchNetwork(params: AdapterBlueprint.SwitchNetworkParams): Pr
```

### Solana适配器

SolanaAdapter 提供与 Solana 区块链的集成。

**主要特点：**

- 连接到 Solana RPC 端点
- 支持各种 Solana 钱包适配器
- 交易创建和签名
- 通过 RPC 检查余额
- Solana 钱包状态的事件监听器

```js
// Sign a message
public async signMessage(params: AdapterBlueprint.SignMessageParams): Promise<AdapterBlueprint.SignMessageResult>

// Send a transaction
public async sendTransaction(params: AdapterBlueprint.SendTransactionParams): Promise<AdapterBlueprint.SendTransactionResult>
```

### 比特币适配器

BitcoinAdapter 提供与比特币区块链的集成。

**主要特点：**

- 支持比特币钱包（Xverse、Leather 等）
- 通过BitcoinApi管理UTXO
- 使用不同的协议（ECDSA、BIP322）对消息进行签名
- 多地址用途支持（付款、序数、堆栈）

```js
// Connect to a wallet
override async connect(params: AdapterBlueprint.ConnectParams): Promise<AdapterBlueprint.ConnectResult>

// Sign a message
override async signMessage(params: AdapterBlueprint.SignMessageParams): Promise<AdapterBlueprint.SignMessageResult>
```

## 核心适配器功能

所有适配器都实现了由类定义的一组一致的方法`AdapterBlueprint`：

### 连接管理

- `connect(params)`：与钱包提供商建立连接
- `disconnect(params)`：终止与钱包提供商的连接
- `syncConnectors(options, appKit)`：将可用连接器与当前状态同步
- `syncConnection(params)`：同步当前连接状态

### 事务操作

- `signMessage(params)`：使用连接的钱包签署消息
- `sendTransaction(params)`：向区块链发送交易
- `writeContract(params)`：执行合约方法
- `estimateGas(params)`：估算交易的 gas 成本

### 帐户信息

- `getAccounts(params)`：从连接的钱包中检索账户
- `getBalance(params)`：获取特定地址的余额
- `getProfile(params)`：检索个人资料信息（例如 ENS 数据）

### 网管

- `switchNetwork(params)`：更改连接的网络
- `getEnsAddress(params)`：将 ENS 名称解析为地址

### 多链集成示例

```ts
import { createAppKit } from '@reown/appkit';
import { EthersAdapter } from '@reown/appkit-adapter-ethers';
import { SolanaAdapter } from '@reown/appkit-adapter-solana';
import { BitcoinAdapter } from '@reown/appkit-adapter-bitcoin';
import { 
  mainnet, polygon, sepolia,
  solana, solanaDevnet,
  bitcoin, bitcoinTestnet
} from '@reown/appkit/networks';

// Create adapters for each blockchain
const ethersAdapter = new EthersAdapter({
  networks: [mainnet, polygon, sepolia],
  projectId: 'YOUR_PROJECT_ID'
});

const solanaAdapter = new SolanaAdapter({
  networks: [solana, solanaDevnet]
});

const bitcoinAdapter = new BitcoinAdapter({
  networks: [bitcoin, bitcoinTestnet]
});

// Create and configure AppKit with all adapters
const appKit = createAppKit({
  adapters: [ethersAdapter, solanaAdapter, bitcoinAdapter],
  networks: [
    mainnet, polygon, sepolia,
    solana, solanaDevnet,
    bitcoin, bitcoinTestnet
  ],
  projectId: 'YOUR_PROJECT_ID'
});
```

### 自定义RPC

```ts
const customRpcUrls = {
  'eip155:1': [{ 
    url: 'https://your-custom-mainnet-url.com',
    config: {
      fetchOptions: {
        headers: {
          'Content-Type': 'text/plain'
        }
      }
    }
  }],
  'eip155:137': [{ url: 'https://your-custom-polygon-url.com' }]
};

// Pass to both adapter and AppKit
const wagmiAdapter = new WagmiAdapter({
  networks: [mainnet, polygon],
  projectId: 'YOUR_PROJECT_ID',
  customRpcUrls
});

const appKit = createAppKit({
  adapters: [wagmiAdapter],
  networks: [mainnet, polygon],
  projectId: 'YOUR_PROJECT_ID',
  customRpcUrls
});
```

## 自定义 RPC URL

AppKit 允许覆盖网络的默认 RPC URL。这对于连接到自定义 RPC 端点或配置特定的传输选项非常有用。

```ts
const customRpcUrls = {
  'eip155:1': [{ url: 'https://your-custom-mainnet-url.com' }],
  'eip155:137': [{ url: 'https://your-custom-polygon-url.com' }]
};

const appKit = createAppKit({
  projectId: 'YOUR_PROJECT_ID',
  networks: [...],
  adapters: [...],
  customRpcUrls: customRpcUrls
});
```

您还可以传递其他传输配置选项：

```ts
const customRpcUrls = {
  'eip155:1': [{
    url: 'https://your-custom-mainnet-url.com',
    config: {
      fetchOptions: {
        headers: {
          'Content-Type': 'text/plain'
        }
      }
    }
  }]
};
```

如果您使用 Wagmi 适配器，则需要将相同的`customRpcUrls`配置传递给适配器和 AppKit：

```ts
const wagmiAdapter = new WagmiAdapter({
  networks: [...],
  projectId: "project-id",
  customRpcUrls: customRpcUrls
});

const appKit = createAppKit({
  adapters: [wagmiAdapter],
  networks: [...],
  projectId: "project-id",
  customRpcUrls: customRpcUrls
});
```

### 连接器类型顺序

自定义模式中连接选项的顺序：

```ts
const appKit = createAppKit({
  adapters: [ethersAdapter],
  networks: [mainnet],
  projectId: 'YOUR_PROJECT_ID',
  features: {
    connectorTypeOrder: ['injected', 'walletConnect', 'recent']
  }
});
```

### 命名空间特定的操作

AppKit 支持针对特定区块链命名空间的操作：

### 打开特定命名空间的模式

```ts
// Open modal for Ethereum
appKit.open({ namespace: 'eip155' });

// Open connect view for Solana
appKit.open({ view: 'Connect', namespace: 'solana' });

// Open account view for Bitcoin
appKit.open({ view: 'Account', namespace: 'bip122' });
```

资料来源：`packages/scaffold-ui/CHANGELOG.md`

### 断开特定命名空间

```ts
// Using hooks in React
const { disconnect } = useDisconnect();

// Disconnect only Solana
disconnect({ namespace: 'solana' });

// Disconnect all chains
disconnect();
```

## 高级集成

### 在 React 中使用 Hooks

AppKit 提供了 React hooks，以便于集成：

```ts
import { 
  useAppKit,
  useAppKitAccount,
  useAppKitNetwork,
  useConnect,
  useDisconnect,
  useSwitchNetwork
} from '@reown/appkit/react';

function MyComponent() {
  // Core AppKit hooks
  const { open, close } = useAppKit();
  const { address, isConnected } = useAppKitAccount();
  const { network } = useAppKitNetwork();

  // Action hooks
  const { connect, isPending: isConnecting } = useConnect();
  const { disconnect } = useDisconnect();
  const { switchNetwork } = useSwitchNetwork();
  
  // Get chain-specific account data
  const ethAccount = useAppKitAccount({ chainNamespace: 'eip155' });
  const solAccount = useAppKitAccount({ chainNamespace: 'solana' });
  const btcAccount = useAppKitAccount({ chainNamespace: 'bip122' });

  return (
    <div>
      {isConnected ? (
        <>
          <div>Connected: {address}</div>
          <div>Network: {network.name}</div>
          <button onClick={() => disconnect()}>Disconnect</button>
          <button onClick={() => switchNetwork('eip155:5')}>
            Switch to Goerli
          </button>
        </>
      ) : (
        <button onClick={() => open()} disabled={isConnecting}>
          Connect Wallet
        </button>
      )}
    </div>
  );
}
```

## 可用网络

AppKit 提供了预定义的网络配置：

```ts
// apps/laboratory/src/utils/ConstantsUtil.ts
import {
  // Ethereum networks
  mainnet, optimism, polygon, zkSync, arbitrum, base, 
  baseSepolia, sepolia, gnosis, hedera, aurora, mantle,
  
  // Solana networks
  solana, solanaDevnet, solanaTestnet,
  
  // Bitcoin networks
  bitcoin, bitcoinTestnet
} from '@reown/appkit/networks';
```

## 四、Connection连接方法

## 连接方法

AppKit 支持多种连接方式，以适应各种用户偏好和钱包类型：

### 1. WalletConnect

WalletConnect 支持通过二维码扫描或深度链接连接到移动钱包应用。该功能由 WalletConnect 的 Universal Provider 实现。

```js
// WalletConnect connection flow
async connectWalletConnect(chainId?: number | string) {
  // Authenticate with WalletConnect connector
  const walletConnectConnector = this.getWalletConnectConnector();
  await walletConnectConnector.authenticate();

  // Connect with wagmi to store the session
  const wagmiConnector = this.getWagmiConnector('walletConnect');
  const res = await connect(this.wagmiConfig, {
    connector: wagmiConnector,
    chainId: chainId ? Number(chainId) : undefined
  });

  // Switch chain if needed
  if (res.chainId !== Number(chainId)) {
    await switchChain(this.wagmiConfig, { chainId: res.chainId });
  }

  return { clientId: await walletConnectConnector.provider.client.core.crypto.getClientId() };
}
```

### 2. 注入钱包

注入式钱包（例如 MetaMask）是将提供商注入网页的浏览器扩展程序。AppKit 会检测并与这些提供商进行交互。

```js
// Injected wallet connection is added during adapter initialization
if (options.enableInjected !== false) {
  customConnectors.push(injected({ shimDisconnect: true }));
}
```

### 3. 电子邮件和社交登录

AppKit 通过 W3mFrameProvider 提供电子邮件和社交登录选项，它利用安全的 iframe 进行身份验证。

### 4. EIP-6963 浏览器钱包

AppKit 支持 EIP-6963 标准，用于检测和连接宣布其存在的浏览器钱包。

```js
// EIP-6963 event handling
private eip6963EventHandler(event: CustomEventInit<EIP6963ProviderDetail>) {
  if (event.detail) {
    const { info, provider } = event.detail;
    const existingConnector = this.connectors?.find(c => c.name === info?.name);

    if (!existingConnector) {
      this.addConnector({
        id: info?.rdns || info?.name || info?.uuid,
        type: 'ANNOUNCED',
        imageUrl: info?.icon,
        name: info?.name || 'Unknown',
        provider,
        info,
        chain: this.namespace,
        chains: []
      });
    }
  }
}
```

## 五、事件系统和状态管理

连接流依赖于事件系统来管理状态变化和组件之间的通信：

### 重要事件

1. **accountChanged**：当连接的钱包账户发生变化时触发
2. **switchNetwork**：当用户切换到不同的网络/链时触发
3. **disconnect**：钱包断开连接时触发
4. **pendingTransactions**：检测到新的待处理交易时触发

### 事件处理

每个适配器都为提供程序事件实现事件监听器，并向 AppKit 系统发出标准化事件：

```ts
private listenProviderEvents(provider: Provider | CombinedProvider) {
  const disconnect = () => {
    this.removeProviderListeners(provider);
    this.emit('disconnect');
  };

  const accountsChangedHandler = (accounts: string[]) => {
    if (accounts.length > 0) {
      this.emit('accountChanged', {
        address: accounts[0] as `0x${string}`
      });
    } else {
      disconnect();
    }
  };

  const chainChangedHandler = (chainId: string) => {
    const chainIdNumber = typeof chainId === 'string' 
      ? EthersHelpersUtil.hexStringToNumber(chainId) 
      : Number(chainId);

    this.emit('switchNetwork', { chainId: chainIdNumber });
  };

  provider.on('disconnect', disconnect);
  provider.on('accountsChanged', accountsChangedHandler);
  provider.on('chainChanged', chainChangedHandler);
}
```
