### 货币价值转换组件

```ts
// fiatStore.ts
import { create } from "zustand";

/** ISO 4217，与 `Intl.NumberFormat` `currency` 一致；用于法币展示与 `rates` 键 */
export const SUPPORTED_FIAT_CURRENCIES = [
  "USD",
  "CNY",
  "EUR",
  "GBP",
  "JPY",
  "HKD",
  "KRW",
] as const;

export type Currency = (typeof SUPPORTED_FIAT_CURRENCIES)[number];

/** USD 基准下「1 USD = ? 目标法币」占位；`init` 成功时由 CoinGecko 覆盖 */
export const DEFAULT_FIAT_RATES: Record<Currency, number> = {
  USD: 1,
  CNY: 6.98,
  EUR: 0.92,
  GBP: 0.79,
  JPY: 150,
  HKD: 7.78,
  KRW: 1350,
};

/** 内置默认单价键（与 `init` 拉取后的 symbol 一致）；新资产仍可用任意 string */
export const DEFAULT_TOKEN_PRICES = {
  ETH: 3000,
  BTC: 60000,
  USDT: 1,
  USDC: 1,
  SOL: 150,
  BNB: 600,
} as const;

export type KnownTokenSymbol = keyof typeof DEFAULT_TOKEN_PRICES;

/** 已知符号有 IDE 补全；`| string & {}` 保留对链上/API 新资产的赋值能力 */
export type AssetSymbol = KnownTokenSymbol | (string & {});

type State = {
  currency: Currency;
  rates: Record<string, number>;
  prices: Record<string, number>;
  loading: boolean;
  setCurrency: (c: Currency) => void;
  setRates: (rates: Record<string, number>) => void;
  setPrices: (prices: Record<string, number>) => void;
  setLoading: (loading: boolean) => void;
  init: () => Promise<void>;
};

/**
 * @description 将数值转换为数字，如果数值不是数字，则返回默认值
 * @param v - 数值
 * @param fallback - 默认值
 * @returns { result } - result: 数值
 */
function num(v: unknown, fallback: number): number {
  return typeof v === "number" && Number.isFinite(v) ? v : fallback;
}

/**
 * @description 从汇率数据中读取汇率
 * @param rates - 汇率数据
 * @param lowerKey - 货币符号的小写形式
 * @param fallback - 默认值
 * @returns { result } - result: 汇率，如果汇率数据不存在，则返回默认值
 */
function readCoingeckoFiatRate(
  rates: Record<string, { value?: number }> | undefined,
  lowerKey: string,
  fallback: number,
): number {
  return num(rates?.[lowerKey]?.value, fallback);
}

/**
 * @description 法币数据Store
 * @param set - 设置状态
 * @param get - 获取状态
 * @returns { result } - result: 法币数据Store，包含货币、汇率、币价、加载状态等
 */
export const useFiatStore = create<State>((set, get) => ({
  currency: "USD",

  rates: { ...DEFAULT_FIAT_RATES },

  prices: { ...DEFAULT_TOKEN_PRICES },

  loading: false,

  setCurrency: (currency) => set({ currency }),

  setRates: (rates) => set({ rates }),

  setPrices: (prices) => set({ prices }),

  setLoading: (loading) => set({ loading }),

  init: async () => {
    const prevRates = get().rates;
    const prevPrices = get().prices;

    try {
      set({ loading: true });

      const rateRes = await fetch(
        "https://api.coingecko.com/api/v3/exchange_rates",
      );
      const rateData = await rateRes.json();
      const r = rateData.rates as
        | Record<string, { value?: number }>
        | undefined;

      const rates: Record<Currency, number> = {
        USD: 1,
        CNY: readCoingeckoFiatRate(
          r,
          "cny",
          num(prevRates.CNY, DEFAULT_FIAT_RATES.CNY),
        ),
        EUR: readCoingeckoFiatRate(
          r,
          "eur",
          num(prevRates.EUR, DEFAULT_FIAT_RATES.EUR),
        ),
        GBP: readCoingeckoFiatRate(
          r,
          "gbp",
          num(prevRates.GBP, DEFAULT_FIAT_RATES.GBP),
        ),
        JPY: readCoingeckoFiatRate(
          r,
          "jpy",
          num(prevRates.JPY, DEFAULT_FIAT_RATES.JPY),
        ),
        HKD: readCoingeckoFiatRate(
          r,
          "hkd",
          num(prevRates.HKD, DEFAULT_FIAT_RATES.HKD),
        ),
        KRW: readCoingeckoFiatRate(
          r,
          "krw",
          num(prevRates.KRW, DEFAULT_FIAT_RATES.KRW),
        ),
      };

      const priceRes = await fetch(
        "https://api.coingecko.com/api/v3/simple/price?ids=ethereum,bitcoin,tether,usd-coin,solana,binancecoin&vs_currencies=usd",
      );
      const priceData = (await priceRes.json()) as Record<
        string,
        { usd?: number }
      >;

      const prices = {
        ETH: num(
          priceData.ethereum?.usd,
          num(prevPrices.ETH, DEFAULT_TOKEN_PRICES.ETH),
        ),
        BTC: num(
          priceData.bitcoin?.usd,
          num(prevPrices.BTC, DEFAULT_TOKEN_PRICES.BTC),
        ),
        USDT: num(
          priceData.tether?.usd,
          num(prevPrices.USDT, DEFAULT_TOKEN_PRICES.USDT),
        ),
        USDC: num(
          priceData["usd-coin"]?.usd,
          num(prevPrices.USDC, DEFAULT_TOKEN_PRICES.USDC),
        ),
        SOL: num(
          priceData.solana?.usd,
          num(prevPrices.SOL, DEFAULT_TOKEN_PRICES.SOL),
        ),
        BNB: num(
          priceData.binancecoin?.usd,
          num(prevPrices.BNB, DEFAULT_TOKEN_PRICES.BNB),
        ),
      };

      set({
        rates,
        prices,
      });
    } catch (e) {
      console.error("fiat init error", e);
    } finally {
      set({ loading: false });
    }
  },
}));

```

```tsx
// fiat-text
import { useFiat } from "@/hooks/use-fait";
import type { AssetSymbol } from "@/store/faitStore";
import React from "react";
import { Text, type StyleProp, type TextStyle } from "react-native";
import { formatFiat } from "./format";

/**
 * @description 法币文本
 * @param value - 价值
 * @param symbol - 货币符号
 * @returns { result } - result: 法币文本
 * @example
 * <FiatText value={100} symbol="ETH" /> // 100 ETH = 100 * 3000 USD = 300000 USD
 * <FiatText value={100} symbol="BTC" /> // 100 BTC = 100 * 30000 USD = 3000000 USD
 */

export const FiatText = React.memo(function FiatText({
  value,
  symbol,
  style,
}: {
  value: number;
  symbol?: AssetSymbol;
  style?: StyleProp<TextStyle>;
}) {
  const { currency, convert } = useFiat();

  const result = convert(value, symbol);

  if (result == null || Number.isNaN(result)) {
    return <Text style={style}>--</Text>;
  }

  return <Text style={style}>{formatFiat(result, currency)}</Text>;
});

```

```ts
// format.ts
/**
 * @description 格式化 token 金额
 * @param value - 金额
 * @param precision - 精度
 * @returns { result } - result: 金额文本
 */
export function formatToken(value: number, precision = 4) {
  return Number(value).toFixed(precision);
}

/**
 * 法币展示：`currency` 须为 **ISO 4217**（如 USD、JPY），与 `faitStore` 的 `Currency` 一致。
 * 已知限制：统一使用 `maximumFractionDigits: 2`，JPY/KRW 等零小数货币在部分区域可能非最优展示。
 */
export function formatFiat(value: number, currency: string) {
  return new Intl.NumberFormat(undefined, {
    style: "currency",
    currency,
    maximumFractionDigits: 2,
  }).format(value);
}

```

```tsx
// amount-text
import { FiatText } from "@/components/fiat/fiat-text";
import { formatToken } from "@/components/fiat/format";
import { useAppTheme } from "@/hooks/use-app-theme";
import type { AssetSymbol } from "@/store/faitStore";
import React from "react";
import { Text, View } from "react-native";

interface AmountTextProps {
  value: number;
  symbol: AssetSymbol;
  /** 主展示：代币数量行 或 法币估值行 */
  primary?: "token" | "fiat";
  /** 是否显示次行（主代币时次行为法币；主法币时次行为代币） */
  showSecondary?: boolean;
  precision?: number;
}

/**
 * @description 金额文本
 * @param primary - 主行展示代币还是法币
 * @param showSecondary - 是否显示另一币种作为次行
 *
 * @remarks 当前使用点：`app/(tabs)/settings/index.tsx`
 */
export const AmountText = React.memo(function AmountText({
  value,
  symbol,
  primary = "fiat",
  showSecondary = true,
  precision = 4,
}: AmountTextProps) {
  const theme = useAppTheme();
  const formatted = formatToken(value, precision);

  const primaryStyle = {
    fontSize: 16,
    fontWeight: "600" as const,
    color: theme.colors.onSurface,
  };
  const secondaryStyle = {
    fontSize: 12,
    color: theme.colors.onSurfaceVariant,
  };

  if (primary === "fiat") {
    return (
      <View>
        <FiatText value={value} symbol={symbol} style={primaryStyle} />
        {showSecondary ? (
          <Text style={secondaryStyle}>
            {formatted} {symbol}
          </Text>
        ) : null}
      </View>
    );
  }

  return (
    <View>
      <Text style={primaryStyle}>
        {formatted} {symbol}
      </Text>
      {showSecondary ? (
        <Text style={secondaryStyle}>
          ≈ <FiatText value={value} symbol={symbol} style={secondaryStyle} />
        </Text>
      ) : null}
    </View>
  );
});

```

