# Next.js App Router와 파일 기반 라우팅

Next.js의 App Router는 `app` 디렉토리 안의 폴더 구조가 곧 URL이 되는 파일 기반 라우팅 방식이다. 별도의 라우터 설정 파일을 만들지 않고, `page.tsx`·`layout.tsx` 같은 약속된 파일명이 각자의 역할을 갖는다.

## 기본 규칙

- `app/about/page.tsx` 를 만들면 `/about` 경로가 자동으로 생긴다.
- 폴더 구조 = URL 구조.
- `page.tsx`, `layout.tsx` 같은 파일명은 약속된 역할을 가진다.

## 동적 경로

폴더 이름을 대괄호로 감싸면 동적 세그먼트가 된다. `app/posts/[slug]/page.tsx` 를 만들면 `[slug]` 자리에 어떤 값이든 들어올 수 있고, 컴포넌트 안에서 `params`로 받아 쓴다. `/posts/hello` 로 접근하면 `slug`가 `"hello"`가 된다.

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

`slug`를 키로 글 내용을 찾아오는 함수만 붙이면 글 페이지가 완성된다.
