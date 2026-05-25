# Apollo Server formatError

`formatError`는 Apollo Server가 클라이언트에 내보내는 에러 응답을 가공하는 옵션이다. 리졸버에서 발생한 에러를 그대로 throw하면 스택 트레이스나 내부 SQL이 클라이언트에 노출될 수 있는데, `formatError`에서 외부로 나가는 메시지를 일반화하면 그걸 막을 수 있다.

내부 에러는 로깅하고, 외부에는 일반화된 메시지만 내보내는 식으로 쓴다.

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

리졸버에서는 예상 못한 에러는 그대로 throw하고, 마스킹은 `formatError`에서 일괄 처리하는 게 깔끔하다.

## safe code만 통과시키기

클라이언트에 그대로 노출해도 되는 에러("유저 없음" 같은)는 미리 정한 코드 목록에 들어 있을 때만 통과시키고, 나머지는 마스킹한다. 코드 분기에는 `formattedError.extensions?.code`를 본다.

```js
formatError: (formattedError, error) => {
  const code = formattedError.extensions?.code;
  const safeCodes = ['NOT_FOUND', 'BAD_USER_INPUT', 'UNAUTHENTICATED'];
  if (safeCodes.includes(code)) return formattedError;
  console.error(error);
  return { message: 'Internal server error' };
},
```
