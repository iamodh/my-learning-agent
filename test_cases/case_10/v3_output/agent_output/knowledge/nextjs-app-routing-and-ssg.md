# Next.js App Router와 SSG

Next.js의 최신 라우팅은 `app` 디렉토리의 폴더 구조가 URL 경로가 되는 방식이다. 별도 라우터 설정 파일을 만들지 않고, `page.tsx`, `layout.tsx` 같은 약속된 파일명이 역할을 가진다.

## 파일 기반 라우팅

`app/about/page.tsx`를 만들면 `/about` 경로가 자동으로 생긴다.

## 동적 라우팅

블로그 글처럼 `/posts/[slug]` 형태의 경로는 폴더 이름을 대괄호로 감싸서 만든다.

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

`/posts/hello`로 접근하면 `slug` 값은 `"hello"`가 된다.

## SSG

개인 블로그에는 빌드 시점에 페이지를 미리 만들어두는 SSG가 잘 맞는다. `generateStaticParams`를 export하고 모든 `slug` 목록을 반환하면, Next.js가 빌드할 때 각 글 페이지를 정적 HTML로 생성한다.

새 글을 추가하면 기본적으로 다시 빌드해서 배포해야 반영된다. ISR도 검토했지만, 개인 블로그에서는 글을 올릴 때마다 푸시해서 재빌드하는 방식이 더 깔끔한 흐름으로 정리됐다.

## ISR

ISR은 일정 시간마다 백그라운드에서 페이지를 다시 생성하게 하는 방식이다.

```tsx
export const revalidate = 60;
```

위처럼 설정하면 60초마다 재생성을 시도한다.
