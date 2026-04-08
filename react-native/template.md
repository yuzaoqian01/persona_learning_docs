#### 1. Page Layout

```jsx
<PageLayout>
  <Header></Header>
  <Body></Body>
</PageLayout>
```

2. Screen Layout

```jsx
<Screen preset="scroll">
  <Text>Content</Text>
</Screen>
```

```jsx

import type { ReactNode } from "react";
import { useTranslation } from "react-i18next";
import {
  Modal,
  Pressable,
  ScrollView,
  StyleSheet,
  View,
  useWindowDimensions,
} from "react-native";
import { IconButton, Text, useTheme } from "react-native-paper";
import { useSafeAreaInsets } from "react-native-safe-area-context";

const SHEET_RADIUS = 16;

export interface AppBottomSheetProps {
  visible: boolean;
  onDismiss: () => void;
  /** 不传则不渲染标题文字，仍保留关闭按钮区域 */
  title?: string;
  children: ReactNode;
  /** 覆盖默认 i18n 无障碍文案 */
  closeAccessibilityLabel?: string;
  backdropAccessibilityLabel?: string;
}

/**
 * 通用底部弹层：仅负责遮罩、顶圆角面板、标题栏与可滚动内容区。
 * 业务内容通过 children 传入；其他页面可复用同一容器（筛选、代币列表等）。
 */
export default function AppBottomSheet({
  visible,
  onDismiss,
  title,
  children,
  closeAccessibilityLabel,
  backdropAccessibilityLabel,
}: AppBottomSheetProps) {
  const { t } = useTranslation();
  const theme = useTheme();
  const insets = useSafeAreaInsets();
  const { height: windowH } = useWindowDimensions();
  const maxSheetHeight = Math.round(windowH * 0.78);

  const a11yClose = closeAccessibilityLabel ?? t("sheet.a11y.close");
  const a11yBackdrop =
    backdropAccessibilityLabel ?? t("sheet.a11y.backdropDismiss");

  return (
    <Modal
      visible={visible}
      transparent
      animationType="fade"
      onRequestClose={onDismiss}
      statusBarTranslucent
    >
      <View style={styles.wrap}>
        <Pressable
          style={[styles.backdrop, StyleSheet.absoluteFillObject]}
          onPress={onDismiss}
          accessibilityRole="button"
          accessibilityLabel={a11yBackdrop}
        />
        <View
          style={[
            styles.sheet,
            {
              backgroundColor: theme.colors.surface,
              maxHeight: maxSheetHeight,
              paddingBottom: insets.bottom,
            },
          ]}
          accessibilityViewIsModal
        >
          <View style={styles.header}>
            {title ? (
              <Text
                variant="titleMedium"
                style={[styles.title, { color: theme.colors.onSurface }]}
              >
                {title}
              </Text>
            ) : (
              <View style={styles.titlePlaceholder} />
            )}
            <IconButton
              icon="close"
              size={22}
              onPress={onDismiss}
              accessibilityLabel={a11yClose}
              iconColor={theme.colors.onSurface}
              style={styles.closeBtn}
            />
          </View>
          <ScrollView
            style={styles.scroll}
            contentContainerStyle={styles.scrollContent}
            keyboardShouldPersistTaps="handled"
            showsVerticalScrollIndicator
          >
            {children}
          </ScrollView>
        </View>
      </View>
    </Modal>
  );
}

const styles = StyleSheet.create({
  wrap: {
    flex: 1,
    justifyContent: "flex-end",
  },
  backdrop: {
    backgroundColor: "rgba(0,0,0,0.45)",
  },
  sheet: {
    borderTopLeftRadius: SHEET_RADIUS,
    borderTopRightRadius: SHEET_RADIUS,
    overflow: "hidden",
  },
  header: {
    flexDirection: "row",
    alignItems: "center",
    justifyContent: "space-between",
    paddingLeft: 20,
    paddingRight: 8,
    paddingTop: 12,
    paddingBottom: 4,
  },
  title: { flex: 1, fontWeight: "700" },
  titlePlaceholder: { flex: 1 },
  closeBtn: { margin: 0 },
  scroll: { flexGrow: 0 },
  scrollContent: {
    paddingHorizontal: 16,
    paddingTop: 8,
    paddingBottom: 16,
  },
});

```

