# 基本設計書(外部設計)- バックエンド: <プロジェクト名>

| 項目 | 内容 |
| ---- | ---- |
| 関連文書 | 要件定義書: 01_requirements-definition.md / 基本設計書(共通): 02_basic-design-common.md / フロントエンド基本設計書: 02_basic-design-frontend.md |
| 版数 | v0.1 |
| ステータス | Draft / レビュー中 / 承認済 |

<!-- 対象スタック: API(Node.js + Express) / DB -->

全体構成図・環境一覧は [02_basic-design-common.md](02_basic-design-common.md) を参照。画面・フロントエンドの設計は [02_basic-design-frontend.md](02_basic-design-frontend.md) を参照。

## 1. システム構成

### 1.1 全体構成図

[02_basic-design-common.md](02_basic-design-common.md)「1.1 全体構成図」を参照。

### 1.2 バックエンド実行環境

Web/Mobile の実行環境は [02_basic-design-frontend.md](02_basic-design-frontend.md)「1.2 フロントエンド実行環境」を参照。

| 層 | 技術・サービス | 備考 |
| -- | -------------- | ---- |
| API | Node.js + Express | ホスティング先: 例: AWS ECS(Fargate) |
| DB | 例: PostgreSQL 16 | ホスティング先: 例: Amazon RDS |
| CI/CD | 例: GitHub Actions | lint/test/build後にstg環境へ自動デプロイ |

### 1.3 環境一覧

[02_basic-design-common.md](02_basic-design-common.md)「1.2 環境一覧」を参照。

## 2. データベース設計(論理)

### 2.1 ER図

<!-- テーブル間のリレーションを図示 -->

例:
```
users 1 ─── N orders
orders 1 ─── N order_items
order_items N ─── 1 products
```

### 2.2 テーブル一覧

| テーブル名 | 概要 | 主なカラム | 備考 |
| ---------- | ---- | ---------- | ---- |
| users | 会員情報 | id, email, password_hash, created_at | |
| | | | |

## 3. 外部インターフェース設計

### 3.1 API一覧(Express ルーティング)

| API ID | Method | パス | 概要 | 認証要否 |
| ------ | ------ | ---- | ---- | -------- |
| API-001 | POST | /api/v1/users | 会員登録を行う | 不要 |

### 3.2 外部連携システム

| 連携先 | 連携方式 | 概要 | 備考 |
| ------ | -------- | ---- | ---- |
| SendGrid | REST API | 会員登録確認メール等のメール送信 | APIキーは環境変数で管理 |
| | | | |

## 4. 非機能設計方針(バックエンド)

### 4.1 性能方針

<!-- クエリ最適化、キャッシュ層(Redis等)、コネクションプーリング、ページネーション方針 -->

フロントエンド側の性能方針は [02_basic-design-frontend.md](02_basic-design-frontend.md)「3.1 パフォーマンス方針」を参照。

### 4.2 セキュリティ方針

- 認証方式の実装方針(例: JWT + Refresh Token)
- 認可(ロールベースアクセス制御)の実装方針
- 通信経路(HTTPS強制、CORS設定)
- 入力値検証・サニタイズ方針
- 秘密情報の管理方針(環境変数、シークレットマネージャ)

### 4.3 可用性・監視方針

<!-- ヘルスチェック、冗長化、監視ツール、アラート閾値 -->

例: `/healthz` エンドポイントでヘルスチェックし、APIは2台以上で冗長化。Datadogでレスポンスタイム・エラー率を監視し、エラー率5%超過でSlack通知。

### 4.4 ログ設計方針

| ログ種別 | 出力内容 | 保存先 | 保存期間 |
| -------- | -------- | ------ | -------- |
| アクセスログ | リクエストパス・ステータスコード・応答時間 | CloudWatch Logs | 90日 |
| エラーログ | スタックトレース・リクエストID | CloudWatch Logs | 90日 |
| 監査ログ | 誰が・いつ・何を変更したか(会員情報変更等) | 専用DBテーブル | 1年 |

