# async 함수에서 await 생략 시 동작

`async` 함수 안에서 Promise를 반환하는 호출에 `await`을 빼면 동기처럼 풀리지 않는다. 호출 자체는 일어나지만, 반환값이 Promise면 그 Pending Promise가 그대로 변수에 담긴다. 따라서 그 변수에서 객체 속성을 꺼내려 하면 `undefined`가 나온다.

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

`async` 함수는 반환값을 Promise로 감싸기만 할 뿐, 내부 호출을 자동으로 기다려주지는 않는다. 비동기 결과를 실제 값으로 풀려면 호출부에 `await`이 필요하다.
