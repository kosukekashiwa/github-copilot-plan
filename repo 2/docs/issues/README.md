# docs/issues の運用ルール

Issueごとに連番のフォルダを作成し、その配下に `issue.md` を配置する。

```
docs/issues/
  1/
    issue.md
  2/
    issue.md
```

- `issue.md` は `docs/templates/issue-template.md` をコピーして作成する。
- VSCode上でGitHub Copilot Chatの「設計壁打ち」agentを呼び出す際、対象のIssue番号（フォルダ名）を伝える。
  - 例: 「issue 2 の設計を壁打ちしたい」
- 該当フォルダ・`issue.md` が存在しない場合、agentは `docs/templates/issue-template.md` の項目に沿ってチャット上で直接ヒアリングする。
- agentが提案した設計（`docs/templates/design-template.md` 形式）は、チャット出力後に開発者自身が `docs/issues/{数字}/design.md` などとして保存する（agentはファイルを直接編集しない）。
