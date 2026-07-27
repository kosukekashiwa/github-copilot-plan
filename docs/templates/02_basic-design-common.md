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
| local | 開発者ローカル | | |
| dev | 開発環境 | | |
| stg | 検証環境 | | |
| prod | 本番環境 | | |

## 2. 機能設計(概要)

<!-- FE/BE双方から参照するため、本表はここに一元管理する -->

要件定義書([01_requirements-definition.md](01_requirements-definition.md))の機能要件ごとに、実現方式の概要を記載する。詳細は [03_detailed-design-frontend.md](03_detailed-design-frontend.md) / [03_detailed-design-backend.md](03_detailed-design-backend.md) に譲る。

| 機能ID | 機能名 | 処理概要 | 関連画面 | 関連API |
| ------ | ------ | -------- | -------- | ------- |
| F-001 | | | S-001 | API-001 |

