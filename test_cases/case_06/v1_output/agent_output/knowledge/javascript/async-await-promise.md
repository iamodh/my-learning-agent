# async/await와 Promise 반환 동작

## 핵심 요약

- `async` 함수는 반환값을 **항상 Promise로 감싼다**. 그게 전부다.
- `await`을 쓰지 않으면 Promise가 풀리지 않고 **Promise 객체 그대로** 변수에 담긴다.
- Promise를 반환하는 함수(예: DB 호출)를 호출만 하고 `await`하지 않으면 결과 객체가 아니라 `Promise { <pending> }`를 다루게 된다.
- `return await x`는 보통 불필요하다. `async` 함수가 어차피 Promise로 감싸므로 `return x`와 결과가 같고, 마이크로태스크가 하나 덜 생긴다.

## 왜 그런가

`await` 없는 호출도 함수 실행 자체는 일어난다. 다만 반환값이 Promise면 동기처럼 풀리지 않고 그 Promise가 그대로 변수에 들어간다. 그래서 `.name` 같은 속성에 접근하면 `undefined`가 나온다.

## 예시

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

## return await 생략

```js
// 불필요한 return await
posts: async (parent) => {
  return await db.post.findMany({ where: { userId: parent.id } });
}

// 동등하며 더 가볍다 (try/catch가 없으면 async도 생략 가능)
posts: (parent) => {
  return db.post.findMany({ where: { userId: parent.id } });
}
```
