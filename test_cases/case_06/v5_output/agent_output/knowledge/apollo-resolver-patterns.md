# Apollo GraphQL 리졸버 패턴

Apollo Server의 리졸버는 GraphQL 스키마의 각 필드에 대응하는 함수로, 반환값(혹은 Promise의 resolve 값)이 응답에 채워진다. non-nullable 필드(`User!` 같은)에서 함수가 명시적 `return` 없이 끝나면 undefined가 반환되어 `Cannot return null for non-nullable field` 에러가 발생한다. 리졸버에서의 비동기 처리, 의도한 에러 던지기, 외부 노출 가공, 운영 로깅까지가 한 리졸버 호출·한 Apollo Server 설정 안에서 함께 다뤄지는 측면이라 같이 정리한다.

## 비동기 리졸버와 async/await

리졸버가 Promise를 반환하면 GraphQL이 resolve를 기다린 뒤 값을 응답에 채운다. 따라서 DB 호출 같은 비동기 작업은 그 Promise를 그대로 `return`만 해줘도 동작한다.

```js
const resolvers = {
  Query: {
    user: (_, { id }) => db.user.findUnique({ where: { id } }),
  },
};
```

리졸버 안에서 결과를 가공해야 하면 `async`/`await`로 통일하는 게 안전하다. `async` 함수 안에서 `await` 없이 Promise 반환 함수를 호출하면, 변수에는 실제 값이 아니라 pending Promise가 담긴다. 동기처럼 풀리지 않으므로 `.name` 같은 속성 접근은 undefined가 된다.

```js
async function bad() {
  const user = db.user.findUnique({ where: { id: 1 } });
  console.log(user); // Promise { <pending> }
  return user.name;  // undefined
}

async function good() {
  const user = await db.user.findUnique({ where: { id: 1 } });
  console.log(user); // { id: 1, name: 'Lee' }
  return user.name;  // 'Lee'
}
```

`async` 함수는 반환값을 항상 Promise로 감싸기 때문에, 내부에 `try/catch` 등 추가 처리가 없고 그냥 Promise를 그대로 흘려보내기만 한다면 `return await` 대신 `return`만 써도 동일하게 동작하며 마이크로태스크가 하나 덜 생긴다. 그런 경우 `async` 키워드 자체도 생략 가능하다.

```js
User: {
  posts: (parent) => db.post.findMany({ where: { userId: parent.id } }),
},
```

## GraphQLError로 의도한 에러 던지기

리졸버 안에서 클라이언트에 그대로 보여주고 싶은 에러(예: "유저 없음")는 `GraphQLError` 인스턴스로 throw하고 `extensions.code`에 식별자를 붙인다. 이 코드를 뒤에서 분기에 쓴다.

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

예상 못한 에러는 리졸버에서 그대로 throw하고, 외부 노출 가공은 아래 formatError에서 처리한다.

## formatError로 외부 노출 마스킹

`ApolloServer`의 `formatError` 옵션은 클라이언트로 나가는 에러를 가공하는 훅이다. 내부 에러는 로깅하고, 외부엔 일반화된 메시지만 내보낸다. `extensions.code`를 보고 통과시킬 코드 목록(safe codes)을 정해 그것만 원본대로 내려주고, 나머지는 마스킹한다.

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

## 플러그인으로 에러 로깅

`formatError` 자리에서 로깅하면 어느 요청에서 터졌는지 같은 컨텍스트가 부족하다. Apollo Server는 플러그인의 `requestDidStart` → `didEncounterErrors` 훅으로 요청 정보까지 같이 남길 수 있다.

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

logger 자체는 winston/pino 같은 라이브러리를 쓰고, 실제 모니터링은 Sentry/Datadog 같은 외부 서비스에 연결한다는 일반적인 구성이 언급됐다(구체 설정은 대화에서 다루지 않았다).
