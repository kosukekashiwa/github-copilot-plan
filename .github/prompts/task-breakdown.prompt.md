---
agent: task-splitter
description: 設計ドキュメントをタスク分解する(task-splitter エージェントで実行)
---

指定された設計ドキュメントを読み、実装タスクに分解してください。

- 分解ルール・出力形式は task-splitter エージェントの定義(`.github/agents/task-splitter.agent.md`)に従うこと
- 対象: ${input:doc:docs/design/ 配下の設計ドキュメントのパスを入力}
- 出力後、私が確認したら Issue として起票する
