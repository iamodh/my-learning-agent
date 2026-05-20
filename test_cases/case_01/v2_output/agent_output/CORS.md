# CORS (Cross-Origin Resource Sharing)

CORS 에러는 사실 서버가 내는 게 아니라 **브라우저가** 내는 차단이다. 브라우저는 보안상 "다른 출처(origin)에서 받아온 스크립트가 마음대로 다른 출처의 API를 호출하면 위험하다"고 보고, 응답에 `Access-Control-Allow-Origin` 헤더가 붙어 있는지 검사한다. 그 헤더가 없거나 내 출처와 맞지 않으면 응답 자체는 도착했더라도 자바스크립트가 읽지 못하게 막는다. 따라서 서버는 적절한 허용 헤더를 응답에 붙여주기만 하면 된다.

## Express에서 `cors` 미들웨어로 처리하기

가장 간단한 방법은 `cors` 미들웨어를 쓰는 것이다.

```sh
npm install cors
```

모든 출처를 일단 허용하려면 한 줄이면 된다.

```js
app.use(cors())
```

운영 환경에서는 보통 특정 출처만 허용하도록 좁혀서 쓴다.

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

- `origin`: 허용할 프론트 주소
- `methods`: 허용할 HTTP 메서드 목록
- `credentials: true`: 쿠키나 인증 정보를 같이 보낼 때 켠다. 프론트 쪽에서도 `fetch`에 `credentials: 'include'`를 같이 줘야 실제로 쿠키가 실린다.

## 프리플라이트 (Preflight, OPTIONS 요청)

POST 같은 요청을 보낼 때 브라우저가 본 요청 전에 OPTIONS 메서드로 한 번 더 요청을 먼저 보내는 경우가 있는데, 이를 프리플라이트라고 한다. JSON content-type이나 커스텀 헤더가 들어간 "단순하지 않은 요청"에서 자동으로 발동한다.

미리 물어보는 이유는, 서버가 거절할 요청이라면 실제 DELETE나 POST가 서버에 도달해 부작용을 일으키기 전에 차단하기 위해서다. 즉 "이 요청 보내도 서버가 받아줄 거야?"를 본 요청 전에 미리 확인하는 절차다.

`cors` 미들웨어는 이 OPTIONS 요청에 대해서도 `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` 같은 응답을 자동으로 만들어 돌려주므로, 별도의 OPTIONS 라우트를 직접 만들 필요는 없다.
