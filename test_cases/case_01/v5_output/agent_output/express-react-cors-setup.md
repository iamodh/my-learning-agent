# 리액트 → 익스프레스 API CORS 설정

리액트 앱에서 익스프레스로 만든 API를 호출할 때 발생하는 CORS 에러를, 익스프레스 쪽에 `cors` 미들웨어를 적용해 해결했다. 특정 프론트 출처만 허용하고, 쿠키 등 인증 정보를 함께 실어 보낼 수 있도록 설정한다.

## 구현

```js
const express = require('express')
const cors = require('cors')
const app = express()

app.use(cors({
  origin: 'https://myapp.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true,
}))

app.get('/api/users', (req, res) => {
  res.json({ users: [] })
})

app.listen(3000)
```

프론트(`fetch`) 쪽에서는 쿠키를 실어 보내기 위해 `credentials: 'include'`를 함께 지정한다.

## 참조

- [CORS](./cors.md)

## 참조 이유

- **CORS**: 에러가 브라우저의 출처 검사에서 발생하므로, 익스프레스 응답에 허용 헤더를 붙여야 한다는 점, `cors` 미들웨어의 옵션(`origin`, `methods`, `credentials`) 사용법, 그리고 POST 등에서 발생하는 프리플라이트(OPTIONS) 요청까지 `cors` 미들웨어가 함께 처리해 준다는 점이 본 구현 전체의 근거다.
