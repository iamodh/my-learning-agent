# GraphQL 리졸버 안전 패턴

Apollo Server 기반 GraphQL 서버에서 리졸버 비동기 처리와 에러 노출을 안전하게 다루기 위한 패턴이다. 네 단계로 책임이 분리되어 있다.

1. **리졸버는 전부 `async`로 통일** — Promise 반환과 `await` 사용을 일관시켜 non-nullable 필드 에러와 `undefined` 누락을 막는다.
2. **의도한 에러는 `GraphQLError`로 `extensions.code`를 붙여 throw** — "유저 없음" 같이 클라이언트에 그대로 노출해야 하는 에러를 명시적으로 식별한다.
3. **예상 못한 에러는 플러그인에서 컨텍스트 포함해 로깅** — `didEncounterErrors` 훅으로 요청·유저·스택까지 남긴다.
4. **`formatError`는 safe code만 통과시키고 나머지는 마스킹** — 내부 정보가 응답에 새지 않도록 외부 노출 정책을 한 곳에 모은다.

각 레이어가 서로 섞이지 않게 — 리졸버는 도메인 로직만, 플러그인은 관찰성, `formatError`는 외부 노출 정책 — 분리해 유지한다.

## 참조 매핑

| 단계 | 참조 지식 | 참조 이유 |
|------|----------|----------|
| 1. 리졸버 async 통일 | [GraphQL 리졸버의 비동기 처리](./graphql-resolver-async.md) | `return` 누락 시 non-nullable 에러, `await` 누락 시 Promise가 그대로 변수에 담겨 `undefined`가 되는 동작이 이 패턴이 `async`로 통일해야 하는 직접적 근거다. |
| 2. GraphQLError로 의도 에러 throw | [Apollo Server 에러 처리](./apollo-error-handling.md) | `GraphQLError`에 `extensions.code`를 붙이는 방식이 의도한 에러를 식별 가능하게 만드는 메커니즘이다. |
| 3. 플러그인 로깅 | [Apollo Server 에러 처리](./apollo-error-handling.md) | `requestDidStart` → `didEncounterErrors` 훅이 요청 컨텍스트까지 포함한 로깅의 유일한 진입점으로 채택됐다. |
| 4. formatError 마스킹 | [Apollo Server 에러 처리](./apollo-error-handling.md) | `safeCodes` 화이트리스트로 통과·마스킹을 분기하는 방식이 외부 노출 정책의 구현 근거다. |
