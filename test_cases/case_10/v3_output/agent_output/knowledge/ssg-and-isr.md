# SSG와 ISR

SSG(Static Site Generation)는 빌드 시점에 페이지를 정적 HTML로 미리 생성해두는 렌더링 방식이다. 블로그처럼 콘텐츠가 자주 바뀌지 않는 사이트에 잘 맞는다.

## generateStaticParams

동적 경로 페이지(`app/posts/[slug]/page.tsx` 등)에서 `generateStaticParams` 함수를 export하면, 거기서 반환한 모든 slug 목록에 대해 Next.js가 빌드 시점에 정적 HTML 페이지를 미리 생성한다.

## 빌드와 갱신

기본적으로 빌드할 때 만든 정적 HTML이 그대로 배포된다. 따라서 새 글이 추가되면 다시 빌드해서 배포해야 반영된다.

## ISR

ISR(Incremental Static Regeneration)은 일정 시간마다 백그라운드에서 페이지를 다시 생성하게 하는 옵션이다.

```ts
export const revalidate = 60;
```

이렇게 적어두면 60초마다 재생성을 시도한다. 다만 개인 블로그처럼 글이 잦지 않은 경우에는 글을 올릴 때마다 푸시해서 재빌드하는 쪽이 보통 더 깔끔하다.
