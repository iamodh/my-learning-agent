# 안전한 GraphQL 리졸버 패턴

만들려는 것은 DB 호출을 포함한 GraphQL 리졸버에서 비동기 반환, 의도한 에러, 예상 못한 에러 로깅, 클라이언트 노출 정책을 분리한 안전한 구현 패턴이다.

## 최종 패턴

- 리졸버는 비동기 DB 호출 결과를 `return`하거나 필요한 곳에서 `await`한다.
- 의도한 에러는 `GraphQLError`로 던지고 `extensions.code`를 붙인다.
- 예상 못한 에러는 Apollo 플러그인에서 요청 컨텍스트와 함께 로깅한다.
- `formatError`는 safe code만 통과시키고 나머지는 일반 메시지로 마스킹한다.

## 참조 지식

- `graphql-resolver-async.md`: 리졸버가 Promise를 반환해야 GraphQL이 값을 resolve할 수 있고, `await` 없이 Promise의 필드를 읽으면 실제 데이터가 나오지 않기 때문에 필요하다.
- `apollo-error-handling.md`: 의도한 에러와 내부 에러를 구분하고, 운영 환경에서 로깅과 클라이언트 노출 정책을 분리하기 위해 필요하다.
