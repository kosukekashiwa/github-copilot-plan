# 詳細設計書(内部設計)- フロントエンド: <プロジェクト名>

| 項目 | 内容 |
| ---- | ---- |
| 関連文書 | 基本設計書: 02_basic-design-frontend.md / バックエンド詳細設計書: 03_detailed-design-backend.md |
| 版数 | v0.1 |
| ステータス | Draft / レビュー中 / 承認済 |

<!-- 対象スタック: Web(React) / Mobile(React Native) -->

API・DBの詳細設計は [03_detailed-design-backend.md](03_detailed-design-backend.md) を参照。

## 1. コンポーネント構成

### 1.1 フロントエンド(React)コンポーネント設計

<!-- コンポーネントツリー、Container/Presentational の分離方針、状態管理の配置 -->

| コンポーネント名 | 種別(Page/Container/Presentational) | 概要 | Props |
| ---------------- | -------------------------------------- | ---- | ----- |
| UserProfileCard | Presentational | ユーザーのプロフィール情報を表示するカード | `{ user: User; onEditPress?: () => void }` |
| | | | |

### 1.2 モバイル(React Native)コンポーネント設計

<!-- ナビゲーション構成(React Navigation 等)、Web との共通コンポーネント範囲 -->

| コンポーネント名 | 概要 | Web側との共通/差分 |
| ---------------- | ---- | ------------------- |
| UserProfileCard | ユーザーのプロフィール情報を表示するカード | ロジックは `packages/shared` を共通利用、UIのみプラットフォーム別実装 |
| | | |

## 2. 画面詳細仕様

<!-- 画面IDごとに作成。フォームがある画面は入力項目・チェック内容まで記載 -->

### 画面ID: S-001 <画面名>

- URLパス: <!-- 例: /users/:userId -->([02_basic-design-frontend.md](02_basic-design-frontend.md)「2.1 画面一覧」のURLパスと一致させる)

**パスパラメータ**

<!-- 型: string, number, boolean 等のプリミティブ型。IDは string(UUID) か number か明記する -->

| パラメータ名 | 型 | 必須 | 説明 |
| ------------- | -- | ---- | ---- |
| userId | string | 必須 | 対象ユーザーのID(UUID) |
| | | | |

**クエリパラメータ**

<!-- 型: string, number, boolean, string[] 等。複数値を取る場合は配列型で記載 -->

| パラメータ名 | 型 | 必須 | デフォルト | 説明 |
| ------------- | -- | ---- | ---------- | ---- |
| page | number | 任意 | 1 | ページ番号 |
| | | | | |

**入力項目**

<!-- 型: text, email, password, number, select, checkbox, radio, date, textarea, file 等の入力コントロール種別 -->

| 項目名 | 型 | 必須 | 入力チェック内容 | エラーメッセージ |
| ------ | -- | ---- | ------------------ | ------------------ |
| メールアドレス | email | 必須 | メール形式であること | 有効なメールアドレスを入力してください |
| | | | | |

- 使用API: <!-- 例: API-001 -->([03_detailed-design-backend.md](03_detailed-design-backend.md) のAPI IDを記載)
- 状態遷移: <!-- ローディング/成功/エラー/空状態 の表示パターン -->
- アクセシビリティ: <!-- semantic HTML(Web)、accessibilityLabel(Mobile) -->

## 3. フロントエンド共通処理・ユーティリティ設計

### 3.1 エラーハンドリング(クライアント側)

<!-- React エラーバウンダリ設計。APIエラーレスポンスを画面表示用メッセージへ変換する方針 -->

APIエラーレスポンスの形式は [03_detailed-design-backend.md](03_detailed-design-backend.md)「2.2 共通エラーレスポンス形式」を参照。

### 3.2 バリデーション設計(クライアント側)

<!-- 使用ライブラリ、重複方針、packages/shared でのルール共有方針 -->

サーバー側バリデーションは [03_detailed-design-backend.md](03_detailed-design-backend.md)「6.2 バリデーション設計(サーバー側)」を参照。

### 3.3 状態管理設計

<!-- グローバル状態(認証情報等)とローカル状態の切り分け、使用ライブラリ、キャッシュ戦略 -->

