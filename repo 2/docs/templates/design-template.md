# 設計書: {対象Issueのタイトル}

## 対象Issue
- docs/issues/{数字}/issue.md

## 概要
<!-- 今回の設計が解決する課題と、採用した方針を1〜2行で -->

## 検討した設計案
<!-- 実際に比較検討した案を列挙する。1案のみだった場合も比較対象（現状維持など）を書く -->

### 案A: {案名}
- 概要:
- メリット:
- デメリット:

### 案B: {案名}
- 概要:
- メリット:
- デメリット:

## 採用案
- 採用: 案{X}
- 採用理由:
- トレードオフとして許容した点:

## データモデル・型定義
<!-- Zodスキーマ + z.infer、interface/typeの使い分けは data-model-design Skill の方針に従う -->
```typescript
// 例:
// const XxxSchema = z.object({ ... });
// type Xxx = z.infer<typeof XxxSchema>;
```

## コンポーネント構成（FE-Web / FE-Mobile が対象の場合）
```
src/
  features/
    xxx/
      components/
      hooks/
      store/        // Zustand slice
```
- 画面/コンポーネント一覧と責務:
- 状態管理方針（ローカル state / Zustand store の使い分け）:
- 多言語・マルチテナント対応時の考慮点:

## API設計（BEが対象の場合）
| Method | Path | 概要 | リクエスト | レスポンス |
| --- | --- | --- | --- | --- |
| | | | | |

- ドメイン境界・他ドメインとの依存関係:
- 非同期処理・イベント連携（ai-analysis連携など該当する場合）:
- マルチテナントにおけるデータ分離方式:

## 影響範囲
<!-- 既存の他ドメイン・画面・APIへの影響 -->
-

## 未解決事項・次のアクション
-

## 参考
-
