# 안전한 에러 처리 패턴의 GraphQL 리졸버

Apollo Server 위에서 동작하는 GraphQL 리졸버를, 의도한 에러는 클라이언트에 그대로 보여주고 예상 못한 에러는 내부 정보 노출 없이 마스킹하도록 정리한 결과물이다. 책임을 네 레이어로 분리한다.

1. **리졸버**: 도메인 로직만. 의도한 에러는 `GraphQLError`에 코드를 붙여 throw, 예상 못한 에러는 그대로 흘려보낸다.
2. **로깅 플러그인**: `didEncounterErrors` 훅에서 요청 컨텍스트까지 포함해 로깅.
3. **formatError**: 외부 노출 정책. safe code 목록에 든 에러만 통과시키고 나머지는 일반 메시지로 마스킹.
4. **응답**: 클라이언트는 가공된 에러만 받는다.

## 리졸버

```js
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
```

`User.posts`는 try/catch가 필요 없어 `async` 없이 Promise만 그대로 return한다.

## 서버 설정

```js
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

## 참조 매핑

| 참조 지식 | 참조 이유 |
|---|---|
| [knowledge/graphql-resolver-return.md](../knowledge/graphql-resolver-return.md) | 리졸버가 Promise를 그대로 return하면 GraphQL이 resolve해주는 규칙. `User.posts`에서 `return await`을 빼고 `async`까지 생략한 근거. |
| [knowledge/async-await-behavior.md](../knowledge/async-await-behavior.md) | 리졸버 내부에서 DB 결과를 실제 객체로 풀어 `!user` 분기를 돌리려면 `await`이 필요해서 `Query.user`는 `async`로 둠. |
| [knowledge/graphql-error-throw.md](../knowledge/graphql-error-throw.md) | "유저 없음"을 클라이언트에 그대로 보여주기 위해 `GraphQLError` + `extensions.code: 'NOT_FOUND'`로 throw. |
| [knowledge/apollo-format-error.md](../knowledge/apollo-format-error.md) | 외부 노출 정책 레이어. safe code 목록에 든 코드만 통과시키고 나머지는 일반 메시지로 마스킹. |
| [knowledge/apollo-plugin-error-logging.md](../knowledge/apollo-plugin-error-logging.md) | `formatError`만으로는 요청 컨텍스트가 부족해, 플러그인 훅에서 operation·userId까지 묶어 로깅. |
