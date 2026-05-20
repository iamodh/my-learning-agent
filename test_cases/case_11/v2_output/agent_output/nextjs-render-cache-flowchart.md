# Next 16 렌더링·캐싱 흐름도

## 만들려던 것

Next 16에서 빌드, RSC Payload 생성, HTML 생성, 데이터 패칭, revalidate, prerender, CDN 캐싱이 어떻게 이어지는지 보여주는 흐름도.

## 포함할 흐름

1. `next build`가 소스를 컴파일·번들하고 라우트를 static, dynamic, PPR 등으로 분류한다.
2. Static Generation에서는 빌드 타임에 데이터 패칭, RSC Payload 생성, HTML 생성이 완료된다.
3. 빌드 결과로 HTML과 RSC Payload가 만들어진다.
4. 새로고침은 HTTP 요청을 보내며, CDN 또는 서버 캐시 HIT 시 HTML을 받는다.
5. `<Link>` 이동은 브라우저의 Router Cache를 먼저 사용하고, miss 시 RSC Payload를 받아 화면을 합성한다.
6. `revalidateTag()`나 `revalidate: N`은 Data Cache를 stale 처리한다.
7. Full Route Cache가 해당 Data Cache를 참조하는 경우, Data Cache 무효화가 Full Route Cache 무효화로 이어지고 다음 요청에서 re-prerender가 발생할 수 있다.
8. Suspense로 분리된 dynamic 홀처럼 Full Route Cache의 셸이 해당 데이터를 참조하지 않는 경우, re-prerender가 아니라 Data Cache refill과 dynamic render 흐름으로 이해한다.

## 참조한 지식

- `nextjs-cache-prerender.md`: Next.js 캐시 레이어, prerender, revalidate, cache refill의 차이를 설명하기 위해 사용했다.
