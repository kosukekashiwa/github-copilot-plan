# データベース設計: <プロジェクト名>

| 項目 | 内容 |
| ---- | ---- |
| 関連文書 | 基本設計書: 02_backend-design.md |
| 版数 | v0.1 |
| ステータス | Draft / レビュー中 / 承認済 |

<!-- 対象スタック: API(Node.js + Express) / DB -->

## 1. データベース設計(論理)

### 1.1 ER図

<!-- テーブル間のリレーションを図示 -->

例:
```
users 1 ─── N orders
orders 1 ─── N order_items
order_items N ─── 1 products
```

### 1.2 テーブル一覧

| テーブル名 | 概要 | 主なカラム | 備考 |
| ---------- | ---- | ---------- | ---- |
| users | 会員情報 | id, email, password_hash, created_at | |
| | | | |

## 2. データベース物理設計

### 2.1 テーブル定義書

<!-- テーブルごとに作成 -->

#### テーブル名: `users`

| カラム名 | 型 | NULL許容 | デフォルト | 制約 | 説明 |
| -------- | -- | -------- | ---------- | ---- | ---- |
| id | uuid | NO | gen_random_uuid() | PK | 会員ID |
| email | varchar(255) | NO | | UNIQUE | ログインに使用するメールアドレス |
| password_hash | varchar(255) | NO | | | bcryptによるハッシュ値 |
| created_at | timestamptz | NO | now() | | 登録日時 |

インデックス:

| インデックス名 | 対象カラム | 種別 |
| --------------- | ---------- | ---- |
| users_email_idx | email | UNIQUE |

### 2.2 マイグレーション方針

<!-- マイグレーションツール、命名規則、ロールバック方針 -->

例: `node-pg-migrate` を使用。ファイル名は `<timestamp>_<変更内容>.js`(例: `20260727090000_create_users_table.js`)。本番適用前に必ず down マイグレーションでロールバック可能なことを確認する。
