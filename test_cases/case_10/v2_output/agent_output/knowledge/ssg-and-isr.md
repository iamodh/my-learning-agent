# SSG와 ISR

SSG(Static Site Generation)는 빌드 시점에 페이지를 정적 HTML로 미리 생성해두는 방식이다. 블로그처럼 콘텐츠가 자주 바뀌지 않는 사이트에 잘 맞는다.

## generateStaticParams

동적 라우트에서 SSG를 쓰려면 `page.tsx`에서 `generateStaticParams` 함수를 export한다. 이 함수가 모든 slug 목록을 반환하면, 빌드 시 Next.js가 각 slug마다 정적 HTML을 만들어준다.

## 재빌드

기본적으로 새 글을 추가하면 다시 빌드해야 반영된다. 개인 블로그에서는 글을 올릴 때마다 푸시해 재배포하는 흐름이 보통 더 깔끔하다.

## ISR

ISR을 쓰면 일정 시간마다 백그라운드에서 페이지를 재생성할 수 있다.

```ts
export const revalidate = 60; // 60초마다 재생성 시도
```
