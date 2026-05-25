# CORS와 익스프레스 cors 미들웨어

CORS(Cross-Origin Resource Sharing)는 브라우저가 "다른 출처(origin)에서 받아온 스크립트가 또 다른 출처의 API를 호출하는 것"을 보안상 위험하게 보고 제한하는 정책이다. 응답에 `Access-Control-Allow-Origin` 같은 허용 헤더가 붙어 있는지 브라우저가 검사하고, 헤더가 없거나 내 출처와 맞지 않으면 응답이 실제로 도착했더라도 자바스크립트가 그 응답을 읽지 못하도록 막는다.

## 에러를 내는 주체는 서버가 아니라 브라우저

리액트 앱이 익스프레스 API를 호출했을 때 뜨는 CORS 에러는 서버가 거절해서 나는 것이 아니다. 서버는 응답을 정상적으로 보낸다. 다만 그 응답에 허용 헤더가 없으면 브라우저가 응답 내용을 스크립트에 넘기지 않고 차단한다. 그래서 해결은 응답에 적절한 허용 헤더를 붙이는 일, 즉 서버 쪽에서 처리하는 일이 된다.

## 익스프레스에서 cors 미들웨어로 처리

익스프레스에서는 `cors` 미들웨어가 응답에 필요한 허용 헤더들을 자동으로 붙여준다. 가장 간단한 형태는 `app.use(cors())`로, 모든 출처의 요청을 일단 허용한다. 운영에서는 보통 허용할 출처를 명시적으로 좁혀 쓴다.

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
- `credentials: true`: 쿠키나 인증 정보를 같이 실어 보낼 때 켠다. 이 경우 프론트의 `fetch`에도 `credentials: 'include'`를 같이 줘야 쿠키가 실제로 실린다.

## 프리플라이트 (OPTIONS 요청)

POST나 DELETE 같이 부작용이 있는 요청, JSON content-type 또는 커스텀 헤더가 들어간 "단순하지 않은" 요청에서는 브라우저가 본 요청을 보내기 전에 OPTIONS 메서드로 "이 요청 보내도 받아줄 거야?"를 먼저 물어본다. 이 사전 질의가 프리플라이트다.

먼저 물어보는 이유는, 서버가 거절할 요청이라면 실제 POST/DELETE가 서버에 도달해 부작용을 일으키기 전에 미리 차단하기 위해서다.

`cors` 미들웨어는 이 OPTIONS 요청에 대해서도 `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` 같은 응답 헤더를 자동으로 만들어 돌려준다. 그래서 프리플라이트를 위해 별도 라우트를 만들 필요는 없다.
