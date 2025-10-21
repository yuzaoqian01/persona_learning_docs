

### 一. 初始化

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

### 二. 注册钱包事件

```dart
// Quick example:

List<String> supportedChains = ['eip155:1', 'eip155:10', ...];
List<String> walletAddresses = ['0x1234......'];
List<String> supportedEvents = ['chainChanged', 'accountsChanged', ...];

Map<String, dynamic Function(String, dynamic)> get _methodHandlers => {
  'personal_sign': personalSignHandler,
  'eth_sendTransaction': ethSendTransactionHandler,
};

// 注册每个网络的钱包地址
for (final chainId in supportedChains) {
  for (var address in walletAddresses) {
    _walletKit!.registerAccount(
      chainId: chainId, // CAIP-2 format chain id
      accountAddress: address, // 0x.... address
    );
  }

  // 事件处理器
  for (var handler in _methodHandlers.entries) {
    _walletKit.registerRequestHandler(
      chainId: chainId,
      method: handler.key,
      handler: handler.value,
    );
  }

  // 注册事件
  for (final event in supportedEvents) {
    _walletKit.registerEventEmitter(
      chainId: chainId,
      event: event,
    );
  }
}
```

### 三. EVM支持的事件

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

四. dap p 会话处理与批准

```dart
_walletKit.onSessionProposal.subscribe((SessionProposalEvent? event) {
  // display a prompt for the user to approve or reject the session
  // ....
  // If approved
  _walletKit.approveSession(
    id: event.id,
    namespaces: event.params.generatedNamespaces ?? {},
  );
});

// 拒绝回话
_walletKit.onSessionProposal.subscribe((SessionProposalEvent? event) async {
  // display a prompt for the user to approve or reject the session
  // ....
  // If rejected
  await _walletKit.rejectSession(
    id: event.id,
    reason: Errors.getSdkError(Errors.USER_REJECTED).toSignError(),
  );
});


final supportedChains = ['eip155:1', 'eip155:137'];
Map<String, dynamic Function(String, dynamic)> supportedMethods = {
  'personal_sign': _personalSignHandler,
  'eth_sendTransaction': _ethSendTransactionHandler,
};
// Register your handlers as stated in Namespace Builder section
for (var chainId in supportedChains) {
  for (var method in supportedMethods.entries) {
    _walletKit.registerRequestHandler(
      chainId: chainId,
      method: method.key,
      handler: method.value,
    );
  }
}

Future<void> _personalSignHandler(String topic, dynamic params) async {
  final SessionRequest pendingRequest = _walletKit.pendingRequests.getAll().last;
  final int requestId = pendingRequest.id;

  // message should arrive encoded
  final decoded = hex.decode(params.first.substring(2));
  final message = utf8.decode(decoded);

  // display a prompt for the user to approve or reject the request
  // if approved
  if (approved) {
    // Your code to sign the message here
    final signature = await signMessage(message);

    return _walletKit.respondSessionRequest(
      topic: topic,
      response: JsonRpcResponse(
        id: requestId,
        jsonrpc: '2.0',
        result: signature,
      ),
    );
  }

  // if rejected
  return _walletKit.respondSessionRequest(
    topic: topic,
    response: JsonRpcResponse(
      id: id,
      jsonrpc: '2.0',
      error: const JsonRpcError(code: 5001, message: 'User rejected method'),
    ),
  );
}

Future<void> _ethSendTransactionHandler(String topic, dynamic params) async {
  final SessionRequest pendingRequest = _walletKit.pendingRequests.getAll().last;
  final int requestId = pendingRequest.id;
  final String chainId = pendingRequest.chainId;

  final transaction = (params as List<dynamic>).first as Map<String, dynamic>;

  // display a prompt for the user to approve or reject the request
  // if approved
  if (approved) {
    final signedTx = await sendTransaction(transaction, int.parse(chainId));
    // respond to requester
    await _walletKit.respondSessionRequest(
      topic: topic,
      response: JsonRpcResponse(
        id: requestId, 
        jsonrpc: '2.0', 
        result: signedTx,
      ),
    );
  }

  // if rejected
  return _walletKit.respondSessionRequest(
    topic: topic,
    response: JsonRpcResponse(
      id: id,
      jsonrpc: '2.0',
      error: const JsonRpcError(code: 5001, message: 'User rejected method'),
    ),
  );
}
```

五. 扫描并配对Dap p

```dart
Uri uri = Uri.parse(scannedUriString);
await _walletKit.pair(uri: uri);
```

```json
// 会话提案
{
    "id": 1758354512190835,
    "params": {
        "id": 1758354512190835,
        "expiry": 1758354813,
        "relays": [
            {
                "protocol": "irn",
                "data": null
            }
        ],
        "proposer": {
            "publicKey": "87de8ad71e20cac3558e25833d136577c348a82d52d480b3452b18be89c9812f",
            "metadata": {
                "name": "AppKit",
                "description": "AppKit Example",
                "url": "https://dapp-tools.pages.dev",
                "icons": [
                    "https://avatars.githubusercontent.com/u/179229932"
                ]
            }
        },
        "requiredNamespaces": {},
        "optionalNamespaces": {
            "eip155": {
                "chains": [
                    "eip155:1",
                    "eip155:42161",
                    "eip155:8453",
                    "eip155:56",
                    "eip155:137",
                    "eip155:11155111"
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
            },
            "solana": {
                "chains": [
                    "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp",
                    "solana: 4sGjMW1sUnHzSxGspuhpqLDx6wiyjNtZ",
                    "solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1",
                    "solana: 8E9rvCKLFQia2Y35HXjjpWzj8weVo44K"
                ],
                "methods": [
                    "solana_signMessage",
                    "solana_signTransaction",
                    "solana_requestAccounts",
                    "solana_getAccounts",
                    "solana_signAllTransactions",
                    "solana_signAndSendTransaction"
                ],
                "events": [
                    "accountsChanged",
                    "chainChanged"
                ]
            },
            "bip122": {
                "chains": [
                    "bip122: 000000000019d6689c085ae165831e93"
                ],
                "methods": [
                    "sendTransfer",
                    "signMessage",
                    "signPsbt",
                    "getAccountAddresses"
                ],
                "events": [
                    "accountsChanged",
                    "chainChanged"
                ]
            }
        },
        "pairingTopic": "769eb1842bafbd35ed1e6708550df8156d087133833dac6a14d4bc6bda2b8626",
        "generatedNamespaces": {
            "eip155": {
                "chains": [
                    "eip155: 1",
                    "eip155: 42161",
                    "eip155: 8453",
                    "eip155: 56",
                    "eip155: 137"
                ],
                "accounts": [
                    "eip155: 1: 0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                    "eip155: 42161: 0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                    "eip155: 8453: 0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                    "eip155: 56: 0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                    "eip155: 137: 0x00002F528A2256dBB25Cd57D0F1bC6dade192222"
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
        "origin": "https: //dapp-tools.pages.dev",
        "validation": "VALID",
        "verifyUrl": "https://verify.walletconnect.org",
        "isScam": false
    }
}
```

```json
/// 会话连接
{
    "topic": "f4662dfc87eaa462442110ef651fa7dce292689406c4bdb0dfde4032d4dc92f1",
    "pairingTopic": "0ae732766267073ec8c7b08552ef2e83e182cc8b1aa107567b1f5c70a9ef154d",
    "relay": "Instance of 'Relay'",
    "expiry": 1758960166,
    "acknowledged": false,
    "controller": "a95c39555cde43b1fa6758c27428fc9e25a0d611c0677f4cad29ff156a067e20",
    "namespaces": {
        "eip155": {
            "chains": [
                "eip155:1",
                "eip155:42161",
                "eip155:8453",
                "eip155:56",
                "eip155:137"
            ],
            "accounts": {
                "eip155:1": "0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                "eip155:42161": "0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                "eip155:8453": "0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                "eip155:56": "0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
                "eip155:137": "0x00002F528A2256dBB25Cd57D0F1bC6dade192222"
            },
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
    },
    "self": {
        "publicKey": "a95c39555cde43b1fa6758c27428fc9e25a0d611c0677f4cad29ff156a067e20",
        "metadata": {
            "name": "HashLink Wallet",
            "description": "HashLink wallet description",
            "url": "https://www.finx.com/",
            "icons": [
                "https://upload.wikimedia.org/wikipedia/commons/1/17/Google-flutter-logo.png"
            ],
            "verifyUrl": null,
            "redirect": {
                "native": "finx://",
                "universal": "https://finx.com",
                "linkMode": true
            }
        }
    },
    "peer": {
        "publicKey": "273d1a60787e11e29a7d101aad4b8f65a900bc409e181ae28ad6f2e267ebba10",
        "metadata": {
            "name": "AppKit",
            "description": "AppKit Example",
            "url": "https://dapp-tools.pages.dev",
            "icons": [
                "https://avatars.githubusercontent.com/u/179229932"
            ],
            "verifyUrl": null,
            "redirect": null
        }
    },
    "requiredNamespaces": null,
    "optionalNamespaces": null,
    "sessionProperties": null,
    "authentication": null,
    "transportType": "TransportType.relay"
}
```

