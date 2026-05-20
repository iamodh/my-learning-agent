# Apollo Server 에러 처리

리졸버에서 발생한 에러를 그대로 클라이언트에 흘리면 스택 트레이스·내부 SQL 같은 내부 정보가 노출된다. Apollo Server는 **`formatError`로 외부 노출을 가공**하고, **`GraphQLError`로 의도한 에러를 코드와 함께 던지고**, **플러그인으로 요청 컨텍스트까지 포함해 로깅**하는 식으로 책임을 분리한다.

## formatError로 외부 노출 마스킹

`formatError`는 클라이언트에 나가는 에러를 가공한다. 내부 에러는 로깅하고, 외부엔 일반화된 메시지만 내보낸다.

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (formattedError, error) => {
    console.error(error); // 내부 로깅
    if (formattedError.extensions?.code === 'INTERNAL_SERVER_ERROR') {
      return { message: 'Internal server error' };
    }
    return formattedError;
  },
});
```

리졸버에서는 예상 못한 에러는 그대로 throw하고, `formatError`에서 마스킹하는 게 깔끔하다.

## GraphQLError로 의도한 에러를 통과시키기

특정 에러(예: "유저 없음")는 클라이언트에 그대로 보여주고 싶다면, `GraphQLError` 인스턴스를 `extensions.code`와 함께 throw한다. `formatError`에서 그 코드를 보고 통과시킬지 마스킹할지 결정한다.

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

```js
formatError: (formattedError, error) => {
  const code = formattedError.extensions?.code;
  const safeCodes = ['NOT_FOUND', 'BAD_USER_INPUT', 'UNAUTHENTICATED'];
  if (safeCodes.includes(code)) return formattedError;
  console.error(error);
  return { message: 'Internal server error' };
},
```

## 플러그인으로 컨텍스트 포함 로깅

`formatError`에서 로깅하면 어떤 요청에서 터졌는지 같은 컨텍스트가 부족하다. Apollo Server 플러그인의 `requestDidStart` → `didEncounterErrors` 훅에서 요청 정보까지 같이 남긴다.

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

`logger`는 winston/pino 같은 라이브러리를 쓰고, 실제 모니터링은 Sentry나 Datadog에 연결해서 알림까지 받는 구조가 일반적이다.
