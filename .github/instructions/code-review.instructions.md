---
applyTo: "**"
excludeAgent: "cloud-agent"
---

# Copilot Code Review 向け指示

レビューコメントは日本語で書くこと。

## 重点的にチェックする観点(優先順)

1. **バグ・ロジック誤り**: null/undefined 未処理、非同期処理の競合、依存配列の漏れ
2. **セキュリティ**: 秘密情報のハードコード、外部入力の未検証、危険な dangerouslySetInnerHTML
3. **型安全性**: `any` の使用、型アサーション(`as`)の乱用
4. **web / mobile の共通化漏れ**: `packages/shared` に置くべきロジックの重複実装
5. **テスト**: 新規ロジックに対するテストの欠如、テストが実装の詳細に依存していないか
6. **アクセシビリティ**: web は semantic HTML、mobile は accessibilityLabel の欠如

## コメントのスタイル

- 指摘には「なぜ問題か」と「修正案(可能ならコード例)」をセットで書く
- 重大度を明示する: `[must]` 修正必須 / `[should]` 推奨 / `[nits]` 好みの問題
- 良い実装には言及しない(ノイズを減らす)

## 指摘しないこと

- Prettier / ESLint が自動検出できるフォーマットの問題
- Issue のスコープ外の大規模リファクタリング提案
