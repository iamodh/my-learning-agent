# Next.js App 라우팅과 SSG

Next.js의 최신 app 디렉토리는 폴더 구조 자체가 URL 경로가 되는 파일 기반 라우팅을 쓴다. 별도의 라우터 설정 파일 없이 `page.tsx`, `layout.tsx` 같은 약속된 파일명에 역할이 정해져 있다.

## 정적 경로

`app/about/page.tsx`를 만들면 `/about` 경로가 자동으로 생긴다. 폴더 = URL 세그먼트라는 규칙을 따른다.

## 동적 경로

폴더 이름을 대괄호로 감싸면 동적 세그먼트가 된다. `app/posts/[slug]/page.tsx`로 만들면 `[slug]` 자리에 어떤 값이든 들어올 수 있고, 컴포넌트에서 `params`로 그 값을 받아 쓴다. `/posts/hello`로 들어오면 `slug`는 `"hello"`가 된다.

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

## SSG와 generateStaticParams

블로그처럼 콘텐츠가 빌드 시점에 정해지는 경우 SSG(Static Site Generation)가 잘 맞는다. 동적 경로 페이지에서 `generateStaticParams`를 export해 모든 slug 목록을 반환하면, 빌드할 때 Next.js가 각 slug에 대한 페이지를 정적 HTML로 미리 생성한다.

새 글이 추가되면 다시 빌드해서 배포해야 반영된다.

## ISR (Incremental Static Regeneration)

매 변경마다 재빌드하지 않고 일정 시간마다 백그라운드에서 재생성하게 하려면 ISR을 쓴다. 페이지에서 `export const revalidate = 60`처럼 적으면 60초마다 재생성을 시도한다. 개인 블로그라면 글을 올릴 때마다 푸시해 재빌드하는 쪽이 보통 더 깔끔하다.
