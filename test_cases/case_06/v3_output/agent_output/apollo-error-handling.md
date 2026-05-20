# Apollo Server 에러 처리

Apollo Server에서는 클라이언트에 내보낼 에러와 내부에서만 기록할 에러를 분리할 수 있다. 예상 못한 내부 에러는 로깅하고 클라이언트에는 일반화된 메시지만 반환하며, 의도한 에러는 `GraphQLError`와 `extensions.code`로 구분한다.

## formatError로 외부 노출 제어

`formatError`는 클라이언트로 나가는 에러를 가공하는 역할을 한다. 내부 에러는 기록하고, 외부에는 일반화된 메시지를 보낼 수 있다.

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (formattedError, error) => {
    console.error(error);
    if (formattedError.extensions?.code === 'INTERNAL_SERVER_ERROR') {
      return { message: 'Internal server error' };
    }
    return formattedError;
  },
});
```

## 의도한 에러 구분

클라이언트에 그대로 보여주고 싶은 에러는 `GraphQLError` 인스턴스로 던지고 `extensions.code`를 붙인다.

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
};
```

`formatError`에서는 안전하게 통과시킬 code만 허용하고, 나머지는 마스킹한다.

```js
formatError: (formattedError, error) => {
  const code = formattedError.extensions?.code;
  const safeCodes = ['NOT_FOUND', 'BAD_USER_INPUT', 'UNAUTHENTICATED'];
  if (safeCodes.includes(code)) return formattedError;
  console.error(error);
  return { message: 'Internal server error' };
},
```

## 요청 컨텍스트를 포함한 로깅

운영 환경에서는 `formatError`만으로는 요청 정보가 부족할 수 있다. 대화에서는 Apollo Server 플러그인 훅에서 요청 정보, 에러 path, code, operation, userId, stack을 함께 로깅하는 방식이 제시됐다.

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
  formatError,
});
```
