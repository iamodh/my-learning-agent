# SSG와 ISR

SSG(Static Site Generation)는 빌드 시점에 페이지를 정적 HTML로 미리 만들어두는 방식이다. 블로그처럼 글이 자주 바뀌지 않는 사이트에 잘 맞는다. 동적 경로에서 어떤 값으로 정적 페이지를 만들지 알려주려면 `generateStaticParams`를 export하면 된다. 이 함수가 반환한 slug 목록만큼 Next.js가 빌드할 때 각 페이지를 정적 HTML로 미리 생성한다.

기본적으로 한 번 빌드된 페이지는 그대로 유지되므로, 새 글이 추가되면 다시 빌드해서 배포해야 반영된다.

## ISR로 주기적 재생성

ISR(Incremental Static Regeneration)은 일정 시간마다 백그라운드에서 페이지를 다시 생성하게 하는 옵션이다. 페이지 파일에서 `revalidate` 값을 export하면 그 초 단위마다 재생성을 시도한다.

```ts
export const revalidate = 60; // 60초마다 재생성 시도
```

개인 블로그라면 글을 올릴 때마다 푸시해서 재빌드하는 쪽이 보통 더 깔끔하다.
