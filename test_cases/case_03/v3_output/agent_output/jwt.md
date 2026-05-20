# JWT

JWT는 서버가 서명해서 발급한, 그 자체로 정보를 담고 있는 문자열이다. 구조는 `헤더.페이로드.서명` 세 부분이 점으로 이어진 형태이며, 각 부분은 base64로 인코딩된다.

## 세션과의 차이

세션 방식은 서버가 세션 ID를 클라이언트에 주고 실제 정보는 서버 메모리나 DB에 보관한다. JWT는 사용자 정보를 토큰 안에 담아 보내고, 서버는 별도 저장 없이 서명을 검증한다.

이 방식은 서버를 여러 대로 늘릴 때 세션 동기화 부담을 줄일 수 있지만, 한 번 발급된 토큰을 중간에 무효화하기 어렵다는 트레이드오프가 있다.

## 발급 예시

로그인 성공 시점에 `jwt.sign(payload, secret, options)`로 토큰을 발급한다.

```js
const jwt = require('jsonwebtoken');

app.post('/login', async (req, res) => {
  const user = await findUser(req.body.email, req.body.password);
  if (!user) return res.status(401).json({ message: '인증 실패' });

  const accessToken = jwt.sign(
    { sub: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );

  res.json({ accessToken });
});
```

첫 번째 인자는 페이로드, 두 번째 인자는 서명용 비밀키, 세 번째 인자는 옵션이다. `expiresIn`은 토큰에 만료 시각인 `exp` 클레임을 자동으로 넣는다.

## 페이로드

페이로드에는 사용자를 식별할 수 있는 최소 정보만 넣는다. 대화에서는 `sub` 사용자 ID와 `role` 정도가 예시로 다뤄졌다.

비밀번호나 주민번호 같은 민감 정보는 넣지 않는다. JWT의 페이로드는 암호화된 것이 아니라 base64로 인코딩된 것이므로 누구나 내용을 읽을 수 있다.

서명이 보장하는 것은 내용이 발급 이후 변조되지 않았다는 점이지, 내용이 비밀이라는 점이 아니다.

## 클레임

대화에서 다룬 표준 클레임은 다음과 같다.

- `sub`: 사용자 식별자
- `iss`: 발급자
- `exp`: 만료 시각
- `iat`: 발급 시각

나머지는 직접 정의하는 커스텀 클레임으로 둘 수 있다. 토큰은 매 요청마다 전송되므로 너무 많은 정보를 넣지 않는다.
