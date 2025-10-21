1. ### 安装

**一个 Flutter 插件，允许您添加内联 webview、使用无头 webview 以及打开应用内浏览器窗口**

```dart
flutter pub add flutter_inappwebview
```

```dart
// 确保初始化
void main() {
  // it should be the first line in main method
  WidgetsFlutterBinding.ensureInitialized();

  // rest of your app code
  runApp(MyApp());
}
```



```dart
import 'dart:async';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:flutter_inappwebview/flutter_inappwebview.dart';

import 'package:flutter_inappwebview_example/chrome_safari_browser_example.screen.dart';
import 'package:flutter_inappwebview_example/headless_in_app_webview.screen.dart';
import 'package:flutter_inappwebview_example/in_app_webiew_example.screen.dart';
import 'package:flutter_inappwebview_example/in_app_browser_example.screen.dart';
import 'package:flutter_inappwebview_example/web_authentication_session_example.screen.dart';
import 'package:pointer_interceptor/pointer_interceptor.dart';

// import 'package:path_provider/path_provider.dart';
// import 'package:permission_handler/permission_handler.dart';

final localhostServer = InAppLocalhostServer(documentRoot: 'assets');
WebViewEnvironment? webViewEnvironment;

Future main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // await Permission.camera.request();
  // await Permission.microphone.request();
  // await Permission.storage.request();

  if (!kIsWeb && defaultTargetPlatform == TargetPlatform.windows) {
    final availableVersion = await WebViewEnvironment.getAvailableVersion();
    assert(availableVersion != null,
        'Failed to find an installed WebView2 runtime or non-stable Microsoft Edge installation.');

    webViewEnvironment = await WebViewEnvironment.create(
        settings: WebViewEnvironmentSettings(userDataFolder: 'custom_path'));
  }

  if (!kIsWeb && defaultTargetPlatform == TargetPlatform.android) {
    await InAppWebViewController.setWebContentsDebuggingEnabled(kDebugMode);
  }

  runApp(MyApp());
}

PointerInterceptor myDrawer({required BuildContext context}) {
  var children = [
    ListTile(
      title: Text('InAppWebView'),
      onTap: () {
        Navigator.pushReplacementNamed(context, '/');
      },
    ),
    ListTile(
      title: Text('InAppBrowser'),
      onTap: () {
        Navigator.pushReplacementNamed(context, '/InAppBrowser');
      },
    ),
    ListTile(
      title: Text('ChromeSafariBrowser'),
      onTap: () {
        Navigator.pushReplacementNamed(context, '/ChromeSafariBrowser');
      },
    ),
    ListTile(
      title: Text('WebAuthenticationSession'),
      onTap: () {
        Navigator.pushReplacementNamed(context, '/WebAuthenticationSession');
      },
    ),
    ListTile(
      title: Text('HeadlessInAppWebView'),
      onTap: () {
        Navigator.pushReplacementNamed(context, '/HeadlessInAppWebView');
      },
    ),
  ];
  if (kIsWeb) {
    children = [
      ListTile(
        title: Text('InAppWebView'),
        onTap: () {
          Navigator.pushReplacementNamed(context, '/');
        },
      )
    ];
  } else if (defaultTargetPlatform == TargetPlatform.macOS) {
    children = [
      ListTile(
        title: Text('InAppWebView'),
        onTap: () {
          Navigator.pushReplacementNamed(context, '/');
        },
      ),
      ListTile(
        title: Text('InAppBrowser'),
        onTap: () {
          Navigator.pushReplacementNamed(context, '/InAppBrowser');
        },
      ),
      ListTile(
        title: Text('WebAuthenticationSession'),
        onTap: () {
          Navigator.pushReplacementNamed(context, '/WebAuthenticationSession');
        },
      ),
      ListTile(
        title: Text('HeadlessInAppWebView'),
        onTap: () {
          Navigator.pushReplacementNamed(context, '/HeadlessInAppWebView');
        },
      ),
    ];
  } else if (defaultTargetPlatform == TargetPlatform.windows ||
      defaultTargetPlatform == TargetPlatform.linux) {
    children = [
      ListTile(
        title: Text('InAppWebView'),
        onTap: () {
          Navigator.pushReplacementNamed(context, '/');
        },
      ),
      ListTile(
        title: Text('InAppBrowser'),
        onTap: () {
          Navigator.pushReplacementNamed(context, '/InAppBrowser');
        },
      ),
      ListTile(
        title: Text('HeadlessInAppWebView'),
        onTap: () {
          Navigator.pushReplacementNamed(context, '/HeadlessInAppWebView');
        },
      ),
    ];
  }
  return PointerInterceptor(
    child: Drawer(
      child: ListView(
        padding: EdgeInsets.zero,
        children: <Widget>[
          DrawerHeader(
            child: Text('flutter_inappwebview example'),
            decoration: BoxDecoration(
              color: Colors.blue,
            ),
          ),
          ...children
        ],
      ),
    ),
  );
}

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  @override
  void initState() {
    super.initState();
  }

  @override
  void dispose() {
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (kIsWeb) {
      return MaterialApp(initialRoute: '/', routes: {
        '/': (context) => InAppWebViewExampleScreen(),
      });
    }
    if (defaultTargetPlatform == TargetPlatform.macOS) {
      return MaterialApp(initialRoute: '/', routes: {
        '/': (context) => InAppWebViewExampleScreen(),
        '/InAppBrowser': (context) => InAppBrowserExampleScreen(),
        '/HeadlessInAppWebView': (context) =>
            HeadlessInAppWebViewExampleScreen(),
        '/WebAuthenticationSession': (context) =>
            WebAuthenticationSessionExampleScreen(),
      });
    } else if (defaultTargetPlatform == TargetPlatform.windows ||
        defaultTargetPlatform == TargetPlatform.linux) {
      return MaterialApp(initialRoute: '/', routes: {
        '/': (context) => InAppWebViewExampleScreen(),
        '/InAppBrowser': (context) => InAppBrowserExampleScreen(),
        '/HeadlessInAppWebView': (context) =>
            HeadlessInAppWebViewExampleScreen(),
      });
    }
    return MaterialApp(initialRoute: '/', routes: {
      '/': (context) => InAppWebViewExampleScreen(),
      '/InAppBrowser': (context) => InAppBrowserExampleScreen(),
      '/ChromeSafariBrowser': (context) => ChromeSafariBrowserExampleScreen(),
      '/HeadlessInAppWebView': (context) => HeadlessInAppWebViewExampleScreen(),
      '/WebAuthenticationSession': (context) =>
          WebAuthenticationSessionExampleScreen(),
    });
  }
}
```

```json
{
  "id": 1757421379284796,
  "params": {
    "id": 1757421379284796,
    "expiry": 1757421686,
    "relays": [
      {
        "protocol": "irn",
        "data": null
      }
    ],
    "proposer": {
      "publicKey": "ab9812bc57e70ff78d0c8b14c435051b5ffb46deb1a12c86bcbd1c225218a33b",
      "metadata": {
        "name": "AppKit",
        "description": "AppKit Example",
        "url": "https://dapp-tools.pages.dev",
        "icons": ["https://avatars.githubusercontent.com/u/179229932"]
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
        "events": ["accountsChanged", "chainChanged"]
      },
      "solana": {
        "chains": [
          "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp",
          "solana:4sGjMW1sUnHzSxGspuhpqLDx6wiyjNtZ",
          "solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1",
          "solana:8E9rvCKLFQia2Y35HXjjpWzj8weVo44K"
        ],
        "methods": [
          "solana_signMessage",
          "solana_signTransaction",
          "solana_requestAccounts",
          "solana_getAccounts",
          "solana_signAllTransactions",
          "solana_signAndSendTransaction"
        ],
        "events": ["accountsChanged", "chainChanged"]
      },
      "bip122": {
        "chains": ["bip122:000000000019d6689c085ae165831e93"],
        "methods": ["sendTransfer", "signMessage", "signPsbt", "getAccountAddresses"],
        "events": ["accountsChanged", "chainChanged"]
      }
    },
    "pairingTopic": "0607567b64aa2c528fdd897aa3f7390b4ebf016240e15f8c64498aabfbdc4494",
    "generatedNamespaces": {
      "eip155": {
        "chains": ["eip155:1", "eip155:137"],
        "accounts": [
          "eip155:1:0x00002F528A2256dBB25Cd57D0F1bC6dade192222",
          "eip155:137:0x00002F528A2256dBB25Cd57D0F1bC6dade192222"
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
        "events": ["chainChanged", "accountsChanged"]
      }
    }
  },
  "verifyContext": {
    "origin": "https://dapptools.pages.dev",
    "validation": "VALID",
    "verifyUrl": "https://verify.walletconnect.org",
    "isScam": false
  }
}

```

