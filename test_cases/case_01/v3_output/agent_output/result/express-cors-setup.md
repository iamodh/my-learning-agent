# 리액트 → Express API 호출의 CORS 설정

리액트 앱이 Express로 만든 API를 호출할 때 발생하는 CORS 에러를, Express 측에서 `cors` 미들웨어로 해결한다.

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

쿠키나 인증 정보를 함께 보낼 경우, 프론트엔드 `fetch` 호출에도 `credentials: 'include'`를 함께 지정한다.

## 참조

- [CORS](../knowledge/cors.md) — 에러의 원인이 브라우저의 출처 검사이며, 서버가 허용 헤더를 응답에 붙여 줘야 한다는 원리를 이해해야 해결 방법이 정해진다. 프리플라이트(OPTIONS)가 왜 먼저 오는지도 이 정책에서 비롯된다.
- [Express `cors` 미들웨어](../knowledge/express-cors-middleware.md) — 허용 헤더를 응답에 붙이는 작업과 프리플라이트 응답 처리를 자동화해 주는 도구라서, 직접 헤더를 다루지 않고 옵션 설정만으로 구현할 수 있다.
