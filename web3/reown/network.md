## 1. 依赖

Reown AppKit 底层使用[Viem](https://viem.sh/)网络，Viem 为 EVM 链提供了多种网络选择。您可以在此`@reown/appkit/networks`路径下找到 Viem 支持的所有网络。

```bash
import { createAppKit } from '@reown/appkit'
import { mainnet, arbitrum, base, scroll, polygon } from '@reown/appkit/networks'
```

