---
name: backend-implementation
description: Node.js/Expressの実装コードを追加・修正する際に使用する。controller/service/repositoryの実装、Zodバリデーション実装、テナント分離実装の具体的な書き方を提供する。BE対象Issueの実装作業で使う。
---

# BE（Node.js / Express）実装方針

`backend-api-design` Skillで決まった設計を、実際のコードに落とし込むためのガイドです。

## レイヤー実装

```typescript
// routes.ts
router.post("/posts", createPostController);

// controller.ts
export async function createPostController(req: Request, res: Response) {
  const parsed = CreatePostRequestSchema.parse(req.body);
  const post = await createPost(req.auth.tenantId, req.auth.userId, parsed);
  res.status(201).json(post);
}

// service.ts
export async function createPost(
  tenantId: string,
  authorId: string,
  input: CreatePostInput,
) {
  return postRepository.create({ tenantId, authorId, ...input });
}

// repository.ts
export const postRepository = {
  create(data: NewPost) {
    return db.posts.insert(data);
  },
};
```

- controllerはHTTPの関心事（ステータスコード、バリデーション呼び出し、レスポンス整形）のみを扱う。ビジネスロジックはserviceに書く。
- repositoryのクエリには必ず `tenantId` を条件に含める（`backend-api-design` Skillのマルチテナント方針に従う）。

## リクエスト/レスポンスのバリデーション

- リクエストボディは共有Zodスキーマ（`data-model-design` Skill）でパースし、不正なリクエストは早期に400エラーとして返す。
  ```typescript
  const CreatePostRequestSchema = z.object({
    body: z.string().min(1).max(1000),
  });
  type CreatePostInput = z.infer<typeof CreatePostRequestSchema>;
  ```
- バリデーションエラーは共通エラーハンドリングミドルウェアで捕捉し、統一フォーマット（`{ code, message, details }`）で返す。

## エラーハンドリング

- ドメイン固有のエラーはカスタムエラークラスとして定義し、Expressのエラーハンドリングミドルウェアで一元的にHTTPステータスへ変換する。
  ```typescript
  export class NotFoundError extends Error {}
  export class ForbiddenTenantAccessError extends Error {}
  ```

## 非同期処理・ドメイン間連携

- 既存のイベント発行の実装がリポジトリ内に見当たらない場合は、新規に仕組みを作る前にユーザーに確認する。

## 実装後のセルフチェック

- すべてのDBアクセスに `tenantId` 条件が入っているか
- リクエスト/レスポンスがZodスキーマでバリデーションされているか
- エラーハンドリングが共通フォーマットに沿っているか
- 対応するテストコードを `backend-testing` Skillの方針で作成したか
