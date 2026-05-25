# GraphQL 리졸버 반환 규칙

GraphQL 리졸버는 함수의 반환값을 그대로 응답에 사용한다. 본문에서 `return`을 빠뜨리면 `undefined`가 반환되고, 스키마가 non-nullable 필드(`User!`)라면 `"Cannot return null for non-nullable field"` 에러로 떨어진다.

리졸버가 Promise를 반환하면 GraphQL이 알아서 resolve해준다. 따라서 DB 호출 같은 비동기 작업은 Promise를 그대로 `return`하기만 하면 된다.

```js
const resolvers = {
  Query: {
    user: (_, { id }) => {
      return db.user.findUnique({ where: { id } });
    },
  },
};
```

화살표 함수의 표현식 형태로 줄여 쓸 수도 있다.

```js
user: (_, { id }) => db.user.findUnique({ where: { id } })
```

## `return await` 불필요

리졸버가 `async`라면 함수가 자동으로 반환값을 Promise로 감싼다. 그래서 `return await promise`는 `return promise`와 결과가 같고, 오히려 마이크로태스크가 하나 더 생긴다. try/catch로 감쌀 일이 없다면 `async` 키워드 자체를 빼고 Promise를 그대로 반환해도 된다.

```js
User: {
  posts: (parent) => {
    return db.post.findMany({ where: { userId: parent.id } });
  },
},
```
