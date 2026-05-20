# Next.js 캐시와 Prerender

Next.js에서 prerender는 사용자 요청 전에 렌더링 결과를 미리 만들어 두는 것을 뜻한다. Static Generation에서는 빌드 타임에 데이터 패칭, RSC Payload 생성, HTML 생성이 끝나며, 이후 요청은 이미 만들어진 결과를 받는다.

## 캐시 레이어

Request Memoization은 하나의 렌더 패스 안에서 같은 `fetch()` 요청을 중복 실행하지 않게 하는 React 관리 캐시다. 렌더가 끝나면 사라진다.

Data Cache는 `fetch()` 응답 자체를 서버에 저장하는 캐시다. `revalidate: N`이나 `revalidateTag()` 같은 설정과 연결된다.

Full Route Cache는 Data Cache를 사용해 렌더링한 결과물인 HTML과 RSC Payload를 서버에 저장하는 캐시다. Data Cache가 무효화되면, 그 데이터를 참조하는 Full Route Cache도 함께 stale이 될 수 있다.

Router Cache는 브라우저 메모리에 있는 캐시다. `<Link>` 이동 시 RSC Payload를 저장해 두고, 같은 경로로 다시 이동할 때 서버 요청을 생략할 수 있다. 새로고침하면 초기화된다.

## 새로고침과 Link 이동

새로고침은 브라우저의 Router Cache를 버리고 HTTP 요청을 다시 보낸다. 캐시 HIT이면 CDN이나 서버 캐시에서 HTML을 바로 받는다.

`<Link>` 이동은 브라우저의 Router Cache를 먼저 사용한다. 캐시 miss일 때는 RSC Payload를 받아 클라이언트에서 화면을 합성한다.

## Static Generation과 ISR

Static Generation에서는 빌드 때 데이터 패칭, RSC Payload 생성, HTML 생성이 모두 끝난다. 순수 Static Generation이라면 재배포 전까지 prerender가 다시 실행되지 않는다.

ISR에서는 revalidate 조건이 충족되면 stale 상태가 되고, 다음 요청을 계기로 백그라운드 재생성이 일어난다. 이 경우 페이지 전체 HTML을 다시 만드는 흐름은 re-prerender라고 볼 수 있다.

## Data Cache 무효화와 Full Route Cache

`revalidateTag()`나 `revalidate: N`의 직접 대상은 Data Cache다. Full Route Cache는 Data Cache를 재료로 만든 결과물이므로, Full Route Cache에 저장된 셸이나 페이지가 해당 데이터를 참조할 때 Data Cache 무효화가 Full Route Cache 무효화로 이어진다.

반대로 Full Route Cache에 저장된 셸이 그 데이터를 참조하지 않으면 사슬이 연결되지 않는다. 대화에서는 `/`가 Suspense로 셸과 홀을 분리했고, 셸은 시트 데이터를 참조하지 않는 경우가 언급되었다. 이 경우 `revalidateTag("timetable")`이 호출되어도 셸은 그대로이고, 데이터는 홀 안의 dynamic render에서 stale 처리 후 다시 채워진다.

## Cache Refill과 Dynamic Render

Cache refill은 Data Cache를 다시 채우는 과정이다. 즉 `fetch()`가 외부 API를 호출하고 결과를 저장하는 데이터 패칭 단계만 가리킨다.

Dynamic render는 요청마다 실행되는 렌더링이다. cache refill 여부와 별개로 dynamic 홀에서는 RSC Payload와 HTML 생성이 요청마다 일어난다. 차이는 그 결과를 Full Route Cache에 저장하느냐에 있다. re-prerender는 새 HTML을 Full Route Cache에 다시 써넣고, dynamic render는 쓰지 않는다.

## 정정된 오해

처음에는 Data Cache 무효화가 항상 Full Route Cache 무효화와 re-prerender로 이어지는 것처럼 설명되었지만, 이는 Full Route Cache가 해당 데이터를 참조할 때만 성립한다.

또한 프로젝트가 dynamic 페이지라서 Full Route Cache가 없다고 단정한 설명은 탈락했다. 프로젝트 구조를 모르는 상태에서는 `cookies()`, `headers()`, `searchParams`, `noStore()` 같은 동적 API 사용 여부나 라우트 구조를 확인해야 한다.
