# Express에서 CORS 처리

CORS 에러는 서버가 직접 에러를 내는 것이 아니라, 브라우저가 다른 출처의 API 응답을 자바스크립트에서 읽어도 되는지 검사하면서 발생한다. 응답에 `Access-Control-Allow-Origin` 헤더가 없거나 현재 프론트 출처와 맞지 않으면, 응답이 도착했더라도 브라우저가 접근을 막는다.

## cors 미들웨어

Express에서는 `cors` 미들웨어를 사용해 CORS 허용 헤더를 응답에 자동으로 붙일 수 있다.

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

`origin`에는 허용할 프론트 주소를 적는다. 쿠키나 인증 정보를 함께 보내려면 Express 쪽에서 `credentials: true`를 켜고, 프론트의 `fetch`에서도 `credentials: 'include'`를 함께 사용해야 한다.

## 프리플라이트

프리플라이트는 브라우저가 본 요청 전에 `OPTIONS` 메서드로 서버에 미리 허용 여부를 확인하는 절차다. JSON `Content-Type`이나 커스텀 헤더가 들어간 요청처럼 단순하지 않은 요청에서 자동으로 발생한다.

이 절차는 서버가 거절할 요청이라면 실제 `POST`나 `DELETE` 요청이 서버에 도달해 부작용을 일으키기 전에 차단하기 위한 것이다. `cors` 미들웨어는 이 `OPTIONS` 요청에 대해서도 `Access-Control-Allow-Methods`, `Allow-Headers` 같은 응답을 자동으로 만들어 주므로, 대화에서 다룬 범위에서는 별도 라우트를 만들 필요가 없다.
