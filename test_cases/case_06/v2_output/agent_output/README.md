# 문서 배치

Apollo Server 기반 GraphQL 리졸버의 비동기 처리와 에러 노출을 안전하게 다루기 위해 학습한 내용이다.

```
.
├── README.md
├── result.md                       # 결과: GraphQL 리졸버 안전 패턴
├── graphql-resolver-async.md       # 지식: 리졸버 비동기 처리 (Promise 반환, async/await, return await)
└── apollo-error-handling.md        # 지식: formatError, GraphQLError, 로깅 플러그인
```

`result.md`가 두 지식 문서를 조합해 네 단계 안전 패턴을 정의한다. 지식 문서는 결과 문서에서 `참조 매핑`으로 연결된다.
