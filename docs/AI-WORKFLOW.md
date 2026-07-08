# AI 協働開発ワークフロー運用ガイド

GitHub Copilot Business を活用した「設計壁打ち → タスク分解 → 実装 → コードレビュー → 指摘修正」の運用手順。

## 0. 初期設定(1 回だけ)

### Organization 設定(管理者)
1. Org 設定 → Copilot → Policies で以下を有効化
   - **Copilot coding agent**(Issue を Copilot に割り当てる機能)
   - **Copilot code review**
   - 必要に応じてモデルのポリシー(Claude / GPT など)
2. リポジトリ設定 → Rules → Rulesets で `main` ブランチに:
   - PR 必須、承認 1 名以上
   - **「Request pull request review from Copilot」を有効化**(PR 作成時に自動レビュー)

### リポジトリ側
1. この kit の `.github/` と `docs/` をリポジトリにコピー
2. `copilot-instructions.md` の技術スタック・コマンドを **実際のプロジェクトに合わせて書き換える**(ここが品質の 8 割を決める)
3. `copilot-setup-steps.yml` の Node バージョン・インストール手順を合わせる
4. CI(lint / typecheck / test)が PR で走ることを確認

---

## 1. 機能設計の壁打ち

1. `💡 機能設計(壁打ち)` テンプレートで Issue を起票
2. VS Code の Copilot Chat(agent モード)で `/design-session` を実行し、Issue の内容を貼る
   - github.com の Copilot Chat(immersive モード)でリポジトリを指定して行ってもよい
3. Copilot が既存コードを踏まえて複数案を提示 → 議論して案を決定
4. `docs/design/YYYYMMDD-機能名.md` を **PR として提出** し、人間がレビュー・承認
   - 設計を PR に残すことで、後続のタスク分解・実装・レビューすべての参照点になる

## 2. タスク分解

1. 承認済み設計ドキュメントを対象に `/task-breakdown` を実行
2. 出力されたタスク案を確認・調整(粒度: 1 タスク = 1 PR)
3. `🤖 実装タスク` テンプレートで子 Issue を起票(github.com の Copilot に「この内容で Issue を作って」と依頼してもよい)
4. 各 Issue に「Copilot 委任」or「人間実装」のラベルを付ける

## 3. 実装

### Copilot に委任する場合(coding agent)
1. Issue の Assignee に **Copilot** を割り当てる(または PR/Issue で `@copilot` にメンション、github.com/copilot/agents から依頼)
2. Copilot が環境をセットアップし、実装 → lint/test を実行 → **Draft PR** を作成し、あなたをレビュアーに追加
3. セッションログで作業過程を確認できる

### 人間が実装する場合
- VS Code の Copilot(agent モード)と対話しながら実装。`copilot-instructions.md` と `*.instructions.md` が自動で適用される

## 4. コードレビュー

1. PR 作成時、ruleset により **Copilot code review が自動実行**(`code-review.instructions.md` の観点で日本語コメント)
2. Copilot のレビューは一次フィルタ。**人間のレビュー・承認は必須**(Copilot の承認はマージ要件を満たさない)
3. 人間レビュアーは、Copilot が拾いにくい観点(仕様との整合、UX、設計判断)に集中する

## 5. レビュー指摘の修正

- **Copilot が作った PR**: レビューコメントを書き「Submit review」→ `@copilot` メンション付きコメントで修正依頼すると coding agent が追いコミットする
- **人間の PR に対する Copilot の指摘**: コメントの「Implement suggestion」ボタン、または IDE で修正
- 修正 push 後、Copilot に再レビューを依頼(Reviewers の Copilot 横の再リクエストボタン / ruleset で「Review new pushes」を有効化すれば自動)

---

## エージェントの使い分け

`.github/agents/` の 4 エージェントは「設計 → 分解 → 実装 → レビュー」のフェーズに対応し、
フェーズごとにツール権限を絞ることで事故(設計中に勝手にコードを書く等)を防ぎます。

| エージェント | 役割 | 編集 | 実行 | 次へのハンドオフ |
| ------------ | ---- | :--: | :--: | ---------------- |
| planner | 設計壁打ち | ✕ | ✕ | task-splitter |
| task-splitter | タスク分解 | ✕ | ✕ | implementer |
| implementer | 実装 | ◯ | ◯ | reviewer |
| reviewer | ローカルレビュー | ✕ | ◯ | — |

- 呼び出し: VS Code はチャットのエージェントピッカー、CLI は `/agent` で選択。
  `/design-session` `/task-breakdown` は対応エージェントを自動で使う
- reviewer は「PR に出す前のセルフレビュー」用。PR 上の自動レビュー(Copilot code review)とは別物で、
  観点は `code-review.instructions.md` と揃えてある
- tools のツール名は環境(VS Code / CLI / Visual Studio)で異なることがあるため、
  動かない場合はチャットのツール一覧を見て frontmatter を調整する

## 運用のコツ

- **指示ファイルは育てる**: レビューで同じ指摘が繰り返されたら `code-review.instructions.md` に追加。Copilot の実装が規約を外したら `copilot-instructions.md` に追記(追記は該当ファイルの末尾に行うとプロンプトキャッシュが効きやすく、入力トークン削減になる。詳細は `copilot-instructions.md` の「トークン削減の運用ルール」を参照)
- **出力トークンも削減する**: `copilot-instructions.md` の「トークン削減の運用ルール」で、会話的な応答を簡潔にする caveman プロンプトを定義済み。設計ドキュメントや Issue/PR 本文など読みやすさが必要な成果物には適用しない
- **委任するタスクを選ぶ**: 型定義・CRUD・テスト追加・既存パターンの横展開は委任向き。アーキテクチャ判断やネイティブ絡みは人間が持つ
- **Issue の受け入れ条件が命**: coding agent の成果物品質は Issue の具体性に比例する
- **premium requests に注意**: coding agent / code review はプレミアムリクエストを消費する。Org の使用量ダッシュボードを定期確認

## ファイル構成

```
.github/
├── copilot-instructions.md            # 全エージェント共通の指示(最重要)
├── instructions/
│   ├── react-web.instructions.md      # apps/web にのみ適用
│   ├── react-native.instructions.md   # apps/mobile にのみ適用
│   └── code-review.instructions.md    # code review 専用(coding agent からは除外)
├── agents/
│   ├── planner.agent.md               # 設計壁打ち(編集ツールなし=コードを書けない)
│   ├── task-splitter.agent.md         # タスク分解(編集ツールなし)
│   ├── implementer.agent.md           # 実装(フルツール)
│   └── reviewer.agent.md              # ローカルレビュー(読み取り+実行のみ)
├── prompts/
│   ├── design-session.prompt.md       # /design-session → planner で壁打ち開始
│   └── task-breakdown.prompt.md       # /task-breakdown → task-splitter で分解
├── ISSUE_TEMPLATE/
│   ├── feature-design.md              # 設計壁打ち用
│   └── copilot-task.md                # 実装タスク用(Copilot 委任前提の項目構成)
├── PULL_REQUEST_TEMPLATE.md
└── workflows/
    └── copilot-setup-steps.yml        # coding agent の実行環境セットアップ
docs/
└── design/
    └── TEMPLATE.md                    # 設計ドキュメントのテンプレート
```
