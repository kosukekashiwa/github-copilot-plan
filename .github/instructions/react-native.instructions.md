---
applyTo: "apps/mobile/**/*.{ts,tsx}"
---

# React Native 固有ルール

- ナビゲーションは <!-- 例: React Navigation / Expo Router --> を使用
- スタイルは `StyleSheet.create` または <!-- 例: NativeWind --> を使用。オブジェクトリテラルの直書き禁止
- プラットフォーム分岐は `Platform.select` か `.ios.tsx` / `.android.tsx` ファイル分割で行う
- web 用ライブラリ(DOM 依存)を import しない。共通化は `packages/shared` 経由で行う
- リストは `FlatList` / `FlashList` を使用し、`ScrollView` + map の組み合わせは避ける
- タッチターゲットは最小 44x44pt を確保する
- ネイティブモジュール追加(pod / gradle 変更が必要なもの)は PR 説明に必ず明記する
