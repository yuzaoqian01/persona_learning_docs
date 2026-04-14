```tsx
import { ConversationListItem } from "@/components/conversation-list-item";
import { useAppTheme } from "@/hooks/use-app-theme";
import React, { useCallback, useMemo, useState } from "react";
import {
  Platform,
  StyleSheet,
  Text,
  TouchableOpacity,
  View,
} from "react-native";
import { Appbar, Searchbar } from "react-native-paper";
import { useSafeAreaInsets } from "react-native-safe-area-context";
import { SwipeListView } from "react-native-swipe-list-view";

// 模拟会话数据
const INITIAL_MOCK_CONVERSATIONS = Array.from({ length: 1000 }, (_, index) => {
  return {
    id: index + 1,
    avatar: `https://i.pravatar.cc/150?img=${index + 1}`,
    name: `Alice ${index + 1}`,
    lastMessage: `Hi, how are you Hi, how are youHi, how are youHi, how are youHi, how are youHi, how are you? ${index + 1}`,
    time: "10:00",
    unreadCount: Math.floor(Math.random() * 100),
    isSticky: false,
    isMuted: true,
  };
});

/** 与 ConversationListItem 一致：paddingVertical 12×2 + Avatar 48 */
const CONVERSATION_ROW_HEIGHT = 72;

export default function ChatScreen() {
  const theme = useAppTheme();
  const insets = useSafeAreaInsets();
  const [searchQuery, setSearchQuery] = useState("");
  const [conversations, setConversations] = useState(
    INITIAL_MOCK_CONVERSATIONS,
  );

  // 置顶
  const toggleSticky = useCallback((id: string, rowMap: any) => {
    if (rowMap[id]) {
      rowMap[id].closeRow();
    }
    setConversations((prev) =>
      prev.map((c) => (c.id === id ? { ...c, isSticky: !c.isSticky } : c)),
    );
  }, []);

  // 免打扰
  const toggleMute = useCallback((id: string, rowMap: any) => {
    if (rowMap[id]) {
      rowMap[id].closeRow();
    }
    setConversations((prev) =>
      prev.map((c) => (c.id === id ? { ...c, isMuted: !c.isMuted } : c)),
    );
  }, []);

  // 删除
  const deleteConversation = useCallback((id: string, rowMap: any) => {
    if (rowMap[id]) {
      rowMap[id].closeRow();
    }
    setConversations((prev) => prev.filter((c) => c.id !== id));
  }, []);

  // 渲染列表项
  const renderItem = useCallback(
    ({ item }: { item: (typeof INITIAL_MOCK_CONVERSATIONS)[0] }) => (
      <ConversationListItem
        avatar={item.avatar}
        name={item.name}
        lastMessage={item.lastMessage}
        time={item.time}
        unreadCount={item.unreadCount}
        isSticky={item.isSticky}
        isMuted={item.isMuted}
        onPress={() => console.log("Pressed", item.name)}
      />
    ),
    [],
  );

  // 渲染隐藏项
  const renderHiddenItem = useCallback(
    (data: { item: (typeof INITIAL_MOCK_CONVERSATIONS)[0] }, rowMap: any) => {
      const item = data.item;
      return (
        <View style={styles.rowBack}>
          <View style={styles.rightActionsContainer}>
            <TouchableOpacity
              style={[
                styles.actionButton,
                { backgroundColor: theme.colors.blue06 },
              ]}
              onPress={() => toggleSticky(item.id, rowMap)}
            >
              <Text style={styles.actionText}>
                {item.isSticky ? "取消置顶" : "置顶"}
              </Text>
            </TouchableOpacity>
            <TouchableOpacity
              style={[
                styles.actionButton,
                { backgroundColor: theme.colors.yellow06 },
              ]}
              onPress={() => toggleMute(item.id, rowMap)}
            >
              <Text style={styles.actionText}>
                {item.isMuted ? "取消免打扰" : "免打扰"}
              </Text>
            </TouchableOpacity>
            <TouchableOpacity
              style={[
                styles.actionButton,
                { backgroundColor: theme.colors.red06 },
              ]}
              onPress={() => deleteConversation(item.id, rowMap)}
            >
              <Text style={styles.actionText}>删除</Text>
            </TouchableOpacity>
          </View>
        </View>
      );
    },
    [
      theme.colors.blue06,
      theme.colors.red06,
      theme.colors.yellow06,
      toggleSticky,
      toggleMute,
      deleteConversation,
    ],
  );

  // 对列表进行排序：置顶的在前面（避免每次渲染全量 sort）
  const sortedConversations = useMemo(
    () =>
      [...conversations].sort((a, b) => {
        if (a.isSticky === b.isSticky) return 0;
        return a.isSticky ? -1 : 1;
      }),
    [conversations],
  );

  // 获取列表项布局
  const getItemLayout = useCallback(
    (_: unknown, index: number) => ({
      length: CONVERSATION_ROW_HEIGHT,
      offset: CONVERSATION_ROW_HEIGHT * index,
      index,
    }),
    [],
  );

  // 提取唯一标识符
  const keyExtractor = useCallback(
    (item: (typeof INITIAL_MOCK_CONVERSATIONS)[0]) => item.id,
    [],
  );

  return (
    <View
      style={[styles.container, { backgroundColor: theme.colors.background }]}
    >
      <Appbar.Header
        statusBarHeight={insets.top}
        style={{ backgroundColor: theme.colors.background }}
      >
        <Searchbar
          placeholder="搜索"
          onChangeText={setSearchQuery}
          value={searchQuery}
          style={styles.searchBar}
          inputStyle={{ minHeight: 0 }}
        />
        <Appbar.Action icon="qrcode-scan" onPress={() => {}} />
        <Appbar.Action icon="plus-circle-outline" onPress={() => {}} />
      </Appbar.Header>

      <SwipeListView
        data={sortedConversations}
        keyExtractor={keyExtractor}
        renderItem={renderItem}
        renderHiddenItem={renderHiddenItem}
        rightOpenValue={-240} // 3 个按钮，每个 80 宽
        disableRightSwipe
        disableHiddenLayoutCalculation
        getItemLayout={getItemLayout}
        initialNumToRender={12}
        maxToRenderPerBatch={12}
        windowSize={7}
        updateCellsBatchingPeriod={50}
        removeClippedSubviews={Platform.OS === "android"}
        contentContainerStyle={{ paddingBottom: insets.bottom }}
        showsVerticalScrollIndicator={false}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  searchBar: {
    flex: 1,
    marginHorizontal: 8,
    height: 36,
  },
  rowBack: {
    alignItems: "center",
    flex: 1,
    flexDirection: "row",
    justifyContent: "flex-end",
  },
  rightActionsContainer: {
    flexDirection: "row",
    height: "100%",
  },
  actionButton: {
    justifyContent: "center",
    alignItems: "center",
    width: 80,
    height: "100%",
  },
  actionText: {
    color: "white",
    fontSize: 14,
    fontWeight: "500",
  },
});

```

```tsx
// ConversationListItem
import { useAppTheme } from "@/hooks/use-app-theme";
import React from "react";
import {
  StyleProp,
  StyleSheet,
  Text,
  TouchableHighlight,
  View,
  ViewStyle,
} from "react-native";
import { Avatar, Badge, Icon } from "react-native-paper";

interface ConversationListItemProps {
  /**
   * 头像 URL 或本地图片资源
   */
  avatar: string | number;
  /**
   * 会话名称
   */
  name: string;
  /**
   * 最后一条消息
   */
  lastMessage: string;
  /**
   * 最后一条消息时间
   */
  time: string;
  /**
   * 未读消息数量
   */
  unreadCount?: number;
  /**
   * 是否置顶
   */
  isSticky?: boolean;
  /**
   * 是否免打扰
   */
  isMuted?: boolean;
  /**
   * 自定义样式
   */
  style?: StyleProp<ViewStyle>;
  /**
   * 点击事件
   */
  onPress?: () => void;
}

/**
 * 微信风格的会话列表项组件
 */
export const ConversationListItem = React.memo(function ConversationListItem({
  avatar,
  name,
  lastMessage,
  time,
  unreadCount,
  isSticky,
  isMuted,
  style,
  onPress,
}: ConversationListItemProps) {
  const theme = useAppTheme();

  // TouchableHighlight 很重要 在swiper-native-list 里面
  return (
    <TouchableHighlight
      onPress={onPress}
      underlayColor={isSticky ? theme.colors.grey04 : theme.colors.grey03}
      activeOpacity={0.8}
    >
      <View
        style={[
          styles.container,
          {
            backgroundColor: isSticky
              ? theme.colors.grey04
              : theme.colors.background,
            borderBottomColor: theme.colors.outlineVariant,
          },
          style,
        ]}
      >
        <View style={styles.avatarContainer}>
          {typeof avatar === "string" ? (
            <Avatar.Image size={48} source={{ uri: avatar }} />
          ) : (
            <Avatar.Image size={48} source={avatar} />
          )}
          {unreadCount && unreadCount > 0 ? (
            <Badge size={20} style={styles.badge}>
              {unreadCount > 99 ? "99+" : unreadCount}
            </Badge>
          ) : null}
        </View>

        <View style={styles.content}>
          <View style={styles.header}>
            <Text style={[styles.name, { color: theme.colors.onSurface }]}>
              {name}
            </Text>
            <Text style={[styles.time, { color: theme.colors.grey05 }]}>
              {time}
            </Text>
          </View>
          <View style={styles.messageRow}>
            <Text
              style={[styles.lastMessage, { color: theme.colors.grey05 }]}
              numberOfLines={1}
            >
              {lastMessage}
            </Text>
            {isMuted ? (
              <Icon
                source="bell-off-outline"
                size={16}
                color={theme.colors.grey05}
                style={styles.muteIcon}
              />
            ) : null}
          </View>
        </View>
      </View>
    </TouchableHighlight>
  );
});

const styles = StyleSheet.create({
  container: {
    flexDirection: "row",
    alignItems: "center",
    paddingVertical: 12,
    paddingHorizontal: 16,
    borderBottomWidth: StyleSheet.hairlineWidth,
  },
  avatarContainer: {
    position: "relative",
    marginRight: 12,
  },
  badge: {
    position: "absolute",
    top: -4,
    right: -4,
    backgroundColor: "red",
  },
  content: {
    flex: 1,
    justifyContent: "center",
  },
  header: {
    flexDirection: "row",
    justifyContent: "space-between",
    alignItems: "center",
    marginBottom: 4,
  },
  messageRow: {
    flexDirection: "row",
    alignItems: "center",
  },
  muteIcon: {
    marginLeft: 8,
  },
  name: {
    fontSize: 16,
    fontWeight: "600",
    flexShrink: 1,
    marginRight: 8,
  },
  time: {
    fontSize: 12,
  },
  lastMessage: {
    flex: 1,
    minWidth: 0,
    fontSize: 14,
  },
});

```

