# GraphQL 리졸버의 반환과 async 동작

GraphQL 리졸버는 스키마가 요구하는 값을 반환해야 한다. 반환값이 없으면 `undefined`가 되고, non-nullable 필드라면 "Cannot return null for non-nullable field" 에러로 떨어진다. 리졸버에서 비동기 호출(예: DB 조회)을 할 때는 Promise를 그대로 반환하거나 `await`으로 값을 꺼낸 뒤 반환해야 한다.

## 리졸버는 반환값을 명시해야 한다

화살표 함수 본문에 `{}` 블록을 쓰면 `return`을 빼먹기 쉽다. 이 경우 `undefined`가 반환된다.

```js
// 잘못된 예 — return이 없어 undefined가 반환됨
user: (_, { id }) => {
  db.user.findUnique({ where: { id } });
}

// 올바른 예 — Promise를 그대로 반환
user: (_, { id }) => {
  return db.user.findUnique({ where: { id } });
}

// 한 줄 축약 (블록 없이) — 암시적 반환
user: (_, { id }) => db.user.findUnique({ where: { id } })
```

GraphQL은 리졸버가 Promise를 반환하면 알아서 resolve해준다. 그래서 `async`/`await`을 쓰지 않아도, Promise를 return만 하면 동작한다.

## async 함수 안에서 await 없이 호출하면

Promise를 반환하는 함수를 `await` 없이 호출하면, 호출 자체는 일어나지만 변수에는 **Pending Promise**가 담긴다. 동기처럼 풀리지 않는다.

```js
async function bad() {
  const user = db.user.findUnique({ where: { id: 1 } });
  console.log(user); // Promise { <pending> }
  return user.name;  // undefined — Promise에 .name은 없다
}

async function good() {
  const user = await db.user.findUnique({ where: { id: 1 } });
  console.log(user); // { id: 1, name: 'Lee' }
  return user.name;  // 'Lee'
}
```

`async` 키워드 자체는 반환값을 Promise로 감싸는 역할만 한다. 함수 내부에서 `await`을 쓰지 않으면 비동기 결과를 그대로 흘려보낼 뿐, 동기로 바꿔주지는 않는다.

## return await은 보통 불필요하다

`async` 함수에서 Promise를 그대로 돌려줄 거라면 `return await`은 마이크로태스크 하나를 더 만들 뿐 동작은 같다.

```js
// 불필요한 await
posts: async (parent) => {
  return await db.post.findMany({ where: { userId: parent.id } });
}

// 동등한 형태 — async도 await도 없이 Promise만 반환
posts: (parent) => {
  return db.post.findMany({ where: { userId: parent.id } });
}
```

`try/catch`로 감쌀 일이 없고 그냥 Promise를 흘려보낼 뿐이라면 `async` 키워드도 뺄 수 있다. 단, 함수 내부에서 값을 꺼내 가공하거나 분기해야 하면 `async`/`await`이 필요하다.
