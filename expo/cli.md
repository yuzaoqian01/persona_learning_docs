清理

```cmd
# 1. 删除依赖
rm -rf node_modules yarn.lock pnpm-lock.yaml

# 2. 删除 iOS 构建
rm -rf ios/Pods ios/build

# 3. 删除 Xcode 缓存（关键）
rm -rf ~/Library/Developer/Xcode/DerivedData

# 4. 删除 Metro 缓存
rm -rf $TMPDIR/metro-* $TMPDIR/haste-map-*

# 5. 重新安装
yarn install

# 6. 重新 pods
cd ios && pod install && cd ..

# 7. 启动
npx expo start -c

npx expo start --clear

npx expo prebuild --clean


```

```cmd
npx expo run:ios
```

