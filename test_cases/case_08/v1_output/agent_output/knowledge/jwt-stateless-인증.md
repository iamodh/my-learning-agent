# JWT 기반 stateless 인증

## 핵심

로그인 성공 후 사용자 상태를 유지하기 위해, 서버가 서명된 토큰(JWT)을 발급한다. 클라이언트는 매 요청마다 토큰을 첨부하고, 서버는 토큰의 서명만 검증한다. 별도 저장소가 필요 없어 stateless하고 확장에 유리하다.

## 세션 방식 vs 토큰 방식

- **세션 방식**: 서버가 세션 ID를 발급하고 메모리/Redis에 사용자 정보를 저장. 클라이언트는 쿠키로 ID만 들고 다닌다.
- **토큰 방식 (JWT)**: 서버가 서명된 토큰을 발급. 클라이언트가 토큰을 들고 다니며 매 요청에 첨부. 서버는 서명만 검증하므로 저장소 불필요. SPA·모바일 앱에서 많이 쓴다.

## JWT 구조와 동작 원리

JWT는 `header.payload.signature` 형태의 문자열이다.

- **header**: 알고리즘 정보
- **payload**: 사용자 ID 같은 식별 정보
- **signature**: 서버만 아는 비밀키로 header+payload를 서명한 값

서버는 발급 시 서명을 만들고, 수신 시 같은 비밀키로 서명을 재계산해 일치하는지만 확인한다. 일치하면 payload를 신뢰할 수 있다 — 누가 payload를 바꿨다면 서명이 안 맞기 때문이다. 그래서 토큰 자체를 서버에 저장할 필요가 없다(stateless의 핵심).

> 주의: 서명은 위변조만 막을 뿐, payload 내용은 누구나 디코딩해 볼 수 있다. 식별자 정도만 넣고 비밀번호 등 민감 정보는 절대 넣지 않는다.

## 사용법

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

## 토큰 만료 전략

- **액세스/리프레시 토큰 분리**: 액세스 토큰은 짧게(15분~1시간) 잡아 탈취 피해를 최소화, 리프레시 토큰은 길게(7~30일) 잡아 재로그인 빈도를 줄인다. 액세스가 만료되면 리프레시 토큰으로 새 액세스를 받는다.
- 리프레시 토큰은 보통 DB에 저장해 강제 로그아웃(폐기)이 가능하게 한다.
- 간단한 앱이면 액세스 토큰 하나만 1~2시간으로 잡고 시작해도 된다.

## 토큰 저장 위치 (클라이언트)

- **로컬스토리지**: 다루기 쉽지만 JavaScript 접근이 가능해 XSS로 토큰이 털릴 수 있다.
- **쿠키**: `httpOnly`를 주면 JS 접근이 막혀 XSS에 안전하나, 브라우저가 자동 첨부하므로 CSRF에 노출된다.
- **권장**: `httpOnly + Secure + SameSite=Strict(또는 Lax)` 쿠키. XSS를 막고 SameSite로 CSRF도 상당 부분 차단한다. 중요한 작업엔 CSRF 토큰을 별도로 두면 더 단단해진다.
