---
name: frontend-web-implementation
description: React（Web）の実装コードを追加・修正する際に使用する。コンポーネント実装、Zustand store実装、API呼び出し実装の具体的な書き方を提供する。FE-Web対象Issueの実装作業で使う。
---

# React（Web）実装方針

`frontend-web-design` Skillで決まった設計を、実際のコードに落とし込むためのガイドです。

## コンポーネント実装

- 関数コンポーネント + Hooksで実装する。クラスコンポーネントは使わない。
- propsの型は `interface` で定義し、コンポーネントの直前に置く。
  ```tsx
  interface PostCardProps {
    post: Post;
    onSelect?: (postId: string) => void;
  }

  export function PostCard({ post, onSelect }: PostCardProps) {
    return (
      <div onClick={() => onSelect?.(post.id)}>
        <p>{post.body}</p>
      </div>
    );
  }
  ```
- Container（データ取得・状態購読）とPresentational（表示のみ）を分離する（`frontend-web-design` Skillの構成に従う）。

## データ取得・API呼び出し

- API呼び出しは `features/{domain}/hooks/` にカスタムフックとして実装し、コンポーネントから直接fetchを呼ばない。
  ```tsx
  export function usePosts(tenantId: string) {
    const [posts, setPosts] = useState<Post[]>([]);
    const [isLoading, setIsLoading] = useState(false);

    useEffect(() => {
      setIsLoading(true);
      fetchPosts(tenantId)
        .then(setPosts)
        .finally(() => setIsLoading(false));
    }, [tenantId]);

    return { posts, isLoading };
  }
  ```
- レスポンスの型・バリデーションは共有スキーマ（`data-model-design` Skillで定義したZodスキーマ）を使い、`XxxSchema.parse(response)` で検証してから利用する。

## 状態管理（Zustand）

- ドメインごとのslice実装は `store/{domain}Slice.ts` に置く。
  ```typescript
  export const createTimelineSlice: StateCreator<StoreState, [], [], TimelineSlice> = (set) => ({
    posts: [],
    fetchPosts: async (tenantId) => {
      const posts = await fetchPostsApi(tenantId);
      set({ posts });
    },
  });
  ```
- コンポーネントからのstore参照は、必要な値・関数のみをセレクタで取得し、不要な再レンダリングを避ける。
  ```tsx
  const posts = useStore((state) => state.posts);
  ```

## 多言語・マルチテナント対応

- 表示文言はi18nライブラリのフック（例: `useTranslation`）経由で取得し、直書きしない。
- テナント固有の設定（ロゴ・機能フラグなど）はcontext経由で取得する専用フック（例: `useTenantConfig()`）を用意し、コンポーネントに直接注入する。

## 実装後のセルフチェック

- propsの型に `any` を使っていないか
- API応答をスキーマでバリデーションしているか
- i18n・テナント考慮が漏れていないか
- 対応するテストコードを `frontend-web-testing` Skillの方針で作成したか
