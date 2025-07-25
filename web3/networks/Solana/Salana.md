#### Salana

##### 构建Solana交互步骤

```ts
//创建您想要调用的指令。
const transferInstruction = SystemProgram.transfer({
  fromPubkey: sender.publicKey,
  toPubkey: receiver.publicKey,
  lamports: 0.01 * LAMPORTS_PER_SOL
});

//将指令添加到交易中
const transaction = new Transaction().add(transferInstruction);

//签名并发送交易
const transactionSignature = await sendAndConfirmTransaction(
  connection,
  transaction,
  [sender] // signer keypair
);
```

##### 创建代币

在此示例中，您将学习如何使用 Token Extensions Program 在 Solana 上创建一个新代币。这需要两个指令：

1. 调用 System Program 创建一个新账户。
2. 调用 Token Extensions Program 将该账户初始化为一个铸币账户。

```ts
import {
  Connection,
  Keypair,
  SystemProgram,
  Transaction,
  sendAndConfirmTransaction,
  LAMPORTS_PER_SOL
} from "@solana/web3.js";
import {
  MINT_SIZE,
  TOKEN_2022_PROGRAM_ID,
  createInitializeMint2Instruction,
  getMinimumBalanceForRentExemptMint
} from "@solana/spl-token";

// 链接到网络
const connection = new Connection("http://localhost:8899", "confirmed");

//生成一个账户
const wallet = new Keypair();
// Fund the wallet with SOL 给钱包领取空投增加余额
const signature = await connection.requestAirdrop(
  wallet.publicKey,
  LAMPORTS_PER_SOL
);
//等待线上确认
await connection.confirmTransaction(signature, "confirmed");

// Generate keypair to use as address of mint account
const mint = new Keypair();

// Calculate lamports required for rent exemption计算租金豁免所需的最低 lamports
const rentExemptionLamports =
  await getMinimumBalanceForRentExemptMint(connection);

// Instruction to create new account with space for new mint account 调用系统程序账户创建一个代币账户
const createAccountInstruction = SystemProgram.createAccount({
  fromPubkey: wallet.publicKey,
  newAccountPubkey: mint.publicKey,
  space: MINT_SIZE,
  lamports: rentExemptionLamports,
  programId: TOKEN_2022_PROGRAM_ID
});

// Instruction to initialize mint account
const initializeMintInstruction = createInitializeMint2Instruction(
  mint.publicKey,
  2, // decimals
  wallet.publicKey, // mint authority
  wallet.publicKey, // freeze authority
  TOKEN_2022_PROGRAM_ID
);

// Build transaction with instructions to create new account and initialize mint account
const transaction = new Transaction().add(
  createAccountInstruction,
  initializeMintInstruction
);

const transactionSignature = await sendAndConfirmTransaction(
  connection,
  transaction,
  [
    wallet, // payer
    mint // mint address keypair
  ]
);

console.log("Mint Account:", `${mint.publicKey}`);
console.log("Transaction Signature:", `${transactionSignature}`);
```



##### 部署操作