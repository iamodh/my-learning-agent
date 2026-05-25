# CORS

CORS(Cross-Origin Resource Sharing)는 브라우저가 "다른 출처(origin)에서 받아온 스크립트가 다른 출처의 API를 호출하는 것"을 보안상 제어하는 메커니즘이다. 응답에 `Access-Control-Allow-Origin` 헤더가 붙어 있지 않거나 내 출처와 맞지 않으면, 응답 자체는 도착했더라도 브라우저가 자바스크립트에서 그 응답을 읽지 못하도록 차단한다. 즉 CORS 에러는 서버가 내는 것이 아니라 브라우저가 내는 것이다.

## 익스프레스에서 cors 미들웨어로 처리하기

익스프레스에서는 `cors` 미들웨어를 써서 응답에 허용 헤더를 자동으로 붙인다. `app.use(cors())`만 하면 모든 출처를 허용하고, 운영에서는 보통 특정 출처만 허용하도록 옵션을 좁혀서 쓴다.

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

- `origin`: 허용할 프론트 주소.
- `methods`: 허용할 HTTP 메서드 목록.
- `credentials: true`: 쿠키나 인증 정보를 함께 보내는 요청을 허용한다. 이 경우 프론트에서도 `fetch`에 `credentials: 'include'`를 같이 줘야 쿠키가 실린다.

## 프리플라이트(Preflight)

JSON content-type이나 커스텀 헤더가 들어간 요청처럼 단순하지 않은 요청의 경우, 브라우저는 본 요청을 보내기 전에 OPTIONS 메서드로 "이 요청 보내도 받아줄 거야?"를 서버에 먼저 물어본다. 이를 프리플라이트라고 한다.

미리 물어보는 이유는, 서버가 거절할 요청이라면 실제 DELETE/POST 같은 부작용이 있는 요청이 서버에 도달하기 전에 차단하기 위해서다.

`cors` 미들웨어는 이 OPTIONS 요청에 대해서도 `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` 같은 응답을 자동으로 만들어 돌려주기 때문에, OPTIONS용 라우트를 따로 만들 필요는 없다.
