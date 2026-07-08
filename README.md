# github-copilot-plan

> Github copilot と共同で、機能設計の壁打ち、タスク分解、実装、コードレビュー、レビュー指摘修正、という開発プロセスを github を活用し実施したい。
> 設定やマークダウンなど、何を用意すればよいか？まだどのように運用するのか？
> テンプレートも用意して。

テンプレート一式を作成しました。zip をリポジトリのルートに展開して、`copilot-instructions.md` の技術スタック部分を自プロジェクトに合わせて書き換えるだけで使い始められます。

## 用意したもの(役割の要点)

- `.github/copilot-instructions.md` — Chat / coding agent / code review すべてが読む共通指示。リポジトリ全体に適用されるカスタム指示で、`.github` ディレクトリ直下に置きます。ここに実際のビルド・テストコマンドを正確に書くことが coding agent の成功率を最も左右します
- `.github/instructions/*.instructions.md` — YAML frontmatter の `applyTo` で対象パスを指定でき、コードベースの部分ごとに異なる指示を与えられます。web / React Native / レビュー専用の 3 ファイルを用意。レビュー専用ファイルは `excludeAgent` キーワードで coding agent から隠し、code review だけに読ませる構成にしています
- `.github/agents/` — 壁打ち(`planner`)・タスク分解(`task-splitter`)・実装(`implementer`)・軽微な修整(`quick-fix`)・レビュー(`reviewer`)の5エージェント。フェーズごとに編集・実行ツールの権限を絞り、設計中に勝手にコードを書くといった事故を防ぎます。`quick-fix` は typo 修正など設計・分解が不要な軽微な修整を、通常フローを飛ばして直接実装するためのものです
- `.github/prompts/` — 壁打ち用 `/design-session`、分解用 `/task-breakdown`、軽微な修整用 `/quick-fix` のプロンプトファイル(VS Code の Copilot Chat でスラッシュコマンドとして呼び出せます。それぞれ対応するエージェントを自動で使います)
- Issue テンプレート 2 種(設計壁打ち用/Copilot 委任前提の実装タスク用)、PR テンプレート
- `.github/workflows/copilot-setup-steps.yml` — coding agent が作業前に依存をインストールするための必須ワークフロー
- `docs/design/TEMPLATE.md` と運用ガイド `docs/AI-WORKFLOW.md`

## 運用フローの概要(詳細は AI-WORKFLOW.md)

1. 壁打ち: 設計 Issue を起票 → `/design-session` で Copilot と複数案を比較 → 設計ドキュメントを PR にして人間がレビュー
1. タスク分解: `/task-breakdown` で「1 タスク = 1 PR」粒度の子 Issue 群を生成。型定義や CRUD は Copilot 委任、アーキテクチャ判断やネイティブ絡みは人間、と振り分け
1. 実装: Issue の Assignee に Copilot を割り当てると coding agent が Draft PR を作成。coding agent は Copilot Business では管理者がポリシーを有効化した場合に利用できますので、Org 設定の有効化が前提です
1. レビュー: ルールセットで自動コードレビューを有効化すれば PR 作成時に Copilot が自動レビューし、push ごとの再レビューも設定できます。人間の承認は必須のまま残します
1. 指摘修正: Copilot のレビューコメントの「Fix with Copilot」から修正を指示でき、同じ PR へのコミットか新規 PR かを選べます。Copilot 製 PR への指摘は @copilot メンションで追いコミットさせます

運用開始後は「レビューで繰り返される指摘 → instructions に追記」のループで指示ファイルを育てるのが定着のコツです。なお 指示は非決定的に扱われるため、少数の焦点を絞った指示から始めて実 PR で試しながら拡張していくのが公式推奨のアプローチです。
