# JWT 인증

JWT(JSON Web Token)는 서버가 서명한 토큰을 클라이언트가 들고 다니면서 매 요청마다 첨부하는 stateless 인증 방식이다. 서버는 토큰의 서명만 검증하면 되므로 별도 세션 저장소가 필요 없고, SPA나 모바일 앱에서 확장하기 좋아 많이 쓰인다.

대안으로 세션 방식(서버가 세션 ID를 발급하고 메모리/Redis에 사용자 정보를 저장, 클라이언트는 쿠키로 ID를 들고 다님)이 있으나, 여기서는 stateless 특성을 우선해 JWT를 선택했다.

## 토큰 구조와 stateless 원리

JWT는 세 부분으로 구성된 문자열이다: `header.payload.signature`.

- **header**: 서명 알고리즘 정보.
- **payload**: 사용자 ID 같은 식별 정보.
- **signature**: 서버만 아는 비밀키로 header+payload를 서명한 값.

서버는 발급 시 서명을 만들고, 검증 시 같은 비밀키로 서명을 다시 계산해 일치하는지만 확인한다. 일치하면 payload를 신뢰할 수 있다 — 누가 payload를 바꿨다면 서명이 맞지 않기 때문이다. 그래서 서버에 토큰 자체를 저장할 필요가 없고, 이것이 stateless 인증의 핵심이다.

서명은 위변조를 막을 뿐, payload 내용 자체는 누구나 디코딩해서 볼 수 있다. 따라서 payload에는 식별자 정도만 넣고 비밀번호 같은 민감 정보는 절대 넣지 않는다.

## 발급과 검증

Node에서는 jsonwebtoken 라이브러리의 `sign`으로 발급하고 `verify`로 검증한다.

```js
const jwt = require('jsonwebtoken');

// 발급
const token = jwt.sign(
  { userId: user.id, email: user.email },
  SECRET,
  { expiresIn: '1h' }
);

// 검증
const payload = jwt.verify(token, SECRET);
```

`verify`가 통과하면 payload를 신뢰하고 요청 처리에 활용한다.

## 만료 정책

토큰 만료를 너무 길게 잡으면 탈취 시 피해가 커지고, 너무 짧게 잡으면 사용자가 자주 로그인해야 해 불편하다. 일반적인 패턴은 액세스 토큰과 리프레시 토큰을 분리하는 것이다.

- **액세스 토큰**: 짧게 — 15분 ~ 1시간. 탈취 피해 최소화.
- **리프레시 토큰**: 길게 — 7일 ~ 30일. 자주 로그인 안 해도 되게.
- 액세스가 만료되면 클라이언트가 리프레시 토큰으로 새 액세스를 받아온다.
- 리프레시 토큰은 보통 DB에 저장해서 강제 로그아웃(폐기)이 가능하게 만든다.

간단한 앱이면 액세스 토큰 하나만 1~2시간으로 잡고 시작해도 괜찮다.

## 클라이언트 저장 위치

토큰을 어디에 저장하느냐는 트레이드오프가 있다.

- **로컬스토리지**: 다루기 쉽지만 JavaScript로 접근 가능해 XSS 공격에 토큰이 그대로 털릴 수 있다.
- **쿠키**: `httpOnly` 플래그를 주면 JavaScript에서 접근할 수 없어 XSS에 안전하지만, 브라우저가 자동으로 쿠키를 첨부하기 때문에 CSRF 공격에 노출된다.

권장되는 방식은 `httpOnly` + `Secure` + `SameSite=Strict`(또는 `Lax`) 쿠키에 저장하는 것이다. XSS를 막고 SameSite로 CSRF도 상당 부분 차단된다. 중요한 작업에는 추가로 CSRF 토큰을 별도로 두면 더 단단해진다.
