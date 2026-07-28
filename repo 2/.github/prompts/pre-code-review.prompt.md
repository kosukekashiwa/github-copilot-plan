---
description: commit前のstaged changesを「コードレビュー」agentでレビューする（修正は行わず、優先度と改善案を提示する）
agent: code-review
tools: ['search/codebase', 'runCommands', 'read/terminalLastCommand']
---

commit前の staged changes をレビューしてください。

1. `git diff --staged` の内容を取得する。staged changesが無ければ、その旨を報告して終了する。
2. `code-review` agentの手順（対象レイヤーの判定 → 該当Skillでのレビュー → 共通観点でのレビュー）に従い、staged changesの内容をレビューする。
3. 指摘事項は優先度（High/Medium/Low）と改善案を添えて報告する。
4. ソースコードの修正は一切行わない。修正案を示す場合もコード例の提示に留める。
