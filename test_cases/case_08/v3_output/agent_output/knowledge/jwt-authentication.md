# JWT 인증

JWT는 로그인 성공 후 서버가 서명된 토큰을 발급하고, 클라이언트가 이후 요청마다 그 토큰을 보내 사용자 상태를 증명하는 방식이다.

## 구조와 동작

JWT는 `header.payload.signature` 세 부분으로 구성된다. header에는 알고리즘 정보, payload에는 사용자 ID 같은 정보, signature에는 서버만 아는 비밀키로 `header + payload`를 서명한 값이 들어간다.

서버는 토큰을 받을 때 같은 비밀키로 서명을 다시 계산해 일치 여부를 확인한다. 누군가 payload를 바꾸면 서명이 맞지 않기 때문에 서버가 토큰을 신뢰하지 않는다. 서버에 토큰 자체를 저장하지 않아도 되는 점이 stateless 인증의 핵심으로 설명됐다.

## 발급과 검증 예시

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

payload에는 식별자 정도만 넣고 비밀번호 같은 민감 정보는 넣지 않는다. 서명은 위변조를 막지만, 내용 자체는 누구나 디코딩해서 볼 수 있기 때문이다.

## 만료와 토큰 분리

대화에서는 액세스 토큰과 리프레시 토큰을 분리하는 패턴이 언급됐다. 액세스 토큰은 15분~1시간처럼 짧게 두고, 리프레시 토큰은 7일~30일처럼 길게 둔다. 액세스 토큰이 만료되면 리프레시 토큰으로 새 액세스 토큰을 받는다.

리프레시 토큰은 보통 DB에 저장해 강제 로그아웃이나 폐기가 가능하게 한다. 간단한 앱이라면 액세스 토큰 하나만 1~2시간으로 잡고 시작하는 방법도 언급됐다.

## 클라이언트 저장 위치

로컬스토리지는 다루기 쉽지만 JavaScript로 접근 가능해 XSS 공격에 토큰이 노출될 수 있다. 쿠키는 `httpOnly` 플래그를 주면 JavaScript에서 접근할 수 없어 XSS에는 안전하지만, 브라우저가 자동으로 쿠키를 첨부하기 때문에 CSRF 위험이 있다.

권장 방식으로는 `httpOnly + Secure + SameSite=Strict` 또는 `SameSite=Lax` 쿠키 저장이 언급됐다. 중요한 작업에는 CSRF 토큰을 별도로 두는 방법도 언급됐다.
