# Express `cors` 미들웨어

Express에서 CORS 허용 헤더를 응답에 자동으로 붙여 주는 미들웨어. `app.use(cors())`처럼 인자 없이 쓰면 모든 출처에서 오는 요청을 허용한다.

## 옵션

특정 출처만 허용하거나, 허용할 메서드와 인증 정보 전송 여부를 옵션으로 좁힐 수 있다.

```js
app.use(cors({
  origin: 'https://myapp.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true,
}))
```

- `origin`: 허용할 프론트엔드 주소.
- `methods`: 허용할 HTTP 메서드 목록.
- `credentials: true`: 쿠키나 인증 정보를 함께 주고받을 때 켠다. 이 경우 프론트 쪽 `fetch`에도 `credentials: 'include'`를 같이 줘야 쿠키가 실린다.

## 프리플라이트 자동 처리

미들웨어는 OPTIONS 프리플라이트 요청에 대해서도 `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` 같은 응답을 자동으로 만들어 돌려준다. 따라서 OPTIONS용 라우트를 따로 만들 필요가 없다.
