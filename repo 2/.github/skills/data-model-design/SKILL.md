---
name: data-model-design
description: データモデル・TypeScript型定義・Zodスキーマの設計を行う際に使用する共通Skill。FE-Web/FE-Mobile/BEいずれの設計でも、データ構造そのものを設計するときに使う。
---

# データモデル・型定義 設計方針

このSkillは、FE/BEを問わず共通で使う、データモデルとTypeScript型定義の設計ガイドです。
`frontend-web-design` / `frontend-mobile-design` / `backend-api-design` の各Skillと組み合わせて使用する。

## 使うタイミング

- 新しいエンティティ・データ構造を設計するIssue
- FE/BEで共有するリクエスト/レスポンス型を設計するIssue

## 設計の進め方

1. 対象のエンティティ（例: `Post`, `Survey`, `SurveyAnswer`）を洗い出し、各エンティティが属するドメインを明確にする。
2. 各エンティティについて、必須/任意フィールド、他エンティティとの関連（1対多・多対多など）を整理する。
3. マルチテナント対応のため、テナントに紐づくエンティティには必ず `tenantId` を含める。
4. 多言語対応が必要なフィールド（例: 表示名、本文）は、言語コードをキーとした構造にするか、翻訳リソースを別テーブル/別構造で持つかを検討し、Issueの要件に応じて選択する。

## Zodスキーマ + `z.infer` を基本とする

- データの検証と型定義を二重管理しないよう、Zodスキーマを正とし、型は `z.infer` で導出する。
  ```typescript
  import { z } from 'zod';

  const PostSchema = z.object({
    id: z.string().uuid(),
    tenantId: z.string().uuid(),
    authorId: z.string().uuid(),
    body: z.string().min(1).max(1000),
    createdAt: z.string().datetime(),
  });

  type Post = z.infer<typeof PostSchema>;
  ```
- FE/BEで共有するスキーマは、モノレポ内の共有パッケージ（例: `packages/schemas`）に配置し、両者から参照する。

## `interface` と `type` の使い分け（プロジェクト方針）

- `z.infer<typeof XxxSchema>` などZodから導出する型、およびUnion型は **`type`** を使う。
- Reactコンポーネントのprops、特にHTML属性や既存の型を拡張するものは **`interface`** を使う（`frontend-web-design` / `frontend-mobile-design` Skill参照）。
- ドメインのエンティティ自体はZod由来の `type` を正とし、UIコンポーネント側で追加のprops型が必要な場合のみ `interface` で拡張する。

## 設計案を出すときの観点

- 多言語フィールドの持ち方（フィールド内多言語構造 vs 翻訳テーブル分離）とそのトレードオフ
- FE/BEでスキーマを共有パッケージ化するか、ドメインごとに別々に定義するか
- エンティティの正規化度合い（過剰な正規化によるJOINコスト増 vs 非正規化による整合性リスク）
