# AI 協働開発ワークフロー運用ガイド

GitHub Copilot Business を活用した「設計壁打ち → タスク分解 → 実装 → コードレビュー → 指摘修正」の運用手順。
なお、設計判断を伴わない軽微な修整は、この工程を経ずに直接対応できる(後述「軽微な修整」を参照)。

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

## 軽微な修整(設計・タスク分解が不要な場合)

typo・文言修正・影響範囲の小さいバグ修正など、設計判断を伴わない軽微な修整は、
1〜5 の全工程を経由せず `quick-fix` エージェントで直接対応する。

1. VS Code の Copilot Chat(agent モード)で `/quick-fix` を実行し、直したい内容を伝える
2. quick-fix エージェントが直接修正 → lint/typecheck/test を実行して報告する
3. 対応範囲(`quick-fix.agent.md` 参照)を超えると quick-fix が判断した場合、
   その場で `/design-session` からの通常フローへの切り替えを提案される
4. 必要に応じて reviewer にセルフレビューを依頼してから PR を作成する

判断基準: 仕様判断や複数レイヤー(`packages/shared` / `apps/web` / `apps/mobile`)にまたがる
変更が必要になった時点で、通常の「設計壁打ち → タスク分解」フローに切り替える。

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

`.github/agents/` の 5 エージェントは「設計 → 分解 → 実装 → レビュー」のフェーズ、および
それらを飛ばす「軽微な修整」のショートカットに対応し、フェーズごとにツール権限を絞ることで
事故(設計中に勝手にコードを書く等)を防ぎます。

| エージェント | 役割 | 編集 | 実行 | 次へのハンドオフ |
| ------------ | ---- | :--: | :--: | ---------------- |
| planner | 設計壁打ち | ✕ | ✕ | task-splitter |
| task-splitter | タスク分解 | ✕ | ✕ | implementer |
| implementer | 実装(Issue ベース) | ◯ | ◯ | reviewer |
| quick-fix | 軽微な修整の直接実装(設計・分解なし) | ◯ | ◯ | reviewer |
| reviewer | ローカルレビュー | ✕ | ◯ | — |

- 呼び出し: VS Code はチャットのエージェントピッカー、CLI は `/agent` で選択。
  `/design-session` `/task-breakdown` `/quick-fix` は対応エージェントを自動で使う
- implementer と quick-fix はどちらも実装用だが、粒度で使い分ける:
  Issue の受け入れ条件に沿って実装するなら implementer、
  Issue 起票すら不要な typo・小さなバグ修正なら quick-fix
- reviewer は「PR に出す前のセルフレビュー」用。PR 上の自動レビュー(Copilot code review)とは別物で、
  観点は `code-review.instructions.md` と揃えてある
- tools のツール名は環境(VS Code / CLI / Visual Studio)で異なることがあるため、
  動かない場合はチャットのツール一覧を見て frontmatter を調整する
- **model はコスト抑制のために意図的に差を付けている**: 機械的な工程(task-splitter, reviewer)は軽量モデル、
  判断の質が問われる工程(planner, implementer)は中位以上のモデルを既定にしている。
  モデル名の表記は環境ごとに異なり、利用可能なモデルも時期によって変わるため、
  実際に使う前にチャットのモデルピッカーで名前を確認し、各 `.agent.md` の `model:` を書き換えること

## 運用のコツ

- **指示ファイルは育てる**: レビューで同じ指摘が繰り返されたら `code-review.instructions.md` に追加。Copilot の実装が規約を外したら `copilot-instructions.md` に追記(追記は該当ファイルの末尾に行うとプロンプトキャッシュが効きやすく、入力トークン削減になる。詳細は `copilot-instructions.md` の「トークン削減の運用ルール」を参照)
- **出力トークンも削減する**: `copilot-instructions.md` の「トークン削減の運用ルール」で、会話的な応答を簡潔にする caveman プロンプトを定義済み。設計ドキュメントや Issue/PR 本文など読みやすさが必要な成果物には適用しない
- **委任するタスクを選ぶ**: 型定義・CRUD・テスト追加・既存パターンの横展開は委任向き。アーキテクチャ判断やネイティブ絡みは人間が持つ
- **Issue の受け入れ条件が命**: coding agent の成果物品質は Issue の具体性に比例する
- **AI Credits に注意**: coding agent / code review はトークン消費に応じた GitHub AI Credits を消費する(2026年6月〜、旧プレミアムリクエストから移行)。詳細は「6. コストを抑える運用」を参照

## 6. コストを抑える運用

2026年6月以降、Copilot の課金は「プレミアムリクエスト数」から**トークン消費量に応じた GitHub AI Credits**に変わった。コード補完は引き続き無課金・無制限だが、Chat・coding agent・code review はすべてクレジットを消費する。加えて code review は Actions 分数も別途消費する。以下の工夫で消費を抑える。

### Ruleset の設定を絞る(効果が大きい)

- **「Review new pushes」は必要な場合だけ ON にする**
  場所: `Repo → Settings → Rules → Rulesets` → 対象ルールを開く →
  「Automatically request Copilot code review」のサブオプション
  - ON: push のたびに自動で再レビューが走る(便利だが、小さな修正を都度 push するチームだと消費が増えやすい)
  - OFF: 初回 PR 作成時に 1 回だけレビュー。以降は必要なタイミングで手動リクエスト
  - WIP コミットが多いリポジトリでは OFF、レビュー前提で 1 コミット単位の PR が多いリポジトリでは ON、が目安
- **Draft PR レビューは目的を絞る**
  同じ画面の「Review draft pull requests」。coding agent が作る Draft PR は
  「Ready for review にした 1 回だけレビュー」で足りることが多く、Draft 中の頻繁な変更にも
  レビューさせると倍コストになりやすい
- **Review effort level は Low を既定にする**
  場所: `Repo → Settings → Copilot → Code review` → Review effort level
  - Low: 標準レビュー(既定)
  - Medium: ロジック・セキュリティ・サービス横断の変更をより深く分析するが、Actions 分数と AI Credits の消費が増える
  - 決済・認証など重要なディレクトリのみ Medium にする、といった使い分けが有効

### Issue とタスクの粒度を絞る(task-splitter エージェントの役割)

- 受け入れ条件が曖昧な Issue は、coding agent が試行錯誤(＝トークン消費)を繰り返す最大の原因になる
- `/task-breakdown`(task-splitter エージェント)で「検証可能な受け入れ条件」「スコープ外の明記」を徹底したタスクほど、
  implementer が一発で収束しやすく、結果的に消費も抑えられる
- 逆に、曖昧なままアサインするのは最もコストがかさむパターンなので避ける

### instructions ファイルは軽量・スコープを絞る

- `copilot-instructions.md` や `*.instructions.md` は呼び出しのたびに入力トークンとして読み込まれ、コストに乗る
- 長大な 1 ファイルにせず、`applyTo` でパスごとに分割する(この kit の `react-web` / `react-native` / `code-review` の分割構成はこの対策を兼ねている)
- `excludeAgent` で不要なエージェントに読ませない設定も、無駄なトークン読み込みを減らす
  (`code-review.instructions.md` は既に coding agent から除外済み)

### モデル選択・プロンプトの出し方(Chat / agent モード利用時)

- 確認・軽い質問は included モデル(0× 相当)や Auto モードに回し、Claude Opus のような重いモデルは
  設計判断など本当に必要な場面だけに使う
- VS Code の Copilot Chat で「Auto」を選ぶと、対象モデルの倍率に割引が適用され、多くの場合軽量モデルが自動選択される
- Agent Mode の内部ループ(ファイル編集・実行・エラー修正)はユーザーが送ったプロンプトの分しか課金対象にならない。
  細かく指示を出し直すより「A〜D を仕様に沿って一気に実装して」のように一括指示し、途中で止めずに完了まで走らせる方が安く済む

### 可視化と予算で歯止めをかける

- `Organization → Settings → Billing → Usage` の使用量レポート(CSV)で per-user・per-model の消費を確認し、
  突出して使っているメンバーやリポジトリを定期的に特定する
- ユーザー単位のデフォルト予算を設定し、パワーユーザーには個別に上限緩和、消費の荒いチームはコストセンターで分離する
- 予算が「アラートのみ」か「実際に停止するか」を必ず確認しておく

### 優先順位の目安

即効性が高い順に: ①「Review new pushes」を必要な場合だけ ON にする → ② Issue の受け入れ条件を具体化する(task-splitter を活用) → ③軽い作業は軽量モデル/Auto に回す → ④ instructions の軽量化 → ⑤予算設定を仕組み化する。

---

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
│   ├── quick-fix.agent.md             # 軽微な修整の直接実装(設計・分解なし、フルツール)
│   └── reviewer.agent.md              # ローカルレビュー(読み取り+実行のみ)
├── prompts/
│   ├── design-session.prompt.md       # /design-session → planner で壁打ち開始
│   ├── task-breakdown.prompt.md       # /task-breakdown → task-splitter で分解
│   └── quick-fix.prompt.md            # /quick-fix → quick-fix で軽微な修整を直接実装
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
