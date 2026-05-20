# JWT 인증

JWT(JSON Web Token)는 서버가 서명한 토큰을 클라이언트가 들고 다니며 매 요청마다 첨부하는 stateless 인증 방식이다. 서버는 토큰의 서명만 검증하면 되므로 별도의 세션 저장소가 필요 없다. SPA·모바일 앱에서 흔히 쓰이며 확장에 유리하다.

토큰은 `header.payload.signature` 세 부분으로 구성된 문자열이다.

- **header**: 알고리즘 정보
- **payload**: 사용자 ID 같은 식별 정보
- **signature**: 서버만 아는 비밀키로 `header+payload`를 서명한 값

서버는 받은 토큰을 같은 비밀키로 다시 서명해 일치 여부만 확인한다. 일치하면 payload를 신뢰할 수 있다 — 중간에 payload가 변조됐다면 서명이 맞지 않기 때문이다. 그래서 토큰 자체를 서버에 저장하지 않아도 인증이 성립한다.

서명은 **위변조만 막을 뿐 내용은 누구나 디코딩해 볼 수 있다.** payload에는 식별자 수준만 넣고 비밀번호 같은 민감 정보는 절대 넣지 않는다.

## 사용 (jsonwebtoken)

`sign`으로 발급, `verify`로 검증한다.

```js
const jwt = require('jsonwebtoken');
const SECRET = process.env.JWT_SECRET;

// 발급
const token = jwt.sign(
  { userId: user.id, email: user.email },
  SECRET,
  { expiresIn: '1h' }
);
res.json({ token });

// 검증 미들웨어
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  try {
    const payload = jwt.verify(token, SECRET);
    req.user = payload;
    next();
  } catch (e) {
    res.status(401).json({ error: 'invalid token' });
  }
}
```

## 만료 시간 (액세스/리프레시 토큰)

만료가 너무 길면 탈취 시 피해가 커지고, 너무 짧으면 사용자가 자주 재로그인해야 한다. 일반적인 패턴은 **액세스 토큰과 리프레시 토큰을 분리**하는 것이다.

- **액세스 토큰**: 15분 ~ 1시간. 짧게 잡아 탈취 피해를 최소화.
- **리프레시 토큰**: 7일 ~ 30일. 길게 잡아 재로그인 빈도를 줄임. 보통 DB에 저장해 강제 로그아웃(폐기)이 가능하게 한다.
- 액세스가 만료되면 클라이언트가 리프레시 토큰으로 새 액세스를 받아온다.
- 간단한 앱이면 액세스 토큰 하나만 1~2시간으로 잡고 시작해도 무방.

## 클라이언트 저장 위치 (XSS / CSRF 트레이드오프)

- **로컬스토리지**: 다루기 쉽지만 JavaScript에서 접근 가능 → XSS로 토큰이 그대로 탈취될 수 있다.
- **쿠키 (httpOnly)**: JavaScript 접근 불가 → XSS에는 안전. 단, 브라우저가 자동으로 쿠키를 첨부하므로 CSRF에 노출된다.
- **권장**: `httpOnly + Secure + SameSite=Strict(또는 Lax)` 쿠키. XSS를 막고 SameSite로 CSRF도 상당 부분 차단된다. 중요한 작업에는 별도 CSRF 토큰을 더하면 더 단단해진다.
