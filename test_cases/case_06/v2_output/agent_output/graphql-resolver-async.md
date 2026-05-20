# GraphQL 리졸버의 비동기 처리

GraphQL 리졸버는 값을 **반환**해야 한다. 함수 본문에 `return`이 없으면 `undefined`가 반환되고, non-nullable 필드(`User!`)의 경우 `Cannot return null for non-nullable field` 에러로 떨어진다. 리졸버가 Promise를 반환하면 GraphQL이 알아서 resolve하므로, DB 호출 같은 비동기 작업의 결과를 그대로 return하면 된다.

```js
const resolvers = {
  Query: {
    user: (_, { id }) => {
      return db.user.findUnique({ where: { id } });
    },
  },
};
```

화살표 함수 한 줄로 줄여 `user: (_, { id }) => db.user.findUnique({ where: { id } })` 형태로도 쓴다.

## async 함수에서 await을 빠뜨리면

`async` 함수 내부에서 Promise를 반환하는 호출에 `await`을 붙이지 않으면, 변수에는 실제 값이 아닌 Pending Promise 자체가 담긴다. 동기처럼 풀리지 않으므로 그 변수의 속성에 접근하면 `undefined`가 된다.

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

`async` 함수는 반환값을 Promise로 감쌀 뿐이고, 내부에서 `await`을 안 쓰면 비동기 결과를 그대로 흘려보낸다. 리졸버에서도 비동기 처리가 들어가면 `async` 붙이고 `await`을 쓰는 패턴으로 통일하면 안전하다.

## `return await`은 불필요

`async` 함수는 어차피 반환값을 Promise로 감싸기 때문에, Promise를 그대로 흘려보낼 거면 `return await` 대신 `return`만 써도 동일하게 동작하고 마이크로태스크가 하나 덜 생긴다. `try/catch`로 감쌀 일이 없으면 `async` 키워드 자체도 빼고 Promise를 그대로 반환해도 GraphQL이 처리한다.

```js
User: {
  posts: (parent) => {
    return db.post.findMany({ where: { userId: parent.id } });
  },
},
```
