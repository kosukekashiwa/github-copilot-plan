---
name: frontend-mobile-testing
description: React Native（Mobile）のテストコード（Vitest + Testing Library）を作成・修正する際に使用する。画面・コンポーネントのレンダリングテスト、ナビゲーションを含むテストの書き方を提供する。FE-Mobile対象Issueのテスト作成で使う。
---

# React Native（Mobile）テスト方針（Vitest + Testing Library）

## 前提と注意点

- テストランナー: Vitest、コンポーネントテスト: `@testing-library/react-native`
- **注意**: React Native公式のテスト環境は歴史的にJest（`jest-expo` / `react-native` プリセット）が標準であり、VitestでReact Nativeをテストする構成は情報・実績が少ない。以下の設定が必要になる場合がある。
  - `react-native` のモジュールをVitest上で解決するための設定（`vite.config.ts` でのエイリアス・トランスフォーム設定）
  - `@testing-library/react-native` がVitest環境で問題なく動作するかの事前検証
  - 動作しない場合は、Mobile側のみJestに切り替える、または動作実績のあるプリセットを別途調査する判断が必要になる可能性がある
- 実装を進める中でVitestでの実行に問題が出た場合は、憶測で回避策を実装せず、ユーザーに状況を報告し方針を確認すること。

## コンポーネントテスト

- Web側同様、ユーザーの見た目・操作からテストする。`getByText` / `getByRole` を優先する。

```tsx
import { render, screen, fireEvent } from '@testing-library/react-native';
import { describe, it, expect, vi } from 'vitest';
import { PostListItem } from './PostListItem';

describe('PostListItem', () => {
  it('タップするとonPressがpostIdと共に呼ばれる', () => {
    const onPress = vi.fn();
    const post = { id: 'post-1', body: 'hello' };

    render(<PostListItem post={post} onPress={onPress} />);
    fireEvent.press(screen.getByText('hello'));

    expect(onPress).toHaveBeenCalledWith('post-1');
  });
});
```

## ナビゲーションを含むテスト

- ナビゲーションコンテナ・ナビゲーターをテスト用にラップするヘルパー（`renderWithNavigation`など）を用意し、画面遷移パラメータの受け渡しをテストする。
- 実際の画面遷移よりも「特定のパラメータで画面が正しくレンダリングされるか」を優先してテストする。

## 共有ロジック（Web共通部分）のテスト

- storeやAPIクライアントなど、Web側と共通化した部分は共有パッケージ側のテスト（`frontend-web-testing` Skillの方針）でカバーされている前提とし、Mobile側では重複テストしない。UIとの結合部分のみをMobile側でテストする。

## プラットフォーム差分のテスト

- `*.ios.ts` / `*.android.ts` のように分岐した実装がある場合、両方の分岐をテストできるようモックを使い分ける。

## テスト作成後のセルフチェック

- Web側テストと重複した内容になっていないか
- オフライン・エラー時の挙動をテストできているか
- Vitest特有の設定不備でテストがそもそも実行できていない、ということがないか
