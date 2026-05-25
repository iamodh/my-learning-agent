# JWT

JWT(JSON Web Token)는 서명된 문자열 토큰으로 사용자 상태를 표현하는 stateless 인증 방식이다. 서버가 토큰을 발급한 뒤 별도로 저장하지 않고, 받은 토큰의 서명만 검증하면 되므로 별도 저장소 없이 인증이 가능하다. SPA나 모바일 앱에서 많이 쓰인다.

대화 초기엔 세션 방식(서버가 세션 ID 발급 + 메모리/Redis에 사용자 정보 저장 + 클라이언트는 쿠키로 ID 보관)도 후보로 검토했으나, stateless하고 확장하기 좋다는 이유로 JWT로 결정했다.

## 구조

JWT는 세 부분으로 구성된 문자열이다: `header.payload.signature`.

- **header**: 알고리즘 정보.
- **payload**: 사용자 ID 같은 정보.
- **signature**: 서버만 아는 비밀키로 `header+payload`를 서명한 값.

서버는 발급 시 서명을 만들고, 받을 때 같은 비밀키로 다시 서명을 계산해 일치하는지만 확인한다. 일치하면 payload를 신뢰할 수 있다 — 누가 중간에 payload를 바꿨다면 서명이 안 맞기 때문이다. 이것이 stateless 인증의 핵심이다.

payload는 누구나 디코딩해서 볼 수 있다. 서명은 위변조만 막을 뿐 내용을 숨기지 않으므로, payload에는 식별자 정도만 넣고 비밀번호 같은 민감 정보는 절대 넣지 않는다.

## 사용 방법

`jsonwebtoken` 라이브러리의 `sign`으로 발급하고 `verify`로 검증한다.

```js
const jwt = require('jsonwebtoken');
const SECRET = process.env.JWT_SECRET;

// 발급
const token = jwt.sign(
  { userId: user.id, email: user.email },
  SECRET,
  { expiresIn: '1h' }
);

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

## 만료 시간과 리프레시 토큰

토큰 만료는 보안과 사용자 편의의 트레이드오프다. 너무 길면 탈취 시 피해가 크고, 너무 짧으면 사용자가 자주 로그인해야 한다.

일반적인 패턴은 액세스 토큰과 리프레시 토큰을 분리하는 것이다.

- **액세스 토큰**: 짧게(15분 ~ 1시간). 탈취돼도 피해를 최소화.
- **리프레시 토큰**: 길게(7일 ~ 30일). 액세스 만료 시 새 액세스를 받아오는 데 사용. 보통 DB에 저장해 강제 로그아웃(폐기)이 가능하게 만든다.

간단한 앱이면 액세스 토큰 하나만 1~2시간으로 잡고 시작해도 된다.

## 클라이언트 저장 위치

JWT를 클라이언트 어디에 저장할지도 트레이드오프가 있다.

- **로컬스토리지**: 다루기 쉽지만 JavaScript로 접근 가능 → XSS 공격에 토큰이 그대로 탈취될 수 있다.
- **쿠키**: `httpOnly` 플래그를 주면 JavaScript에서 접근 못 함 → XSS에는 안전. 단 브라우저가 자동으로 쿠키를 첨부하므로 CSRF 공격에 노출.

권장 방식은 `httpOnly` + `Secure` + `SameSite=Strict`(또는 `Lax`) 쿠키에 저장하는 것. XSS도 막고 SameSite로 CSRF도 상당 부분 차단된다. 중요한 작업엔 CSRF 토큰을 추가로 두면 더 단단해진다.
