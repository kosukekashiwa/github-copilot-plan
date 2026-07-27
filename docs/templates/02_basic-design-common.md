# 基本設計書(外部設計)- 共通: <プロジェクト名>

| 項目 | 内容 |
| ---- | ---- |
| 関連文書 | 要件定義書: 01_requirements-definition.md / フロントエンド基本設計書: 02_basic-design-frontend.md / バックエンド基本設計書: 02_basic-design-backend.md |
| 版数 | v0.1 |
| ステータス | Draft / レビュー中 / 承認済 |

フロントエンド・バックエンド両方の基本設計書から参照する共通内容を集約したドキュメント。フロントエンド固有の設計は [02_basic-design-frontend.md](02_basic-design-frontend.md)、バックエンド固有の設計は [02_basic-design-backend.md](02_basic-design-backend.md) を参照。

## 1. システム構成(全体)

### 1.1 全体構成図

<!--
例:
[React Web SPA] ──┐
                   ├─→ [Express API] ──→ [DB]
[React Native App]─┘        │
                             └─→ [外部連携システム]
-->

層ごとの実行環境・ホスティング先は、[02_basic-design-frontend.md](02_basic-design-frontend.md)「1.2 フロントエンド実行環境」/ [02_basic-design-backend.md](02_basic-design-backend.md)「1.2 バックエンド実行環境」にそれぞれ記載する。

### 1.2 環境一覧

| 環境 | 用途 | URL | 備考 |
| ---- | ---- | --- | ---- |
| local | 開発者ローカル | http://localhost:3000 | 各自のPC上で起動 |
| dev | 開発環境 | https://dev.example.com | 開発中の最新コードを自動デプロイ |
| stg | 検証環境 | https://stg.example.com | 本番相当データでの受入テスト用 |
| prod | 本番環境 | https://example.com | エンドユーザー向け |

