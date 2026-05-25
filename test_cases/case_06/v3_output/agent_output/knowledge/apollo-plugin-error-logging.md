# Apollo Server 플러그인으로 에러 로깅

`formatError`는 응답 가공 시점이라 요청 컨텍스트가 부족하다. 어떤 요청·어떤 유저에서 터졌는지까지 추적하려면 Apollo Server 플러그인의 `requestDidStart` → `didEncounterErrors` 훅에서 로깅한다. 이 훅에서는 `request`, `errors`, `contextValue`가 함께 들어오므로 요청 정보와 에러를 한 레코드로 묶을 수 있다.

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

logger 자리에는 winston/pino 같은 라이브러리를 쓰고, 실제 모니터링은 Sentry/Datadog 같은 외부 시스템에 연결해 알림까지 받는 구조가 일반적이다.
