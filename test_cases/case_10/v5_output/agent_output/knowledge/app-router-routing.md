# App Router 파일 기반 라우팅

Next.js의 App Router는 `app` 디렉토리 안의 폴더 구조가 그대로 URL 경로가 되는 라우팅 방식이다. 별도의 라우터 설정 파일을 만들지 않고, `page.tsx`, `layout.tsx` 같은 약속된 파일명으로 각 경로의 역할을 정의한다.

예를 들어 `app/about/page.tsx`를 만들면 `/about` 경로가 자동으로 생긴다.

## 동적 경로 (Dynamic Segments)

폴더 이름을 대괄호로 감싸면 그 자리에 어떤 값이든 들어올 수 있는 동적 경로가 된다. `app/posts/[slug]/page.tsx`로 만들면 `/posts/hello` 같은 URL이 들어왔을 때 `slug`가 `"hello"`가 되는 식이다.

페이지 컴포넌트는 `params`를 받아 그 값을 꺼내 쓴다.

```tsx
// app/posts/[slug]/page.tsx
export default async function PostPage({
  params,
}: {
  params: { slug: string };
}) {
  const { slug } = params;
  // slug로 글 데이터 가져오기
  return (
    <article>
      <h1>{slug}</h1>
    </article>
  );
}
```

`slug`를 키로 글 데이터를 찾아오는 함수만 붙이면 글 페이지가 완성된다.
