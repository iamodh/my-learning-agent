# Apollo Server 에러 처리

## 핵심 요약

- 의도한 에러는 `GraphQLError`로 `extensions.code`를 붙여 명시적으로 throw 한다.
- 예상 못한 에러는 그대로 throw 하고, 외부 노출은 `formatError`에서 마스킹한다.
- `formatError`는 **safe code 화이트리스트**만 통과시키고 나머지는 일반 메시지로 가린다 (스택 트레이스/내부 SQL 노출 방지).
- 운영 환경 로깅은 `formatError`가 아니라 **플러그인 훅**에서 요청 컨텍스트까지 포함해 남긴다.

## 1. 의도한 에러 throw

```js
import { GraphQLError } from 'graphql';

user: async (_, { id }) => {
  const user = await db.user.findUnique({ where: { id } });
  if (!user) {
    throw new GraphQLError('User not found', {
      extensions: { code: 'NOT_FOUND' },
    });
  }
  return user;
}
```

## 2. formatError로 외부 노출 정책 적용

```js
formatError: (formattedError, error) => {
  const code = formattedError.extensions?.code;
  const safeCodes = ['NOT_FOUND', 'BAD_USER_INPUT', 'UNAUTHENTICATED'];
  if (safeCodes.includes(code)) return formattedError;
  console.error(error);
  return { message: 'Internal server error' };
}
```

## 3. 플러그인으로 컨텍스트 포함 로깅

`formatError`는 요청 정보가 부족하므로, 어떤 요청에서 터졌는지 추적하려면 플러그인 훅을 쓴다.

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
```

logger는 winston/pino, 모니터링/알림은 Sentry나 Datadog에 연결하는 구조가 일반적이다.

## 레이어별 책임 분리

| 레이어 | 책임 |
|--------|------|
| 리졸버 | 도메인 로직, 의도한 에러 throw |
| 플러그인 | 관찰성(요청 컨텍스트 포함 로깅) |
| formatError | 외부 노출 정책(마스킹/통과) |

역할이 섞이지 않게 유지하는 것이 핵심이다.
