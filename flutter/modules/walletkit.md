# walletkit的连接与使用

### 1.初始化

```dart
final _walletKit = ReownWalletKit(
  core: ReownCore(
    projectId: '{YOUR_PROJECT_ID}',
  ),
  metadata: PairingMetadata(
    name: 'Example Wallet',
    description: 'Example wallet description',
    url: 'https://example.com/',
    icons: ['https://example.com/logo.png'],
    redirect: Redirect(
      native: 'examplewallet://',
      universal: 'https://reown.com/examplewallet',
    ),
  ),
);
```

### 2.注册钱包 事件 和方法

```dart
// Quick example:

List<String> supportedChains = ['eip155:1', 'eip155:10', ...];
List<String> walletAddresses = ['0x1234......'];
List<String> supportedEvents = ['chainChanged', 'accountsChanged', ...];

Map<String, dynamic Function(String, dynamic)> get _methodHandlers => {
  'personal_sign': personalSignHandler,
  'eth_sendTransaction': ethSendTransactionHandler,
};

for (final chainId in supportedChains) {
  for (var address in walletAddresses) {
    _walletKit!.registerAccount(
      chainId: chainId, // CAIP-2 format chain id
      accountAddress: address, // 0x.... address
    );
  }

  for (var handler in _methodHandlers.entries) {
    _walletKit.registerRequestHandler(
      chainId: chainId,
      method: handler.key,
      handler: handler.value,
    );
  }

  for (final event in supportedEvents) {
    _walletKit.registerEventEmitter(
      chainId: chainId,
      event: event,
    );
  }
}
```

### 3.EVM methods & events EVM 方法和事件

```dart
{
  //...
  methods: [
    "eth_accounts",
    "eth_requestAccounts",
    "eth_sendRawTransaction",
    "eth_sign",
    "eth_signTransaction",
    "eth_signTypedData",
    "eth_signTypedData_v3",
    "eth_signTypedData_v4",
    "eth_sendTransaction",
    "personal_sign",
    "wallet_switchEthereumChain",
    "wallet_addEthereumChain",
    "wallet_getPermissions",
    "wallet_requestPermissions",
    "wallet_registerOnboarding",
    "wallet_watchAsset",
    "wallet_scanQRCode",
    "wallet_sendCalls",
    "wallet_getCallsStatus",
    "wallet_showCallsStatus",
    "wallet_getCapabilities",
  ],
  events: [
    "chainChanged",
    "accountsChanged",
    "message",
    "disconnect",
    "connect",
  ]
}
```



#### 4.dapp钱包连接钱包会触发SessionProposalEvent事件

```json
// 会话连接申请SessionProposalEvent
{
    "id": 1758174038438174,
    "params": {
        "id": 1758174038438174,
        "expiry": 1758174339,
        "relays": [
            {
                "protocol": "irn",
                "data": null
            }
        ],
        "proposer": {
            "publicKey": "b21be66229a7e19438414d050e72bd113499b0f8afa958a5db269a6521284746",
            "metadata": {
                "name": "AppKit Lab",
                "description": "AppKit Lab is the test environment for Reown's AppKit",
                "url": "https://appkit-lab.reown.com",
                "icons": [
                    "https://appkit-lab.reown.com/logo.png"
                ]
            }
        },
        "requiredNamespaces": {

        },
        "optionalNamespaces": {
            "eip155": {
                "chains": [
                    "eip155:1",
                    "eip155:10",
                    "eip155:137",
                    "eip155:324",
                    "eip155:42161",
                    "eip155:8453",
                    "eip155:84532",
                    "eip155:1301",
                    "eip155:80094",
                    "eip155:11155111",
                    "eip155:100",
                    "eip155:295",
                    "eip155:1313161554",
                    "eip155:5000",
                    "eip155:2741",
                    "eip155:10143"
                ],
                "methods": [
                    "eth_accounts",
                    "eth_requestAccounts",
                    "eth_sendRawTransaction ",
                    "eth_sign",
                    "eth_signTransaction",
                    "eth_signTypedData",
                    "eth_signTypedData_v3",
                    "eth_signTypedData_v4",
                    "eth_sendTransaction",
                    "personal_sign",
                    "wallet_switchEthereumChain",
                    "wallet_addEthereumChain",
                    "wallet_getPermissions",
                    "wallet_requestPermissions",
                    "wallet_registerOnboarding",
                    "wallet_watchAsset",
                    "wallet_scanQRCode",
                    "wallet_getCallsStatus",
                    "wallet_showCallsStatus",
                    "wallet_sendCalls",
                    "wallet_getCapabilities",
                    "wallet_grantPermissions",
                    "wallet_revokePermissions",
                    "wallet_getAssets"
                ],
                "events": [
                    "accountsChanged",
                    "chainChanged"
                ]
            }
        },
        "pairingTopic": "b1057bc02b11174c9a94ee145876dbf21a5870f3a1958985791b0ff52b711168",
        "generatedNamespaces": {
            "eip155": {
                "chains": [
                    "eip155:1",
                    "eip155:10",
                    "eip155:137",
                    "eip155:42161",
                    "eip155:8453"
                ],
                "accounts": [
                    "eip155:1:0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                    "eip155:10:0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                    "eip155:137:0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                    "eip155:42161:0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                    "eip155:8453:0x00002F528A2256dBB25Cd57D0F1bC6dade192222"
                ],
                "methods": [
                    "eth_accounts",
                    "eth_requestAccounts",
                    "eth_sendRawTransaction",
                    "eth_sign",
                    "eth_signTransaction",
                    "eth_signTypedData",
                    "eth_signTypedData_v3",
                    "eth_signTypedData_v4",
                    "eth_sendTransaction",
                    "personal_sign",
                    "wallet_switchEthereumChain",
                    "wallet_addEthereumChain",
                    "wallet_getPermissions",
                    "wallet_requestPermissions",
                    "wallet_registerOnboarding",
                    "wallet_watchAsset",
                    "wallet_scanQRCode",
                    "wallet_sendCalls",
                    "wallet_getCallsStatus",
                    "wallet_showCallsStatus",
                    "wallet_getCapabilities"
                ],
                "events": [
                    "chainChanged",
                    "accountsChanged"
                ]
            }
        }
    },
    "verifyContext": {
        "origin": "https://appkit-lab.reown.com",
        "validation": "VALID",
        "verifyUrl": "https://verify.walletconnect.org",
        "isScam": false
    }
}
```

