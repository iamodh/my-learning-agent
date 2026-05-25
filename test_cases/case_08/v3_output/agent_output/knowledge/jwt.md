# JWT — 토큰 기반 인증

JWT(JSON Web Token)는 서버가 서명해 발급한 토큰을 클라이언트가 들고 다니며 매 요청마다 첨부하는 인증 방식이다. 서버는 토큰의 서명만 검증하면 되므로 별도 저장소가 필요 없고, 이 stateless 특성 덕분에 SPA나 모바일 앱에서 많이 쓰인다.

> 로그인 상태 유지 방식으로 세션(서버가 세션 ID를 발급하고 메모리/Redis에 사용자 정보를 저장, 클라이언트는 쿠키로 세션 ID만 들고 다님)도 검토했으나 stateless 확장성을 이유로 JWT를 채택했다.

## 구조와 검증 원리

JWT는 `header.payload.signature` 세 부분으로 구성된 문자열이다.

- **header**: 알고리즘 정보.
- **payload**: 사용자 ID 같은 식별 정보.
- **signature**: 서버만 아는 비밀키로 header+payload를 서명한 값.

서버는 토큰 발급 시 서명을 만들고, 받을 때 같은 비밀키로 서명을 다시 계산해 일치 여부만 확인한다. 일치하면 payload를 신뢰할 수 있다 — 누가 중간에 payload를 바꿨다면 서명이 맞지 않기 때문이다. 그래서 서버에 토큰 자체를 저장하지 않아도 인증이 가능하며, 이것이 stateless 인증의 핵심이다.

서명은 위변조만 막을 뿐 내용 자체는 누구나 디코딩해서 볼 수 있다. 따라서 payload에는 식별자 정도만 넣고 비밀번호 같은 민감 정보는 절대 넣지 않는다.

## 사용 방법 — sign / verify

`jsonwebtoken` 라이브러리에서 `sign`으로 발급하고 `verify`로 검증한다.

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

## 만료 시간 — 액세스/리프레시 토큰 분리

토큰 만료는 너무 길면 탈취 시 피해가 크고, 너무 짧으면 사용자가 자주 다시 로그인해야 해서 불편하다. 일반적인 패턴은 액세스 토큰과 리프레시 토큰을 분리하는 것이다.

- **액세스 토큰**: 15분~1시간으로 짧게. 탈취돼도 피해가 제한된다.
- **리프레시 토큰**: 7일~30일로 길게. 액세스가 만료되면 클라이언트가 리프레시 토큰으로 새 액세스를 받아온다. 보통 DB에 저장해 강제 로그아웃(폐기)이 가능하게 만든다.

간단한 앱이면 액세스 토큰 하나만 1~2시간으로 잡고 시작해도 무방하다.
