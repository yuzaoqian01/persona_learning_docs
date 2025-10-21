# Appkit 的熟悉与使用

## 一、安装

```cmd
pnpm add @reown/appkit @reown/appkit-universal-connector @reown/appkit-common ethers
```

为了快速集成 Appkit Core，您可以使用 `UniversalConnector` 类。它通过为所有区块链协议提供单一接口来简化 Appkit Core 的集成。

您可以使用要支持的网络配置通用连接器。有关更多信息，请访问我们文档中的 [RPC 参考](https://docs.reown.com/advanced/multichain/rpc-reference/cosmos-rpc)部分。

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

### 视图控制

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

