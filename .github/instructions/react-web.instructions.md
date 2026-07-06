---
applyTo: "apps/web/**/*.{ts,tsx}"
---

# React (Web) 固有ルール

- スタイリングは <!-- 例: Tailwind CSS / CSS Modules --> を使用。inline style は動的値以外禁止
- ルーティングは <!-- 例: React Router v7 --> の規約に従う
- アクセシビリティ: インタラクティブ要素は button/a を使い、div + onClick を避ける。画像には alt を必須とする
- データ取得はカスタム hooks(`useXxxQuery`)に閉じ込め、コンポーネントから fetch を直接呼ばない
- `useEffect` での手動データ取得は禁止(TanStack Query を使う)
- パフォーマンス: リストには安定した key を付与。`useMemo` / `useCallback` は計測に基づく場合のみ使用
