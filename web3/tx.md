EVM 转账交易字段：

| 字段                                | 说明                             |
| ----------------------------------- | -------------------------------- |
| nonce                               | 账户交易计数，每笔交易唯一       |
| to                                  | 接收地址                         |
| value                               | 转账金额 (wei)                   |
| gasLimit                            | 本次交易可消耗最大 gas           |
| maxFeePerGas / maxPriorityFeePerGas | EIP-1559 交易费用                |
| data                                | 调用合约数据，如果是转账可以为空 |
| chainId                             | 链ID，防重放攻击                 |
| v, r, s                             | 签名字段                         |

### 构造 raw transaction

```ts
import { ethers } from "ethers";

async function main() {
  // RPC 地址
  const RPC_URL = "https://rpc.ankr.com/eth"; // 也可以用 Infura / Alchemy
  const provider = new ethers.JsonRpcProvider(RPC_URL);

  // 私钥 (示例，用测试网)
  const PRIVATE_KEY = "0x你的私钥";
  const wallet = new ethers.Wallet(PRIVATE_KEY, provider);

  // 目标地址
  const to = "0x接收地址";

  // 查询 nonce
  const nonce = await provider.getTransactionCount(wallet.address);
  console.log("Nonce:", nonce);

  // 构造交易
  const tx = {
    to,
    value: ethers.parseEther("0.001"),
    nonce,
    gasLimit: 21000,
    maxPriorityFeePerGas: ethers.parseUnits("2", "gwei"),
    maxFeePerGas: ethers.parseUnits("50", "gwei"),
    type: 2, // EIP-1559
    chainId: 1 // 主网
  };

  // 签名
  const signedTx = await wallet.signTransaction(tx);
  console.log("Signed Tx:", signedTx);

  // 广播
  const txHash = await provider.sendTransaction(signedTx);
  console.log("Tx Hash:", txHash.hash);

  // 等待确认
  const receipt = await txHash.wait();
  console.log("Confirmed:", receipt.status);
}

main().catch(console.error);

```

