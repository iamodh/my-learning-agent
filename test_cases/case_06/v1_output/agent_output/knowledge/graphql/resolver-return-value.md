# GraphQL 리졸버의 반환값

## 핵심 요약

- 리졸버는 **반드시 값을 `return`** 해야 한다. `return`이 없으면 `undefined`가 반환된다.
- non-nullable 필드(`User!`)에서 `undefined`/`null`이 반환되면 `Cannot return null for non-nullable field` 에러가 발생한다.
- 리졸버가 Promise를 반환하면 GraphQL이 알아서 resolve해 준다. 직접 `await` 하지 않아도 된다.

## 흔한 실수와 수정

```js
// 잘못됨: 본문에서 return 누락 → undefined 반환 → non-null 에러
user: (_, { id }) => {
  db.user.findUnique({ where: { id } });
}

// 올바름: Promise를 그대로 반환하면 GraphQL이 resolve
user: (_, { id }) => {
  return db.user.findUnique({ where: { id } });
}

// 한 줄 화살표 함수면 암묵적 return으로 더 간결하게
user: (_, { id }) => db.user.findUnique({ where: { id } });
```

## 비동기 리졸버 패턴

리졸버 내부에서 비동기 처리(에러 분기 등)가 필요하면 `async`/`await`로 통일하는 것이 안전하다.

```js
user: async (_, { id }) => {
  const user = await db.user.findUnique({ where: { id } });
  return user;
}
```
