# CORS (Cross-Origin Resource Sharing)

## 핵심 요약

CORS 에러를 내는 건 **서버가 아니라 브라우저**다. 브라우저는 다른
출처(origin)에서 받은 스크립트가 다른 출처의 API를 호출하면, 응답에
`Access-Control-Allow-Origin` 헤더가 붙어 있는지 검사한다. 그 헤더가
없거나 내 출처와 안 맞으면 **응답은 도착했어도 JavaScript가 읽지
못하게** 막는다. 따라서 해결은 "서버가 허용 헤더를 응답에 붙이도록"
하는 것이다.

## 왜 막는가

브라우저 보안 정책상, A 사이트에서 로드된 스크립트가 사용자의
인증 상태로 B 사이트 API를 마음대로 호출하면 위험하다. 그래서
교차 출처 요청의 응답을 기본적으로 차단하고, 서버가 명시적으로
허용한 출처에만 열어준다.

## 프리플라이트 (preflight)

JSON content-type이나 커스텀 헤더가 들어간 "단순하지 않은" 요청은,
본 요청 전에 브라우저가 `OPTIONS` 메서드로 "이 요청 보내도 돼?"를
먼저 물어본다. 서버가 거절할 요청이라면 실제 `POST`/`DELETE`가
서버에 도달해 부작용을 일으키기 전에 차단하기 위해서다. 서버는
이 OPTIONS에 `Access-Control-Allow-Methods`,
`Access-Control-Allow-Headers` 응답을 돌려줘야 한다.

## Express에서의 적용

`cors` 미들웨어가 위 허용 헤더들(프리플라이트 응답 포함)을 자동으로
붙여준다. 전체 허용은 `app.use(cors())`, 운영 환경에서는 출처를
좁혀서 쓴다.

```js
const express = require('express')
const cors = require('cors')
const app = express()

app.use(cors({
  origin: 'https://myapp.com',   // 허용할 프론트 출처
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true,             // 쿠키/인증정보 동반 허용
}))

app.get('/api/users', (req, res) => {
  res.json({ users: [] })
})

app.listen(3000)
```

쿠키·인증 정보를 같이 보내려면 서버 `credentials: true`와 더불어
프론트 `fetch`에 `credentials: 'include'`를 함께 줘야 한다.
