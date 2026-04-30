### 安装数据库

```cmd
npx expo install expo-sqlite
```

```json
// app.json
{
  "expo": {
    "plugins": [
      [
        "expo-sqlite",
        {
          "enableFTS": true,
          "useSQLCipher": true,
          "android": {
            // Override the shared configuration for Android
            "enableFTS": false,
            "useSQLCipher": false
          },
          "ios": {
            // You can also override the shared configurations for iOS
            "customBuildFlags": ["-DSQLITE_ENABLE_DBSTAT_VTAB=1 -DSQLITE_ENABLE_SNAPSHOT=1"]
          }
        }
      ]
    ]
  }
}

```

### 使用方法

```js
import * as SQLite from 'expo-sqlite';


const db = SQLite.openDatabaseSync("chat.db");
    console.log(db);
    await db.execAsync(
      "CREATE TABLE IF NOT EXISTS messages (id INTEGER PRIMARY KEY AUTOINCREMENT, content TEXT, senderId TEXT, timestamp INTEGER)",
    );

    await db.runAsync(
      "INSERT INTO messages (content, senderId, timestamp) VALUES (?, ?, ?)",
      ["Hello, how are you?", "user_1", 1718380800000],
    );
    await db.runAsync(
      "INSERT INTO messages (content, senderId, timestamp) VALUES (?, ?, ?)",
      ["Hello, how are you?", "user_2", 1718380800000],
    );

    const rows = await db.getAllAsync("SELECT * FROM messages");
    console.log("rows", rows);

    db.closeAsync();
```

# Drizzle

```cmd
yarn add drizzle-orm expo-sqlite@next
yarn add -D drizzle-kit
```

在项目根目录下新建`drizzle`目录和`db`目录

```ts
// src/db/schema.ts
import { int, sqliteTable, text } from "drizzle-orm/sqlite-core";

export const usersTable = sqliteTable("users_table", {
  id: int().primaryKey({ autoIncrement: true }),
  name: text().notNull(),
  age: int().notNull(),
  email: text().notNull().unique(),
});

```

#### 设置 Drizzle 配置文件

```ts
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  dialect: 'sqlite',
  driver: 'expo',
  schema: './db/schema.ts',
  out: './drizzle',
});

```

#### 设置`metro`配置

```js
const { getDefaultConfig } = require('expo/metro-config');
/** @type {import('expo/metro-config').MetroConfig} */
const config = getDefaultConfig(__dirname);
config.resolver.sourceExts.push('sql');
module.exports = config;

```

#### 更新`babel`配置

```js
module.exports = function(api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [["inline-import", { "extensions": [".sql"] }]] // <-- add this
  };
};

```

#### 将更改应用到数据库

使用 Expo 时，您需要使用`drizzle-kit generate`命令生成迁移，然后在运行时使用`drizzle-orm` `migrate()`函数应用它们

```cmd
npx drizzle-kit generate


npx drizzle-kit generate --name init_chat_schema
```

#### 应用迁移并查询数据库

让我们创建包含迁移和查询的**App.tsx**文件，用于创建、读取、更新和删除用户。

```tsx
import { Text, View } from 'react-native';
import * as SQLite from 'expo-sqlite';
import { useEffect, useState } from 'react';
import { drizzle } from 'drizzle-orm/expo-sqlite';
import { usersTable } from './db/schema';
import { useMigrations } from 'drizzle-orm/expo-sqlite/migrator';
import migrations from './drizzle/migrations';

const expo = SQLite.openDatabaseSync('db.db');

const db = drizzle(expo);

export default function App() {
  const { success, error } = useMigrations(db, migrations);
  const [items, setItems] = useState<typeof usersTable.$inferSelect[] | null>(null);

  useEffect(() => {
    if (!success) return;

    (async () => {
      await db.delete(usersTable);

      await db.insert(usersTable).values([
        {
            name: 'John',
            age: 30,
            email: 'john@example.com',
        },
      ]);

      const users = await db.select().from(usersTable);
      setItems(users);
    })();
  }, [success]);

  if (error) {
    return (
      <View>
        <Text>Migration error: {error.message}</Text>
      </View>
    );
  }

  if (!success) {
    return (
      <View>
        <Text>Migration is in progress...</Text>
      </View>
    );
  }

  if (items === null || items.length === 0) {
    return (
      <View>
        <Text>Empty</Text>
      </View>
    );
  }

  return (
    <View
      style={{
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        width: '100%',
        height: '100%',
        justifyContent: 'center',
      }}
    >
      {items.map((item) => (
        <Text key={item.id}>{item.email}</Text>
      ))}
    </View>
  );
}

```

#### 预构建并运行 Expo 应用

```cmd
npx expo run:ios
```

