# React + Express API 연동 (CORS 에러 해결)

## 만들려고 한 것

React 앱에서 Express로 만든 API를 호출하는 구성. 호출 시 발생하는 CORS 에러를 Express 쪽에서 해결하는 것이 목표였다.

## 사용된 지식

- [CORS (Cross-Origin Resource Sharing)](../knowledge/cors.md)

### 왜 이 지식이 필요했는가

- React(프론트)와 Express(API)가 **서로 다른 출처**에서 동작하므로 브라우저의 CORS 정책에 걸린다. 에러 원인이 서버가 아니라 브라우저의 응답 헤더 검사라는 점을 알아야 올바른 위치(서버 응답 헤더)에 해결책을 둘 수 있었다.
- `POST`/`DELETE`나 JSON 요청에서 발생하는 추가 `OPTIONS` 요청(프리플라이트)의 정체와 발동 조건을 알아야, 별도 라우트 없이 미들웨어로 충분하다는 판단을 내릴 수 있었다.
- 쿠키/인증 정보를 함께 보내는 경우 양쪽 설정(`credentials`)이 모두 필요하다는 점을 알아야 인증 흐름이 깨지지 않는다.

## 해결 방법

`cors` 미들웨어를 사용해 응답에 허용 헤더를 자동으로 붙인다.

```bash
npm install cors
```

가장 간단한 형태(모든 출처 허용, 개발용):

```js
app.use(cors())
```

운영 환경 권장 설정(특정 출처만 허용):

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

## 핵심 포인트

- `origin`에 허용할 프론트 주소를 지정한다. 운영에서는 `*` 대신 특정 출처로 좁힌다.
- 쿠키/인증을 함께 보낼 경우 서버에 `credentials: true`, 프론트 `fetch`에 `credentials: 'include'`를 둘 다 설정한다.
- 프리플라이트(`OPTIONS`) 요청도 `cors` 미들웨어가 자동으로 응답을 만들어주므로, 별도의 `OPTIONS` 라우트를 직접 만들 필요가 없다.
