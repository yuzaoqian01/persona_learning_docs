```tsx
'use client'
import { SIWXMessenger, SIWXVerifier, DefaultSIWX, type SIWXConfig } from '@reown/appkit-siwx'
import type { SIWXMessage, SIWXSession } from '@reown/appkit-core'



// 自定义消息生成器 (CAIP-122 标准)
export class MyMessenger extends SIWXMessenger {
  protected readonly version = '1'
  
 
  /**
   * 生成 CAIP-122 格式的签名消息
   * @param params - 消息参数
   * @returns 格式化的消息字符串
   */
  protected override stringify(params: SIWXMessage.Data): string {
    const {
      accountAddress,    // 格式: "eip155:1:0x..."
      chainId,           // 格式: "eip155:1"
      domain,
      uri,
      statement,
      nonce,
      issuedAt,
      expirationTime,
      notBefore,
      requestId,
      resources
    } = params

    // 提取实际地址（CAIP-122 要求 account_address 不含 chain_id）
    const addressParts = accountAddress.split(':')
    const account_address = addressParts[addressParts.length - 1]
    
    // chain_id 使用完整的 CAIP-2 格式（如 "eip155:1"）
    const chain_id = chainId

    // 构建消息（CAIP-122 格式）
    let message = `${domain} wants you to sign in with your Ethereum account:\n`
    message += `${account_address}\n`
    message += `\n`
    
    if (statement) {
      message += `${statement}\n`
      message += `\n`
    }
    
    message += `URI: ${uri}\n`
    message += `Version: ${this.version}\n`
    
    if (nonce) {
      message += `Nonce: ${nonce}\n`
    }
    
    if (issuedAt) {
      message += `Issued At: ${issuedAt}\n`
    }
    
    if (expirationTime) {
      message += `Expiration Time: ${expirationTime}\n`
    }
    
    if (notBefore) {
      message += `Not Before: ${notBefore}\n`
    }
    
    if (requestId) {
      message += `Request ID: ${requestId}\n`
    }
    
    message += `Chain ID: ${chain_id}`

    if (resources && resources.length > 0) {
      message += `\nResources:`
      resources.forEach(resource => {
        message += `\n- ${resource}`
      })
    }

    console.log('Generated CAIP-122 message:', message)
    return message
  }
}

// 自定义验证器
export class MyVerifier extends SIWXVerifier {
  public readonly chainNamespace = 'eip155' // set the chain namespace for your verifier

  public async verify(session: SIWXSession): Promise<boolean> {
    // Implement your verification logic here
    console.log('Verifying session:', session)
    return true
  }
}

export const siwx = new DefaultSIWX({
  messenger: new MyMessenger({ 
    domain: 'FTGstake.com',
    uri: 'https://FTGstake.com',
    statement: 'Sign in with Ethereum to the FTG Stake app.',
    getNonce: async () => Math.round(Math.random() * 10000).toString()
  }),
  verifiers: [new MyVerifier()],
})


// export const siwx: SIWXConfig = {
//   createMessage: async (input: SIWXMessage.Input): Promise<SIWXMessage> => {
//     // Implement your logic to create a message
//     return 'my message'
//   },
//   addSession: async (session) => {
//     // Implement your logic to add a session
//   },
//   revokeSession: async (chainId, address) => {
//     // Implement your logic to revoke a session
//   },
//   setSessions: async (sessions) => {
//     // Implement your logic to set sessions
//   },
//   getSessions: async (chainId, address) => {
//     // Implement your logic to get sessions
//     return []
//   },
//   getRequired: () => {
//     // Return whether the wallet should stay connected when user denies signature
//     return false
//   },
//   signOutOnDisconnect: true // Whether to clear sessions when user disconnects
// }


```

```tsx
import { type CaipNetworkId, type SIWXConfig, type SIWXMessage, type SIWXSession, } from '@reown/appkit'

import * as siwxAPI from '~/API/siwx'
import request from '~/lib/api'
import { useAuthStore } from '~/store/use-auth-store'

export const siwx: SIWXConfig = {
  createMessage: async (input: SIWXMessage.Input) => {
    try {
      console.log('===== createMessage 开始 =====');
      console.log('输入参数:', input);
      
      // 检查是否已授权，获取临时 UID
      let { temp_uid, callback } = await siwxAPI.checkAuth();
      console.log('获取到 temp_uid:', temp_uid);
      
      // 后端创建用户记录
      let params = {
        uuID: temp_uid,
        actionId: temp_uid,
        account: input.accountAddress,
        wallet: input.accountAddress,
        action: "login",
        from: "FTGstake",
        consensus: ''
      };
      await request.post(callback, params)
      console.log('后端用户记录创建成功');
      
      // ==========================================
      // 从后端获取消息（用户将签名这个消息）
      // ==========================================
      let {msg} = await siwxAPI.generateMessage({
        'wallet': input.accountAddress,
        chainId: input.chainId,
      });
      console.log('后端返回的消息:', msg);
      
      // 保存临时数据（用于 addSession 时验证）
      localStorage.setItem('callback', callback);
      localStorage.setItem('temp_uid', temp_uid);
      localStorage.setItem('chainId', input.chainId);
      localStorage.setItem('accountAddress', input.accountAddress);
      
      // ==========================================
      // 将后端消息包装成符合 SIWXMessage 接口的对象
      // 注意：后端返回的是加密的 JWT 字符串，无法提取元数据
      // ==========================================
      
      // 1. 提取纯地址（去掉 CAIP-10 前缀）
      const addressParts = input.accountAddress.split(':');
      const pureAddress = addressParts[addressParts.length - 1];
      
      // 2. 确保 msg 是字符串（可能是 JWT 字符串）
      const messageStr = typeof msg === 'string' ? msg : JSON.stringify(msg);
      console.log('后端消息类型:', typeof msg);
      console.log('后端消息内容:', messageStr);
      
      // 3. 由于后端返回的是 JWT 字符串，直接使用默认值
      // 这些元数据仅用于前端 Session 管理，不影响后端验证
      const nonce = temp_uid;  // 使用 temp_uid 作为 nonce
      const issuedAt = new Date().toISOString();
      const expirationTime = new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString(); // 24小时后过期
    //   const expirationTime = new Date(Date.now() + 10 * 1000).toISOString(); // 10秒后过期

      const domain = import.meta.env.VITE_APP_DOMAIN || 'FTGstake.com';
      const uri = import.meta.env.VITE_APP_URI || (typeof window !== 'undefined' ? window.location.origin : 'https://FTGstake.com');
      const version = '1';
      const statement = 'Sign in with Ethereum to the FTG Stake app.';
      
      // 4. 构造符合 SIWXMessage 接口的对象
      // 关键：toString() 返回后端的 JWT 字符串，用户签名这个字符串
      const siwxMessage: SIWXMessage = {
        // Data 部分 - 必填字段
        accountAddress: input.accountAddress,
        chainId: input.chainId,
        domain,
        uri,
        version,
        nonce,
        
        // Data 部分 - 可选字段（使用默认值）
        statement,
        issuedAt,
        expirationTime,
        notBefore: input.notBefore,
        requestId: temp_uid,
        resources: undefined,
        
        // Methods 部分 - 返回后端的 JWT 字符串
        // 用户将签名这个 JWT 字符串，后端验证时能通过
        toString: () => messageStr
      };
      
      console.log('包装后的 SIWXMessage 对象:', {
        accountAddress: siwxMessage.accountAddress,
        chainId: siwxMessage.chainId,
        domain: siwxMessage.domain,
        nonce: siwxMessage.nonce,
        expirationTime: siwxMessage.expirationTime,
        toString: '[Function]'
      });
      console.log('用户将签名这个 JWT 字符串:', messageStr);
      console.log('===== createMessage 完成 =====');
      
      return siwxMessage;
    } catch (error) {
      console.error('❌ createMessage 错误:', error);
      // 清理临时数据
      localStorage.removeItem('callback');
      localStorage.removeItem('temp_uid');
      localStorage.removeItem('chainId');
      localStorage.removeItem('accountAddress');
      throw error;
    }
  },
  // 添加会话
  addSession: async (session: SIWXSession) => {
    console.log('===== addSession 开始 =====');
    console.log('Session:', session);
    
    let temp_uid = localStorage.getItem('temp_uid');
    let chainId = localStorage.getItem('chainId');
    let accountAddress = localStorage.getItem('accountAddress');
    
    // 验证签名并创建 session
    let res = await siwxAPI.createSession({
        uuid: temp_uid as string,
        message: session.message,
        wallet: accountAddress as string,
        signature: session.signature,
        chainId: chainId as string | number,
    })
    console.log('后端验证通过:', res);
    
    // 保存 session 信息到 localStorage（不包含 toString 方法）
    const sessionData = {
      data: session.data,
      message: session.message,
      signature: session.signature,
      cacao: session.cacao
    };
    localStorage.setItem('siwx_session', JSON.stringify(sessionData));
    
    // 使用 AuthStore 保存认证信息
    const { login } = useAuthStore.getState();
    login(res.token, res.user, accountAddress as string, Number(chainId));
    
    // 清理临时数据
    localStorage.removeItem('temp_uid');
    localStorage.removeItem('chainId');
    localStorage.removeItem('callback');
    
    console.log('✅ Session 保存成功');
    console.log('===== addSession 完成 =====');
  },
  // 撤销会话
  revokeSession: async (chainId: CaipNetworkId, address: string) => {
    console.log('===== revokeSession 开始 =====');
    console.log('撤销会话:', { chainId, address });
    
    await siwxAPI.logout();
    
    // 清理 AuthStore 状态
    const { logout } = useAuthStore.getState();
    logout();
    
    localStorage.removeItem('siwx_session');
    
    console.log('✅ 会话已撤销，localStorage 已清理');
    console.log('===== revokeSession 完成 =====');
  },
  // 设置会话
  setSessions: async (sessions: SIWXSession[]) => {
    console.log('===== setSessions 开始 =====');
    console.log('设置会话:', sessions);
    
    // if(localStorage.getItem('token')){
    //   await siwxAPI.logout();
    // }
    
    // 清理 AuthStore 状态
    const { logout } = useAuthStore.getState();
    logout();
    
    localStorage.removeItem('siwx_session');
    
    console.log('✅ 会话已清理');
    console.log('===== setSessions 完成 =====');
  },
  // 获取会话
  getSessions: async (chainId: CaipNetworkId, address: string) => {
    console.log('===== getSessions 开始 =====');
    console.log('查询参数:', { chainId, address });
    
    // 从 localStorage 获取保存的 session
    const sessionStr = localStorage.getItem('siwx_session');
    
    // 从 AuthStore 获取 token
    const { token } = useAuthStore.getState();
    
    // 如果没有 session 或 token，返回空数组
    if (!sessionStr || !token) {
      console.log('没有找到 session 或 token，返回空数组');
      return [] as SIWXSession[]
    }
    
    try {
      const savedSession = JSON.parse(sessionStr);
      console.log('从 localStorage 恢复的 session:', savedSession);
      
      // 验证 session 数据完整性
      if (!savedSession.data || !savedSession.message || !savedSession.signature) {
        console.error('❌ Session 数据不完整');
        localStorage.removeItem('siwx_session');
        return [] as SIWXSession[]
      }
      
      // 验证 session 是否匹配当前的 chainId 和 address
      if (savedSession.data.chainId !== chainId || 
          savedSession.data.accountAddress.toLowerCase() !== address.toLowerCase()) {
        console.log('❌ Session 的 chainId 或 address 不匹配');
        console.log('  期望:', { chainId, address });
        console.log('  实际:', { 
          chainId: savedSession.data.chainId, 
          address: savedSession.data.accountAddress 
        });
        return [] as SIWXSession[]
      }
      
      // 检查 session 是否过期
      if (savedSession.data.expirationTime) {
        const expirationTime = new Date(savedSession.data.expirationTime).getTime();
        const currentTime = Date.now();
        if (currentTime >= expirationTime) {
          console.log('❌ Session 已过期');
          // Session 已过期，清理数据
          localStorage.removeItem('siwx_session');
          
          // 清理 AuthStore 状态
          const { logout } = useAuthStore.getState();
          logout();
          return [] as SIWXSession[]
        }
      }
      
      // 重新构造完整的 SIWXSession 对象
      // 注意：JSON.parse 会丢失方法，需要手动恢复
      const restoredSession: SIWXSession = {
        data: savedSession.data,
        message: savedSession.message,
        signature: savedSession.signature,
        cacao: savedSession.cacao
      };
      
      console.log('✅ Session 有效，返回恢复的 session');
      console.log('===== getSessions 完成 =====');
      return [restoredSession]
    } catch (error) {
      console.error('❌ 解析 session 数据失败:', error);
      // 清理损坏的数据
      localStorage.removeItem('siwx_session');
      return [] as SIWXSession[]
    }
  },
  getRequired: () => {
    console.log('getRequired');
    return true
  },
  signOutOnDisconnect: true
}
```

