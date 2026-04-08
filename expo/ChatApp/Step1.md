1. ### 创建expo客户端初始项目

```cmd
npx create-expo-app@latest
```

```cmd
npx expo run:ios // 运行开发模式到模拟器
cd ios && pod install && .. // ios安装依赖并成功后返回到项目根目录
```

2. 安装`react-native-paper`

```cmd
yarn add react-native-paper
yarn add react-native-safe-area-context
npx pod-install // ios平台
```

3. 配置`react-native-paper`

```tsx
import { Stack } from "expo-router";
import { StatusBar } from "expo-status-bar";
import {
  MD3LightTheme as DefaultTheme,
  PaperProvider,
} from "react-native-paper";
import "react-native-reanimated";

export const unstable_settings = {
  anchor: "(tabs)",
};

export default function RootLayout() {
  // 配置应用主题颜色
  const theme = {
    ...DefaultTheme,
    colors: {
      ...DefaultTheme.colors,
      primary: "tomato",
      secondary: "yellow",
    },
  };

  return (
    <PaperProvider theme={theme}>
      <Stack>
        <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
        <Stack.Screen
          name="modal"
          options={{ presentation: "modal", title: "Modal" }}
        />
      </Stack>
      <StatusBar style="auto" />
    </PaperProvider>
  );
}

```

配置动态主题

```tsx
import { useMaterial3Theme } from '@pchmn/expo-material3-theme';
import { useColorScheme } from 'react-native';
import { MD3DarkTheme, MD3LightTheme, PaperProvider } from 'react-native-paper';
import App from './src/App';

export default function Main() {
  const colorScheme = useColorScheme(); // 系统皮肤识别 dark light
  const { theme } = useMaterial3Theme(); // 获取颜色主题

  const paperTheme =
    colorScheme === 'dark'
      ? { ...MD3DarkTheme, colors: theme.dark }
      : { ...MD3LightTheme, colors: theme.light };

  return (
    <PaperProvider theme={paperTheme}>
      <App />
    </PaperProvider>
  );
}
```

使用内置`useTheme()`钩子来访问主题变量

主题配置

```json
{
  "colors": {
    "primary": "rgb(120, 69, 172)",
    "onPrimary": "rgb(255, 255, 255)",
    "primaryContainer": "rgb(240, 219, 255)",
    "onPrimaryContainer": "rgb(44, 0, 81)",
    "secondary": "rgb(102, 90, 111)",
    "onSecondary": "rgb(255, 255, 255)",
    "secondaryContainer": "rgb(237, 221, 246)",
    "onSecondaryContainer": "rgb(33, 24, 42)",
    "tertiary": "rgb(128, 81, 88)",
    "onTertiary": "rgb(255, 255, 255)",
    "tertiaryContainer": "rgb(255, 217, 221)",
    "onTertiaryContainer": "rgb(50, 16, 23)",
    "error": "rgb(186, 26, 26)",
    "onError": "rgb(255, 255, 255)",
    "errorContainer": "rgb(255, 218, 214)",
    "onErrorContainer": "rgb(65, 0, 2)",
    "background": "rgb(255, 251, 255)",
    "onBackground": "rgb(29, 27, 30)",
    "surface": "rgb(255, 251, 255)",
    "onSurface": "rgb(29, 27, 30)",
    "surfaceVariant": "rgb(233, 223, 235)",
    "onSurfaceVariant": "rgb(74, 69, 78)",
    "outline": "rgb(124, 117, 126)",
    "outlineVariant": "rgb(204, 196, 206)",
    "shadow": "rgb(0, 0, 0)",
    "scrim": "rgb(0, 0, 0)",
    "inverseSurface": "rgb(50, 47, 51)",
    "inverseOnSurface": "rgb(245, 239, 244)",
    "inversePrimary": "rgb(220, 184, 255)",
    "elevation": {
      "level0": "transparent",
      "level1": "rgb(248, 242, 251)",
      "level2": "rgb(244, 236, 248)",
      "level3": "rgb(240, 231, 246)",
      "level4": "rgb(239, 229, 245)",
      "level5": "rgb(236, 226, 243)"
    },
    "surfaceDisabled": "rgba(29, 27, 30, 0.12)",
    "onSurfaceDisabled": "rgba(29, 27, 30, 0.38)",
    "backdrop": "rgba(51, 47, 55, 0.4)"
  }
}
```

```ts
// constants/theme.ts
import {
  DarkTheme as NavigationDarkTheme,
  DefaultTheme as NavigationDefaultTheme,
} from "@react-navigation/native";
import {
  MD3DarkTheme,
  MD3LightTheme,
  adaptNavigationTheme,
} from "react-native-paper";

const sharedColors = {
  primary: "rgb(79, 216, 235)",
  secondary: "yellow",
};

const lightChatColors = {
  chatBubbleSelf: "#DCF8C6",
  chatBubbleOther: "#FFFFFF",
  chatBubbleSelfText: "#1a1a1a",
  chatBubbleOtherText: "#1a1a1a",
  chatTimestamp: "#8e8e93",
  chatInputBackground: "#F2F2F7",
  chatInputText: "#1a1a1a",
  statusOnline: "#34C759",
  statusOffline: "#8e8e93",
  statusBusy: "#FF3B30",
  unreadBadge: "#FF3B30",
  separator: "#E5E5EA",
};

const darkChatColors = {
  chatBubbleSelf: "#075E54",
  chatBubbleOther: "#1F2C34",
  chatBubbleSelfText: "#e0e0e0",
  chatBubbleOtherText: "#e0e0e0",
  chatTimestamp: "#8e8e93",
  chatInputBackground: "#1C1C1E",
  chatInputText: "#e0e0e0",
  statusOnline: "#30D158",
  statusOffline: "#636366",
  statusBusy: "#FF453A",
  unreadBadge: "#FF453A",
  separator: "#38383A",
};

export const lightTheme = {
  ...MD3LightTheme,
  colors: {
    ...MD3LightTheme.colors,
    ...sharedColors,
    ...lightChatColors,
  },
};

export const darkTheme = {
  ...MD3DarkTheme,
  colors: {
    ...MD3DarkTheme.colors,
    ...sharedColors,
    ...darkChatColors,
  },
};

export type AppTheme = typeof lightTheme;

const { LightTheme: navigationLightTheme, DarkTheme: navigationDarkTheme } =
  adaptNavigationTheme({
    reactNavigationLight: NavigationDefaultTheme,
    reactNavigationDark: NavigationDarkTheme,
    materialLight: lightTheme,
    materialDark: darkTheme,
  });

export { navigationDarkTheme, navigationLightTheme };

```

配置context

```tsx
import {
    darkTheme,
    lightTheme,
    navigationDarkTheme,
    navigationLightTheme,
} from "@/constants/theme";
import {
    ThemePreference,
    getThemePreference,
    setThemePreference as persistPreference,
} from "@/lib/storage";
import { StatusBar } from "expo-status-bar";
import React, { createContext, useCallback, useContext, useState } from "react";
import { useColorScheme } from "react-native";
import { PaperProvider } from "react-native-paper";

interface ThemeContextValue {
  preference: ThemePreference;
  setPreference: (p: ThemePreference) => void;
  isDark: boolean;
  navigationTheme: typeof navigationLightTheme;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const systemScheme = useColorScheme();
  const [preference, setPreferenceState] =
    useState<ThemePreference>(getThemePreference);

  const setPreference = useCallback((p: ThemePreference) => {
    persistPreference(p);
    setPreferenceState(p);
  }, []);

  const isDark =
    preference === "dark" ||
    (preference === "system" && systemScheme === "dark");

  const paperTheme = isDark ? darkTheme : lightTheme;
  const navigationTheme = isDark ? navigationDarkTheme : navigationLightTheme;

  return (
    <ThemeContext.Provider
      value={{ preference, setPreference, isDark, navigationTheme }}
    >
      <PaperProvider theme={paperTheme}>
        {children}
        <StatusBar style={isDark ? "light" : "dark"} />
      </PaperProvider>
    </ThemeContext.Provider>
  );
}

export function useThemeContext() {
  const ctx = useContext(ThemeContext);
  if (!ctx) {
    throw new Error("useThemeContext must be used within ThemeProvider");
  }
  return ctx;
}

```

```ts
//lib/storage.ts
import { createMMKV } from "react-native-mmkv";

export const storage = createMMKV({ id: "mmkv.default" });

const THEME_KEY = "theme_preference";

export type ThemePreference = "system" | "light" | "dark";

export function getThemePreference(): ThemePreference {
  const value = storage.getString(THEME_KEY);
  if (value === "light" || value === "dark") {
    return value;
  }
  return "system";
}

export function setThemePreference(preference: ThemePreference): void {
  storage.set(THEME_KEY, preference);
}

```

