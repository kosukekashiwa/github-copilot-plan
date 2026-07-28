---
name: backend-testing
description: Node.js/Express APIのテストコード（Jest + Supertestを基本とする一般的な想定。プロジェクトのBEテストフレームワークは正式決定していないため、決定後は本Skillの更新が必要）を作成・修正する際に使用する。service単体テスト、Supertestによる結合テストの書き方を提供する。BE対象Issueのテスト作成で使う。
---

# BE（Node.js / Express）テスト方針

## 前提

- 本Skillは、プロジェクトのBEテストフレームワークが正式決定するまでの暫定として、一般的に広く使われる **Jest + Supertest** を想定して書かれている。プロジェクトで正式に決定した際は、このSkillを実際のフレームワークに合わせて更新すること。
- テストファイルは実装ファイルと同階層、またはドメインごとの `__tests__/` に `*.test.ts` として配置する。

## serviceレイヤーの単体テスト

- repositoryをモックし、ビジネスロジックのみを検証する。

```typescript
import { createPost } from './service';
import { postRepository } from './repository';

jest.mock('./repository');

describe('createPost', () => {
  it('tenantId・authorIdを含めてrepositoryに渡す', async () => {
    // Arrange
    const createSpy = jest.spyOn(postRepository, 'create').mockResolvedValue({ id: 'post-1' } as any);

    // Act
    await createPost('tenant-1', 'user-1', { body: 'hello' });

    // Assert
    expect(createSpy).toHaveBeenCalledWith(
      expect.objectContaining({ tenantId: 'tenant-1', authorId: 'user-1', body: 'hello' })
    );
  });
});
```

## APIの結合テスト（Supertest）

- Expressアプリ全体（またはルーター単位）に対してHTTPリクエストを送り、レスポンスを検証する。

```typescript
import request from 'supertest';
import { app } from '../app';

describe('POST /api/v1/timeline/posts', () => {
  it('本文が空の場合は400を返す', async () => {
    const res = await request(app)
      .post('/api/v1/timeline/posts')
      .set('Authorization', `Bearer ${validTenantAToken}`)
      .send({ body: '' });

    expect(res.status).toBe(400);
  });
});
```

## マルチテナント分離のテスト

- テナントAのトークンで作成したリソースに、テナントBのトークンからアクセスできないことを必ずテストする。

```typescript
it('他テナントのpostにはアクセスできない', async () => {
  const post = await createPostAsTenant('tenant-a');

  const res = await request(app)
    .get(`/api/v1/timeline/posts/${post.id}`)
    .set('Authorization', `Bearer ${tenantBToken}`);

  expect(res.status).toBe(404); // または403。プロジェクトの方針に合わせる
});
```

## 非同期・イベント連携のテスト

- `ai-analysis` へのイベント発行が必要な処理では、実際のキュー/Pub-Subには送信せず、発行関数をモックして「正しいペイロードで1回呼ばれたか」を検証する。

## テスト作成後のセルフチェック

- 正常系・バリデーションエラー・認可エラー（他テナントアクセス）の3種類を最低限カバーしているか
- repositoryやイベント発行など、外部依存を適切にモックできているか
- BEのテストフレームワークが正式決定した際に、本Skill自体の更新が必要である旨を認識しているか
