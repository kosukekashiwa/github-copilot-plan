# 基本設計書(外部設計)- フロントエンド: <プロジェクト名>

| 項目 | 内容 |
| ---- | ---- |
| 関連文書 | 要件定義書: 01_requirements-definition.md / 基本設計書(共通): 02_basic-design-common.md / バックエンド基本設計書: 02_basic-design-backend.md |
| 版数 | v0.1 |
| ステータス | Draft / レビュー中 / 承認済 |

<!-- 対象スタック: Web(React) / Mobile(React Native) -->

全体構成図・環境一覧は [02_basic-design-common.md](02_basic-design-common.md) を参照。API・DB・インフラの設計は [02_basic-design-backend.md](02_basic-design-backend.md) を参照。

## 1. システム構成

### 1.1 全体構成図

[02_basic-design-common.md](02_basic-design-common.md)「1.1 全体構成図」を参照。

### 1.2 フロントエンド実行環境

API/DB/CI-CDの実行環境は [02_basic-design-backend.md](02_basic-design-backend.md)「1.2 バックエンド実行環境」を参照。

| 層 | 技術・サービス | 備考 |
| -- | -------------- | ---- |
| Web フロントエンド | React | ホスティング先: 例: Vercel |
| モバイル | React Native | 配信: App Store / Google Play |

### 1.3 環境一覧

[02_basic-design-common.md](02_basic-design-common.md)「1.2 環境一覧」を参照。

## 2. 画面設計(Web / Mobile)

### 2.1 画面一覧

<!-- URLパス: Web はルーティングパス(例: /users/:userId)。Mobile はディープリンク対応する画面のみ
     URLスキームのパスを記載(例: myapp://users/:userId)。ディープリンクの実装方針自体は4.2参照 -->

| 画面ID | 画面名 | URLパス | 概要 | 対象プラットフォーム | 関連機能ID |
| ------ | ------ | -------- | ---- | --------------------- | ---------- |
| S-001 | 会員登録画面 | /signup | メールアドレス・パスワードを入力して会員登録する画面 | Web / iOS / Android | F-001 |

### 2.2 画面遷移図

<!-- 画面ID同士の遷移をフロー図で表現。Web / Mobile で共通/差異がある場合は明記 -->

例: トップ画面(S-000) → 会員登録画面(S-001) → 登録完了画面(S-002)。登録失敗時は S-001 に留まりエラーメッセージを表示。

### 2.3 画面レイアウト方針

<!-- 共通レイアウト(ヘッダー・フッター・ナビゲーション)、レスポンシブ方針、
     Web/Mobile でのUIコンポーネント共通化方針(例: packages/shared の共有戦略) -->

例: ヘッダー・フッターは全画面共通レイアウトとし、`packages/shared` に定義したデザイントークン(色・余白)をWeb/Mobile双方から参照する。Webはブレークポイント768px未満をスマートフォン表示とする。

## 3. 非機能設計方針(フロントエンド)

### 3.1 パフォーマンス方針

<!-- バンドルサイズ最適化、画像最適化、コード分割、クライアントキャッシュ戦略(TanStack Query 等) -->

サーバー側の性能方針は [02_basic-design-backend.md](02_basic-design-backend.md)「5.1 性能方針」を参照。

### 3.2 モバイル運用方針

<!-- 要件定義書「6.6 モバイル運用要件」を実現方式に落とし込む。Web のみの場合は表の各項目を「対象外」と明記する(要件定義書との対応関係を追えるよう、節自体は削除しない) -->

| 項目 | 実現方式 |
| ---- | -------- |
| プッシュ通知基盤 | 例: Firebase Cloud Messaging(Android)/ APNs(iOS)を React Native ラッパー経由で利用 |
| ディープリンク | URLスキーム / Universal Links・App Links の設計方針、遷移先画面IDとのマッピング |
| アプリバージョン管理 | 起動時のバージョンチェックAPI呼び出し、強制アップデート画面の表示条件 |
| ストア申請体制 | 審査提出・却下時の対応フロー、審査対象環境(stg/prod)、リリースノート作成者 |
| 多言語対応 | 要件定義書「3.4 対応言語」に基づく実装方式(例: react-i18next。翻訳リソースの管理場所) |

### 3.3 Web / Mobile 共通化方針

<!-- packages/shared のような共通パッケージに何を寄せるか(型定義、API クライアント、
     バリデーションロジック等)。React と React Native で分離が必要な部分の切り分け方針 -->

例: API型定義・APIクライアント・フォームバリデーションルールは `packages/shared` に集約する。UIコンポーネントはWeb/Mobileで別実装とし、見た目に依存しないロジック(カスタムフック)のみ共通化する。

