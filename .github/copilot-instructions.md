# Copilot Instructions

このリポジトリで作業するすべての Copilot エージェント(Chat / coding agent / code review)への共通指示。

## プロジェクト概要

- モノレポ構成: `apps/web`(React)、`apps/mobile`(React Native)、`packages/shared`(共通ロジック・型)
- 言語: TypeScript(strict モード)。JavaScript の新規追加は禁止
- 状態管理: <!-- 例: Zustand / Redux Toolkit / Jotai -->
- API 通信: <!-- 例: TanStack Query + axios -->
- テスト: Jest + React Testing Library / React Native Testing Library
- Lint / Format: ESLint + Prettier(設定はリポジトリルート)

※ 上記はプロジェクトに合わせて書き換えてください。**実際のディレクトリ構成・コマンドと一致していること**が最重要です。

## セットアップと検証コマンド

コード変更後は必ず以下を実行し、すべて成功することを確認してから PR を作成・更新すること。

```bash
npm ci                # 依存インストール(初回のみ)
npm run lint          # ESLint
npm run typecheck     # tsc --noEmit
npm run test          # Jest(変更に関連するテストは必ず実行)
```

## コーディング規約(共通)

- 関数コンポーネント + Hooks のみ。クラスコンポーネントは書かない
- コンポーネントは 1 ファイル 1 コンポーネントを原則とし、200 行を超えたら分割を検討
- `any` の使用禁止。やむを得ない場合は `unknown` + 型ガード
- 共有できるロジック・型は `packages/shared` に置き、web / mobile から重複実装しない
- ユーザー向け文言はハードコードせず i18n 経由 <!-- i18n 未導入なら削除 -->
- 命名: コンポーネントは PascalCase、hooks は `useXxx`、ファイル名はコンポーネントと一致させる

## PR 作成時のルール(coding agent 向け)

- PR は小さく保つ。1 PR = 1 Issue(タスク)を原則とする
- PR 説明には「対応 Issue へのリンク」「変更概要」「テスト方法」「スクリーンショット(UI 変更時)」を含める
- 破壊的変更・依存追加を行う場合は PR 説明に明記する
- Issue に受け入れ条件(Acceptance Criteria)がある場合、すべて満たすこと。満たせない場合は PR 説明に理由を書く

## やってはいけないこと

- `main` への直接コミット
- テスト・lint の失敗を残したまま「完了」とすること
- Issue のスコープ外のリファクタリング(必要なら別 Issue を提案する)
- 秘密情報(API キー等)のハードコード
