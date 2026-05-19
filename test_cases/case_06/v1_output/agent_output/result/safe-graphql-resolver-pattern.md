# 결과: 안전한 GraphQL 리졸버 + 에러 처리 표준 패턴

## 만들려고 했던 것

Apollo Server에서 동작이 깨지지 않고, 내부 정보를 노출하지 않으며, 운영 환경에서 추적 가능한 GraphQL 리졸버/에러 처리 표준 패턴. 처음엔 단순한 `user` 리졸버의 non-null 에러에서 출발해, 단계적으로 안전한 4단계 표준 패턴까지 도달했다.

## 최종 표준 패턴 (4단계)

1. **리졸버는 전부 `async`로 통일** — 비동기 처리가 섞여도 일관되고 안전.
2. **의도한 에러는 `GraphQLError`로 코드 붙여 throw** — 예: 유저 없음 → `NOT_FOUND`.
3. **예상 못한 에러는 플러그인에서 요청 컨텍스트 포함해 로깅** — 어떤 요청/유저에서 터졌는지 추적.
4. **`formatError`는 safe code만 통과, 나머지는 마스킹** — 스택/SQL 노출 방지.

각 레이어의 책임이 분리되어(리졸버=도메인 로직, 플러그인=관찰성, formatError=노출 정책) 유지보수가 쉽다.

## 최종 코드

```js
import { GraphQLError } from 'graphql';

const resolvers = {
  Query: {
    user: async (_, { id }) => {
      const user = await db.user.findUnique({ where: { id } });
      if (!user) {
        throw new GraphQLError('User not found', {
          extensions: { code: 'NOT_FOUND' },
        });
      }
      return user;
    },
  },
  User: {
    // return await / async 불필요: Promise 그대로 반환
    posts: (parent) => db.post.findMany({ where: { userId: parent.id } }),
  },
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [loggingPlugin],
  formatError,
});
```

## 사용된 지식과 이유

| 지식 문서 | 왜 사용되었는가 |
|-----------|------------------|
| [GraphQL 리졸버의 반환값](../knowledge/graphql/resolver-return-value.md) | 최초 문제(`Cannot return null for non-nullable field`)의 원인이 리졸버에서 `return` 누락이었기 때문. 패턴의 출발점. |
| [async/await와 Promise 반환 동작](../knowledge/javascript/async-await-promise.md) | `user.name`이 `undefined`로 나온 원인이 `await` 누락이었고, `User.posts`의 `return await`/`async` 생략 판단 근거가 되었기 때문. |
| [Apollo Server 에러 처리](../knowledge/graphql/error-handling.md) | DB 실패 시 스택/SQL 노출을 막고, 의도한 에러는 그대로 내려주며, 운영 추적을 가능하게 하는 4단계 패턴의 핵심 근거이기 때문. |

## 해결 흐름 요약

1. `return` 누락 → non-null 에러 → 리졸버에서 Promise return.
2. `await` 누락 → `user.name` undefined → `async/await` 통일.
3. DB 실패 시 내부 노출 우려 → `formatError`로 마스킹.
4. 특정 에러는 그대로 노출 필요 → `GraphQLError` + `extensions.code` 분기.
5. 운영 추적 필요 → 플러그인 훅에서 컨텍스트 포함 로깅.
6. → 최종 4단계 표준 패턴으로 정리.
