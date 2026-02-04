### 1. 安装 Web3Auth React Native SDK

```cmd
pnpm add @web3auth/react-native-sdk
```



Ps: 创建项目后需要`npx expo prebuild`

### 2. 安装辅助模块

安装 WebBrowser 和 Storage 所需的辅助模块：

- 我们需要实现一个`WebBrowser`功能，使我们基于 JS 的 SDK 能够与原生 API 进行交互。
- 需要实现一种`Storage`存储用户会话的功能，但不能将用户的私钥存储在设备中

```cmd
pnpm add expo-web-browser expo-secure-store
```

### Expo 管理工作流程

- 请更新文件中的方案`app.json`。

  app.json

  ```json
  {
    "expo": {
      "scheme": "web3authexpoexample" // replace with your own scheme
    }
  }
  ```

- 要构建您的应用程序`redirectUrl`，您需要使用 Web3Auth 工具`expo-linking`，它将帮助您为您的应用程序生成一个 URL 。请确保在[Web3Auth 开发者控制面板](https://dashboard.web3auth.io/)`redirectUrl`中注册该 URL 。

  ```typescript
  import Constants, { AppOwnership } from 'expo-constants'
  import * as Linking from 'expo-linking'
  
  const resolvedRedirectUrl =
    Constants.appOwnership == AppOwnership.Expo || Constants.appOwnership == AppOwnership.Guest
      ? Linking.createURL('web3auth', {})
      : Linking.createURL('web3auth', { scheme: 'web3authexpoexample' }) // replace with your own scheme
  ```

  ### 2. 初始化 Web3Auth 实例

  在使用任何身份验证方法之前，请先初始化 Web3Auth 实例：

  App.tsx

  ```typescript
  import { useEffect } from 'react'
  
  useEffect(() => {
    const init = async () => {
      try {
        await web3auth.init()
        console.log('Web3Auth initialized successfully')
      } catch (error) {
        console.error('Error initializing Web3Auth:', error)
      }
    }
    init()
  }, [])
  ```

  Auth.ts

```ts
// 必须最先导入 Buffer polyfill
import { Buffer } from "buffer";
if (typeof global.Buffer === "undefined") {
  global.Buffer = Buffer;
}

import { CHAIN_NAMESPACES, WEB3AUTH_NETWORK } from "@web3auth/base";
import { EthereumPrivateKeyProvider } from "@web3auth/ethereum-provider";
import Web3Auth, { LOGIN_PROVIDER } from "@web3auth/react-native-sdk";
import { ethers } from "ethers";
import * as SecureStore from "expo-secure-store";
import * as WebBrowser from "expo-web-browser";
import EncryptedStorage from "react-native-encrypted-storage";

// Web3Auth 配置
const WEB3AUTH_CLIENT_ID =
  process.env.WEB3AUTH_CLIENT_ID ||
  "BH6WxH2Dffa-mQB_w2iRxqPfXTnUUMeeluGlYy6ucW-rvTDwD2Bk2JbptHE1RUq9DYf-ZWmbJkp1owe12iP4nng";

const REDIRECT_URL = "bitwallet://auth";
const SESSION_KEY = "web3auth_session";

// Web3Auth 网络配置（默认使用 Sapphire Mainnet）
const WEB3AUTH_NETWORK_TYPE = WEB3AUTH_NETWORK.SAPPHIRE_DEVNET;

export interface UserInfo {
  name: string;
  email: string;
  profileImage?: string;
  address: string;
  provider: string;
}

export type LoginProvider = "google" | "email" | "apple" | "twitter";

// 映射自定义提供商到 Web3Auth LOGIN_PROVIDER
const PROVIDER_MAP: Record<
  LoginProvider,
  (typeof LOGIN_PROVIDER)[keyof typeof LOGIN_PROVIDER]
> = {
  google: LOGIN_PROVIDER.GOOGLE,
  email: LOGIN_PROVIDER.EMAIL_PASSWORDLESS,
  apple: LOGIN_PROVIDER.APPLE,
  twitter: LOGIN_PROVIDER.TWITTER,
};

class Web3AuthService {
  private web3auth: Web3Auth | null = null;
  private isInitialized = false;
  private currentUser: UserInfo | null = null;

  /**
   * 初始化 Web3Auth
   */
  async init(): Promise<void> {
    if (this.isInitialized) return;

    try {
      // 创建以太坊私钥提供商（使用免费公共 RPC）
      const chainConfig = {
        chainNamespace: CHAIN_NAMESPACES.EIP155,
        chainId: "0x1", // 以太坊主网
        rpcTarget: "https://eth.llamarpc.com", // 使用免费的公共 RPC
      };

      const privateKeyProvider = new EthereumPrivateKeyProvider({
        config: { chainConfig },
      });

      // 创建 Web3Auth 实例
      this.web3auth = new Web3Auth(WebBrowser, EncryptedStorage, {
        clientId: WEB3AUTH_CLIENT_ID,//客户端id
        network: WEB3AUTH_NETWORK_TYPE,//web3auth的网络
        redirectUrl: REDIRECT_URL,//应用shecme
        privateKeyProvider,//provider
      });

      // 初始化 Web3Auth SDK
      await this.web3auth.init();

      // 检查是否有存储的会话
      const session = await this.getStoredSession();
      if (session) {
        this.currentUser = session;
      }

      this.isInitialized = true;
      console.log("✅ Web3Auth initialized with real SDK");
    } catch (error) {
      console.error("Failed to initialize Web3Auth:", error);
      throw error;
    }
  }

  /**
   * 使用指定提供商登录
   */
  async login(provider: LoginProvider): Promise<UserInfo> {
    if (!this.isInitialized || !this.web3auth) {
      await this.init();
    }

    try {
      // 使用 Web3Auth SDK 进行登录
      const loginProvider = PROVIDER_MAP[provider];
      console.log(`🔐 使用 Web3Auth SDK 登录: ${provider}`);

      const web3authProvider = await this.web3auth!.login({
        loginProvider,
        redirectUrl: REDIRECT_URL,
      });

      if (!web3authProvider) {
        throw new Error("登录失败：未获取到认证提供商");
      }

      // 获取用户信息
      const web3authUserInfo = this.web3auth!.userInfo();

      // 获取私钥（Web3Auth 管理的安全私钥）
      const privateKey = await web3authProvider.request({
        method: "eth_private_key",
      });

      if (!privateKey || typeof privateKey !== "string") {
        throw new Error("获取私钥失败");
      }

      // 使用真实私钥创建钱包
      const wallet = new ethers.Wallet(privateKey);

      // 构建用户信息
      const userInfo: UserInfo = {
        name: web3authUserInfo?.name || "User",
        email: web3authUserInfo?.email || "",
        profileImage: web3authUserInfo?.profileImage,
        address: wallet.address,
        provider,
      };

      // 存储会话
      await this.storeSession(userInfo);
      this.currentUser = userInfo;

      console.log("✅ Web3Auth 登录成功:", userInfo.address);
      return userInfo;
    } catch (error) {
      console.error("Login error:", error);
      throw error;
    }
  }

  /**
   * 登出
   */
  async logout(): Promise<void> {
    try {
      // 使用 Web3Auth SDK 登出
      if (this.web3auth) {
        await this.web3auth.logout();
      }

      // 清除存储的会话
      await SecureStore.deleteItemAsync(SESSION_KEY);
      this.currentUser = null;
      console.log("✅ 登出成功");
    } catch (error) {
      console.error("Logout error:", error);
      throw error;
    }
  }

  /**
   * 获取当前用户信息
   */
  getUserInfo(): UserInfo | null {
    return this.currentUser;
  }

  /**
   * 检查是否已认证
   */
  isAuthenticated(): boolean {
    return this.currentUser !== null;
  }

  /**
   * 存储会话
   */
  private async storeSession(userInfo: UserInfo): Promise<void> {
    try {
      await SecureStore.setItemAsync(SESSION_KEY, JSON.stringify(userInfo));
    } catch (error) {
      console.error("Failed to store session:", error);
      throw error;
    }
  }

  /**
   * 获取存储的会话
   */
  private async getStoredSession(): Promise<UserInfo | null> {
    try {
      const session = await SecureStore.getItemAsync(SESSION_KEY);
      if (session) {
        return JSON.parse(session);
      }
      return null;
    } catch (error) {
      console.error("Failed to get stored session:", error);
      return null;
    }
  }
}

// 导出单例
export const web3AuthService = new Web3AuthService();

```

```js
// metro.config.js
// Learn more https://docs.expo.io/guides/customizing-metro
const { getDefaultConfig } = require("expo/metro-config");

/** @type {import('expo/metro-config').MetroConfig} */
const config = getDefaultConfig(__dirname);

config.resolver.extraNodeModules = {
  crypto: require.resolve("crypto-browserify"),
  stream: require.resolve("readable-stream"),
  buffer: require.resolve("buffer"),
  process: require.resolve("process"),
};

config.transformer.getTransformOptions = () => ({
  transform: {
    experimentalImportSupport: false,
    inlineRequires: true,
  },
});

module.exports = config;

```

```ts
import Web3Auth, { WEB3AUTH_NETWORK, MFA_LEVELS } from '@web3auth/react-native-sdk' // Web3Auth RN SDK（MPC + OAuth + MFA）
import * as SecureStore from 'expo-secure-store' // iOS Keychain / Android Keystore（存储 device key share）
import * as WebBrowser from 'expo-web-browser' // 系统级浏览器，避免 WebView 被注入

const web3auth = new Web3Auth(
  WebBrowser, // 使用系统浏览器进行 OAuth，防止中间人攻击（生产环境必须）
  SecureStore, // MPC 的 device share 仅存放在安全硬件中，不可用 AsyncStorage
  {
    clientId: process.env.WEB3AUTH_CLIENT_ID!, // Web3Auth 项目 ID（❗必须走环境变量，禁止硬编码）

    network: WEB3AUTH_NETWORK.SAPPHIRE_MAINNET, // 生产环境使用 Sapphire Mainnet（Dev/Test 禁止上线）

    redirectUrl: process.env.
    !, // OAuth 回调地址（需与 Dashboard + OAuth Provider 完全一致）

    mfaLevel: MFA_LEVELS.MANDATORY, // 强制 MFA：防止单点设备丢失导致资产不可恢复

    loginConfig: {
      google: {
        verifier: 'google-verifier', // Dashboard 中配置的 verifier 名称（生产/测试必须区分）
        typeOfLogin: TypeOfLogin.GOOGLE, // Google OAuth 登录方式
        clientId: process.env.GOOGLE_CLIENT_ID!, // Google OAuth Client ID（❗必须走 env）
      },

      jwt: {
        verifier: 'jwt-verifier', // 自定义登录 verifier（通常绑定你自己的 Auth Server）
        typeOfLogin: TypeOfLogin.JWT, // JWT 登录（企业级 / 托管钱包核心方案）
        clientId: process.env.JWT_CLIENT_ID!, // JWT clientId（与 Auth Server 签发逻辑强绑定）
      },
    },
  }
)

```

### `Web3AuthOptions`

| 范围           | 描述                                                         |
| :------------- | :----------------------------------------------------------- |
| `clientId`     | [您在控制面板](https://dashboard.web3auth.io/)中看到的 Web3Auth 客户端 ID 。这是一个必填字段`string`。 |
| `network`      | Web3Auth 网络：`SAPPHIRE_MAINNET`，，，，或。必填字段`SAPPHIRE_DEVNET`，类型为。`MAINNET``CYAN``AQUA``TESTNET``WEB3AUTH_NETWORK` |
| `redirectUrl`  | Web3Auth 在身份验证成功后会将 API 响应重定向到此 URL。这是一个必填字段，类型为`string`. |
| `whiteLabel?`  | 提供自定义用户界面、品牌和翻译的白标选项。`WhiteLabelData`以值的形式呈现。 |
| `loginConfig?` | 自定义验证器的登录配置。接受`LoginConfig`一个值。            |
| `mfaLevel?`    | 配置多因素身份验证 (MFA) 级别。接受`MFA_LEVELS`一个值。      |
| `sessionTime?` | 配置会话管理时间，单位为秒。默认值为 86400 秒（1 天）。最长 30 天。 |