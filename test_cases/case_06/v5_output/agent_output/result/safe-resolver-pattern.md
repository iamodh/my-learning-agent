# 안전한 Apollo 리졸버 패턴

## 무엇을 만들었는가

Apollo Server 리졸버에서 비동기 처리·에러 노출·운영 로깅을 책임 분리한 4단계 표준 패턴.

1. 리졸버는 전부 `async`로 통일 (비동기 결과가 Promise로 흘러나가지 않게).
2. 의도한 에러는 `GraphQLError`에 `extensions.code`를 붙여 throw.
3. 예상 못한 에러는 Apollo 플러그인의 `didEncounterErrors`에서 요청 컨텍스트와 함께 로깅.
4. `formatError`는 safe code 목록만 통과시키고 나머지는 일반 메시지로 마스킹.

각 레이어가 도메인 로직 / 관찰성 / 외부 노출 정책으로 분리돼 역할이 섞이지 않는다.

## 구현 코드

```js
import { ApolloServer } from '@apollo/server';
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
    posts: (parent) => db.post.findMany({ where: { userId: parent.id } }),
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

const formatError = (formattedError, error) => {
  const code = formattedError.extensions?.code;
  const safeCodes = ['NOT_FOUND', 'BAD_USER_INPUT', 'UNAUTHENTICATED'];
  if (safeCodes.includes(code)) return formattedError;
  console.error(error);
  return { message: 'Internal server error' };
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [loggingPlugin],
  formatError,
});
```

## 참조 지식

- [Apollo GraphQL 리졸버 패턴](../knowledge/apollo-resolver-patterns.md)

## 참조 이유

- **비동기 리졸버와 async/await**: 리졸버가 Promise를 어떻게 반환·resolve하는지, `async` 함수에서 `await` 누락 시 변수에 pending Promise가 담겨 후속 속성 접근이 undefined가 되는 동작을 알아야 1단계(전부 async 통일)의 근거가 선다. `User.posts`에서 `return await` 대신 `return`만 쓰는 정리도 같은 지식에서 나온다.
- **GraphQLError로 의도한 에러 던지기**: 2단계의 throw 방식과 `extensions.code` 부여 규약. 4단계의 safe code 분기와 짝이 된다.
- **formatError로 외부 노출 마스킹**: 4단계의 직접 구현. safe code 목록에 들지 않는 에러를 일반 메시지로 바꿔 스택 트레이스·내부 SQL 노출을 막는다.
- **플러그인으로 에러 로깅**: 3단계의 직접 구현. `formatError` 자리에서는 요청 컨텍스트가 부족하기 때문에 `requestDidStart` → `didEncounterErrors` 훅에서 operation/userId/stack까지 묶어 남긴다.
