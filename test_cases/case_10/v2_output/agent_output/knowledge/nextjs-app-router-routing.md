# Next.js App Router 라우팅

Next.js App Router는 `app/` 디렉토리의 폴더 구조가 그대로 URL 경로가 되는 파일 기반 라우팅 방식이다. 별도의 라우터 설정 파일을 만들지 않고, 약속된 파일명(`page.tsx`, `layout.tsx` 등)이 정해진 역할을 한다.

## 정적 경로

`app/about/page.tsx` 파일을 만들면 `/about` 경로가 자동으로 생성된다.

## 동적 경로

폴더 이름을 대괄호로 감싸면 동적 세그먼트가 된다. 예를 들어 `app/posts/[slug]/page.tsx`는 `/posts/hello`, `/posts/foo` 같은 임의의 경로를 받을 수 있고, 컴포넌트는 `params`로 그 값을 받아 쓴다.

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
