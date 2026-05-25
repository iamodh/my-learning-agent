# 안전한 GraphQL 리졸버 패턴

Apollo Server 기반에서 리졸버를 짤 때, non-nullable 에러·내부 정보 노출·로깅 컨텍스트 부족 같은 문제를 한 번에 막는 4단계 표준 패턴을 만들었다. 리졸버는 도메인 로직만, 플러그인은 관찰성, `formatError`는 외부 노출 정책 — 이렇게 레이어별 책임을 분리하는 것이 핵심이다.

## 패턴 4단계

1. **리졸버는 전부 `async`로 통일**하고, 비동기 호출은 항상 `await`으로 값을 꺼낸 뒤 분기·반환한다.
2. **의도한 에러는 `GraphQLError`로 코드를 붙여 throw**한다 (`NOT_FOUND`, `BAD_USER_INPUT`, `UNAUTHENTICATED` 등).
3. **예상 못한 에러는 그대로 throw**하고, 플러그인의 `didEncounterErrors`에서 요청 컨텍스트와 함께 로깅한다.
4. **`formatError`에서 safe code만 통과시키고** 나머지는 일반화된 메시지로 마스킹한다.

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
    posts: (parent) => {
      return db.post.findMany({ where: { userId: parent.id } });
    },
  },
};

const loggingPlugin = {
  async requestDidStart() {
    return {
      async didEncounterErrors({ request, errors, contextValue }) {
        for (const err of errors) {
          logger.error({
            message: err.message,
            path: err.path,
            code: err.extensions?.code,
            operation: request.operationName,
            userId: contextValue.userId,
            stack: err.originalError?.stack,
          });
        }
      },
    };
  },
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [loggingPlugin],
  formatError: (formattedError, error) => {
    const code = formattedError.extensions?.code;
    const safeCodes = ['NOT_FOUND', 'BAD_USER_INPUT', 'UNAUTHENTICATED'];
    if (safeCodes.includes(code)) return formattedError;
    console.error(error);
    return { message: 'Internal server error' };
  },
});
```

`User.posts`는 Promise를 그대로 반환하면 충분해서 `async`/`return await`을 모두 뺐다 — 내부에서 값을 꺼내 가공할 필요가 없다.

## 참조 매핑

- [graphql-resolver-async.md](./graphql-resolver-async.md) — 리졸버가 Promise를 반환·await해야 하는 이유, `User.posts`에서 `async`를 뺀 근거.
- [apollo-error-handling.md](./apollo-error-handling.md) — `GraphQLError`로 코드를 붙여 분류하고, 플러그인에서 로깅하고, `formatError`로 마스킹하는 세 레이어의 역할 분리.
