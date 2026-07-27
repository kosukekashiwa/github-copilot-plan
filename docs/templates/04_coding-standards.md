# コーディング規約: <プロジェクト名>

| 項目 | 内容 |
| ---- | ---- |
| 版数 | v0.1 |

<!-- 対象スタック: Web(React) / Mobile(React Native) / API(Node.js + Express) -->

`.github/instructions/*.instructions.md` と内容が重複する場合はそちらを正とし、本書は「なぜそのルールか」「レビュー時の判断基準」まで含めた人間向け規約として運用する。

## 1. 基本方針

- 言語: TypeScript を標準とする(JS実装は原則不可、既存コード以外)
- フォーマッタ: Prettier / Lint: ESLint(設定ファイルを正とし、本書は補足)
- 迷ったら既存コードのパターンに合わせる。新しいパターンを導入する場合はPRで理由を明記

## 2. 共通規約

### 2.1 命名規則

| 対象 | 規則 | 例 |
| ---- | ---- | -- |
| ファイル名(コンポーネント) | PascalCase | `UserCard.tsx` |
| ファイル名(それ以外) | camelCase | `formatDate.ts` |
| 変数・関数 | camelCase | `getUserList` |
| 型・インターフェース | PascalCase | `type UserProfile` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| ディレクトリ | kebab-case | `user-settings/` |

### 2.2 ディレクトリ構成

<!-- packages/shared, apps/web, apps/mobile, apps/api 構成を想定。実際の構成に合わせて書き換える -->

```
packages/shared/  # Web/Mobile 共通の型定義・API クライアント・バリデーション
apps/web/         # React
apps/mobile/      # React Native
apps/api/         # Node.js / Express
```

### 2.3 型安全性

- `any` の使用は原則禁止。やむを得ない場合はコメントで理由を明記し、`unknown` + 型ガードを優先検討
- 型アサーション(`as`)は最小限に。ライブラリの型不足など明確な理由がある場合のみ許可
- API のリクエスト/レスポンス型は `packages/shared` に定義し、Web/Mobile/API で共有する

### 2.4 コメント

- コードの「何をしているか」は書かない(命名で表現する)
- 「なぜそうしているか」(隠れた制約、既知のバグへのワークアラウンド等)のみコメントする
- TODO コメントは Issue 番号を紐づける(例: `// TODO(#123): ...`)

## 3. Frontend(React)規約

- 関数コンポーネント + Hooks を標準とし、クラスコンポーネントは使用しない
- 1ファイル1コンポーネントを原則とする
- Props の型は同ファイル内に `type Props = {...}` で定義
- ビジネスロジックはカスタムフックに切り出し、コンポーネントは表示に専念させる
- グローバル状態は必要最小限にとどめ、まずローカル状態・Propsで解決できないか検討する
- スタイリング方針: <!-- CSS Modules / styled-components / Tailwind 等、採用方式を明記 -->

## 4. Mobile(React Native)規約

- プラットフォーム差分は `.ios.tsx` / `.android.tsx` の分割 or `Platform.select` で明示する
- ナビゲーションは <!-- React Navigation 等 --> の構成方針を明記
- Web と共通化できるロジック(バリデーション、APIクライアント、型定義)は `packages/shared` に置き、UI 部分のみプラットフォーム別に実装する
- `accessibilityLabel` を対話可能な要素に必ず付与する

## 5. Backend(Node.js/Express)規約

### 5.1 レイヤー構成

- `routes`: ルーティング定義のみ。ロジックを書かない
- `controllers`: リクエスト/レスポンスの変換、`services` の呼び出し
- `services`: ビジネスロジック。DB操作は `repositories` 経由で行う
- `repositories`: DB アクセスのみ

### 5.2 エラーハンドリング

- 共通エラーミドルウェアで一元的にハンドリングする(コントローラ内で個別に try/catch を乱立させない)
- エラーは意味のあるエラークラス(例: `NotFoundError`, `ValidationError`)を投げ、ミドルウェア側でHTTPステータスに変換する

### 5.3 環境変数

- `.env` はコミットしない(`.env.example` のみコミット)
- 環境変数へのアクセスは設定読み込み用モジュールに集約し、`process.env` を各所に直書きしない

## 6. Git運用規約

### 6.1 ブランチ戦略

<!-- 例: main / develop / feature/xxx / fix/xxx -->

### 6.2 コミットメッセージ規約

<!-- 例: Conventional Commits (feat:, fix:, refactor:, docs:, test:, chore:) -->

## 7. セキュリティ規約

- 秘密情報(APIキー、パスワード等)はコード・ログに含めない
- 外部入力(リクエストボディ、クエリパラメータ)は必ずバリデーションする
- `dangerouslySetInnerHTML` の使用は原則禁止。使用する場合はサニタイズ処理とレビューを必須とする
- SQL/NoSQL クエリはパラメータ化し、文字列結合で組み立てない

