# Apollo Server 에러 처리

Apollo Server에서는 리졸버가 throw한 에러가 그대로 클라이언트에 노출되면 스택 트레이스나 내부 정보(SQL 문, DB 구조 등)가 새어나갈 수 있다. 그래서 의도된 에러는 명시적으로 코드를 붙여 throw하고, 예상 못한 에러는 외부에 마스킹한 채 내부 로깅으로 남기는 분리를 둔다. 이를 위해 Apollo는 `GraphQLError`, `formatError`, 플러그인 훅이라는 세 가지 도구를 제공한다.

## GraphQLError로 의도된 에러를 분류

리졸버에서 비즈니스 의미가 있는 에러(유저 없음, 입력 오류 등)는 `GraphQLError` 인스턴스를 만들고 `extensions.code`에 코드를 붙여서 throw한다. 이 코드가 나중에 외부 노출 여부를 결정하는 키가 된다.

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

예상 못한 에러(DB 다운, 네트워크 실패 등)는 별도 처리 없이 그냥 throw한다. 분류와 마스킹은 `formatError`가 한다.

## formatError로 외부 노출 정책 적용

`formatError`는 클라이언트에 나가기 직전 에러를 가공하는 훅이다. 안전한 코드만 통과시키고 나머지는 일반화된 메시지로 마스킹한다.

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (formattedError, error) => {
    const code = formattedError.extensions?.code;
    const safeCodes = ['NOT_FOUND', 'BAD_USER_INPUT', 'UNAUTHENTICATED'];
    if (safeCodes.includes(code)) return formattedError;
    console.error(error);
    return { message: 'Internal server error' };
  },
});
```

리졸버는 정책을 모른 채 도메인 에러만 던지고, `formatError`가 외부 노출 정책을 모아서 결정한다.

## 플러그인 훅으로 컨텍스트 있는 로깅

`formatError`에서 `console.error`만 찍으면 어떤 요청에서 터졌는지 알 수 없다. Apollo의 플러그인 훅 `didEncounterErrors`는 요청 정보와 함께 에러를 받을 수 있어, 운영 환경의 에러 로깅은 보통 여기에서 한다.

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

logger 자체는 winston/pino 같은 라이브러리를 쓰고, 모니터링·알림은 Sentry나 Datadog에 연결하는 게 일반적이다.

## 역할 분리

세 도구가 각자 다른 책임을 진다.

- 리졸버: 도메인 로직만. 의도된 에러는 `GraphQLError`로 코드 붙여 throw, 나머지는 그냥 throw.
- 플러그인 (`didEncounterErrors`): 관찰성. 요청 컨텍스트와 함께 모든 에러를 로깅.
- `formatError`: 외부 노출 정책. safe code만 통과시키고 나머지는 마스킹.
