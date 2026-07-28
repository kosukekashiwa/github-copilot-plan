---
name: frontend-web-design
description: React（Web）の画面・コンポーネント設計を行う際に使用する。コンポーネント構成、props型定義、Zustandによる状態管理の設計方針を提供する。FE-Web対象のIssueを設計するときに使う。
---

# React（Web）設計方針

このSkillは、React（Web）の新規画面・機能のコンポーネント設計を行うためのガイドです。

## 使うタイミング

- 新しい画面・機能をWebフロントエンドに追加するIssueの設計
- 既存画面のコンポーネント分割・状態管理をリファクタリングするIssueの設計

## コンポーネント構成の考え方

1. **ディレクトリ構成**: 機能（ドメイン）単位で `src/features/{domain}/` にまとめる。
   ```
   src/features/timeline/
     components/   # 画面固有のプレゼンテーショナルコンポーネント
     hooks/        # データ取得・ロジックを担うカスタムフック
     store/        # Zustand slice
     types.ts      # ドメイン固有の型定義
   ```
2. **責務の分離**:
   - Page/Containerコンポーネント: データ取得・状態購読・ハンドラの組み立てを担当
   - Presentationalコンポーネント: propsのみに依存し、副作用を持たない
3. 共通UIパーツ（多言語・マルチテナント双方で使う汎用コンポーネント）は `src/components/` に配置し、特定ドメインに依存させない。

## Props / 型定義の方針

- コンポーネントのpropsは **`interface`** で定義する（特にHTML属性を拡張する場合）。
  ```typescript
  interface UserCardProps extends React.ComponentProps<'div'> {
    userId: string;
    onSelect?: (userId: string) => void;
  }
  ```
- Union型やZodから推論する型（`z.infer<typeof XxxSchema>`）は **`type`** を使う。
- データモデル自体の設計は `data-model-design` Skillの方針に従う。

## 状態管理（Zustand スライスパターン）

- ローカルなUI状態（開閉・入力途中の値など）は `useState` で十分。複数コンポーネントで共有する状態、または非同期データのキャッシュはZustand storeに寄せる。
- 大きめの機能では、ドメインごとにスライスを作り、交差型で合成する。
  ```typescript
  type TimelineSlice = { posts: Post[]; fetchPosts: () => Promise<void> };
  type SurveySlice = { surveys: Survey[]; fetchSurveys: () => Promise<void> };

  type StoreState = TimelineSlice & SurveySlice;

  const createTimelineSlice: StateCreator<StoreState, [], [], TimelineSlice> = (set) => ({
    posts: [],
    fetchPosts: async () => { /* ... */ },
  });
  ```
- storeを肥大化させない。機能を横断する状態が増えてきたら、ドメイン境界に沿ってstoreを分割することを検討する。

## 多言語・マルチテナント対応時の考慮点

- 表示テキストはハードコードせず、i18nキー経由で参照する前提でコンポーネントを設計する。
- テナントごとに出し分けが必要なUI（ロゴ・カラー・機能フラグなど）は、propsまたはcontext経由でテナント設定を注入できる構成にする。
- React Native（`frontend-mobile-design` Skill）と共通化できるロジック（hooks・型定義・API呼び出し）は、モノレポの共有パッケージに切り出せるよう意識する。

## 設計案を出すときの観点

- Container/Presentational分割の粒度（コンポーネント数 vs 見通しの良さ）
- 状態をローカルで持つか、Zustand storeに上げるか
- 既存の共通コンポーネントとの再利用可否
