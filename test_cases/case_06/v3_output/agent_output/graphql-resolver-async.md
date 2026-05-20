# GraphQL 리졸버와 async 처리

GraphQL 리졸버는 Promise를 반환할 수 있고, GraphQL은 반환된 Promise를 resolve해서 응답에 사용한다. 리졸버 함수 본문에서 Promise를 만들고도 `return`하지 않으면 `undefined`가 반환되며, 스키마가 non-null 필드라면 null 관련 에러가 발생할 수 있다.

## 리졸버에서 Promise 반환하기

`user(id: ID!): User!`처럼 non-null 필드인데 리졸버가 값을 반환하지 않으면 `Cannot return null for non-nullable field Query.user` 에러가 날 수 있다.

```js
const resolvers = {
  Query: {
    user: (_, { id }) => {
      return db.user.findUnique({ where: { id } });
    },
  },
};
```

한 줄 화살표 함수로 Promise를 바로 반환할 수도 있다.

```js
user: (_, { id }) => db.user.findUnique({ where: { id } })
```

## async 함수 안의 await

`async` 함수 안에서도 `await`을 쓰지 않으면 Promise가 자동으로 풀리지 않는다. Promise 자체가 변수에 담기므로 실제 객체의 필드를 읽을 수 없다.

```js
async function resolveUserName(id) {
  const user = await db.user.findUnique({ where: { id } });
  return user.name;
}
```

대화에서는 `await` 없이 `user.name`을 읽으면 `user`가 실제 객체가 아니라 Pending Promise라서 `undefined`가 나온다고 정리했다.

## return await 생략

try/catch로 감싸는 상황이 아니라면 `return await`는 불필요하다. async 함수가 반환값을 Promise로 감싸기 때문에 Promise를 그대로 반환해도 GraphQL이 처리할 수 있다.

```js
User: {
  posts: (parent) => {
    return db.post.findMany({ where: { userId: parent.id } });
  },
},
```

대화에서는 `User.posts`에서 `return await db.post.findMany(...)` 대신 Promise를 그대로 반환하는 형태로 정리했다.
