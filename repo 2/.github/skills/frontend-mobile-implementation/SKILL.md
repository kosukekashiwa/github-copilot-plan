---
name: frontend-mobile-implementation
description: React Native（Mobile）の実装コードを追加・修正する際に使用する。画面・コンポーネント実装、ナビゲーション実装、Web側との共通ロジック活用の具体的な書き方を提供する。FE-Mobile対象Issueの実装作業で使う。
---

# React Native（Mobile）実装方針

`frontend-mobile-design` Skillで決まった設計を、実際のコードに落とし込むためのガイドです。
状態管理・API呼び出し・型定義のロジックはWeb側（`frontend-web-implementation` Skill）と共有パッケージを介して揃え、UI実装のみをここで扱います。

## 画面・コンポーネント実装

- 画面コンポーネントは `screens/` に、再利用可能な部品は `components/` に置く。
- スタイルは `StyleSheet.create` を使い、インラインスタイルの多用を避ける。

```tsx
interface PostListItemProps {
  post: Post;
  onPress?: (postId: string) => void;
}

export function PostListItem({ post, onPress }: PostListItemProps) {
  return (
    <Pressable style={styles.container} onPress={() => onPress?.(post.id)}>
      <Text>{post.body}</Text>
    </Pressable>
  );
}

const styles = StyleSheet.create({
  container: { padding: 12 },
});
```

## ナビゲーション実装

- 画面遷移パラメータの型は共有スキーマ（`data-model-design` Skill）から導出し、`RootStackParamList` 等に手書きで重複定義しない。
- ドメインごとにスタック/タブを分割する（`frontend-mobile-design` Skillの構成に従う）。

## 状態管理・API呼び出し（Web側との共通化）

- storeのロジック（Zustand slice）・APIクライアント・Zodスキーマは、モノレポの共有パッケージ（例: `packages/shared`）からimportし、Mobile側で再実装しない。
  ```tsx
  import { usePosts } from '@repo/shared/features/timeline';
  ```
- UIに依存する部分（ローディングインジケータ、リスト描画）のみMobile側で実装する。

## プラットフォーム差分の扱い

- ネイティブ機能（カメラ、通知、ファイルアクセスなど）が必要な場合は、抽象化したインターフェースを定義し、実装をプラットフォーム別ファイル（`*.ios.ts` / `*.android.ts`）に分離する。
- オフライン時の挙動（投稿・回答の再送など）は、キューイングして接続復帰時にリトライする実装を基本とする。

## 多言語・マルチテナント対応

- 端末ロケールとアプリ内言語設定の優先順位を明確に実装する（例: アプリ内設定 > 端末ロケール > デフォルト）。
- テナント設定の取得はWeb側と同じ共有フック（`useTenantConfig()`）を利用する。

## 実装後のセルフチェック

- Web側と共通化できるロジックを重複実装していないか
- プラットフォーム差分が必要な箇所を適切に分離できているか
- 対応するテストコードを `frontend-mobile-testing` Skillの方針で作成したか
