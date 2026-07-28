---
name: frontend-web-testing
description: React（Web）のテストコード（Vitest + Testing Library）を作成・修正する際に使用する。コンポーネントテスト、カスタムフックテスト、Zustand storeテストの書き方を提供する。FE-Web対象Issueのテスト作成で使う。
---

# React（Web）テスト方針（Vitest + Testing Library）

## 前提

- テストランナー: Vitest
- コンポーネントテスト: `@testing-library/react`
- テストファイルは実装ファイルと同階層に `*.test.tsx` / `*.test.ts` として配置する。

## コンポーネントテスト

- 実装の詳細（内部state・class名）ではなく、ユーザーから見た振る舞いをテストする。`getByRole` / `getByText` などアクセシビリティベースのクエリを優先する。
- AAA（Arrange-Act-Assert）パターンで書く。

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { PostCard } from './PostCard';

describe('PostCard', () => {
  it('クリックするとonSelectがpostIdと共に呼ばれる', async () => {
    // Arrange
    const onSelect = vi.fn();
    const post = { id: 'post-1', body: 'hello' };

    // Act
    render(<PostCard post={post} onSelect={onSelect} />);
    await userEvent.click(screen.getByText('hello'));

    // Assert
    expect(onSelect).toHaveBeenCalledWith('post-1');
  });
});
```

## API呼び出しを含むフック・コンポーネントのテスト

- 実際のネットワーク呼び出しは行わず、`vi.fn()` でAPIクライアント関数をモックするか、MSW（Mock Service Worker）でHTTPレベルのモックを行う。
- ローディング状態・エラー状態・正常系の3パターンは最低限カバーする。

```tsx
vi.mock('./api', () => ({
  fetchPosts: vi.fn().mockResolvedValue([{ id: '1', body: 'hello' }]),
}));
```

## Zustand storeのテスト

- Reactに依存しない純粋なロジックとして、storeのslice単体をテストする。

```typescript
import { describe, it, expect } from 'vitest';
import { create } from 'zustand';
import { createTimelineSlice } from './timelineSlice';

describe('timelineSlice', () => {
  it('fetchPostsでpostsが更新される', async () => {
    const useStore = create(createTimelineSlice);
    await useStore.getState().fetchPosts('tenant-1');
    expect(useStore.getState().posts.length).toBeGreaterThan(0);
  });
});
```

## 多言語・マルチテナント観点のテスト

- i18nキーが正しく解決されているか（存在しないキーで警告が出ないか）を確認する。
- テナントごとの出し分けが必要なコンポーネントは、異なるテナント設定を渡した場合の描画差分をテストする。

## テスト作成後のセルフチェック

- 正常系だけでなく、エラー・空データなどの異常系もカバーしているか
- モックが過剰でなく、実際の振る舞いを検証できているか
- 実装の変更に追従してテストを更新したか（実装のみ変更してテストが古いまま、になっていないか）
