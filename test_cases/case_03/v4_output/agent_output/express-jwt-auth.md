# Express 서버에 JWT 인증 적용

이미 라우터만 만들어 둔 Express 서버에 JWT 기반 인증을 붙인다. 로그인 성공 시점에 액세스 토큰을 발급해 클라이언트에 내려주고, 만료되면 리프레시 토큰으로 새 액세스 토큰을 재발급하는 흐름을 구성한다.

## 전체 흐름

발급 → 클라이언트가 저장 → 매 요청마다 같이 보냄 → 서버가 검증.

구체적으로는:

1. 사용자가 `/login`으로 자격 증명을 보낸다.
2. 서버가 사용자 확인 후 액세스 토큰(짧은 수명)을 발급해 응답한다. 리프레시 토큰은 httpOnly 쿠키로 함께 내려보낸다.
3. 액세스 토큰이 만료되면 클라이언트는 `/refresh`로 리프레시 토큰을 보내 새 액세스 토큰을 받는다.
4. 보호된 라우터는 미들웨어가 Authorization 헤더의 토큰을 검증한 뒤 통과시킨다.

## 로그인: 액세스 토큰 발급

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

페이로드에는 사용자 식별자(`sub`)와 권한(`role`)만 넣는다. `expiresIn: '15m'`으로 액세스 토큰을 짧게 유지해 탈취 시 피해 시간을 줄인다.

## 재발급: `/refresh` 엔드포인트

```js
app.post('/refresh', async (req, res) => {
  const token = req.cookies.refreshToken;
  if (!token) return res.status(401).json({ message: '토큰 없음' });

  try {
    const payload = jwt.verify(token, process.env.REFRESH_SECRET);

    const stored = await db.refreshTokens.findOne({ token });
    if (!stored) return res.status(401).json({ message: '폐기된 토큰' });

    const newAccessToken = jwt.sign(
      { sub: payload.sub, role: payload.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );

    res.json({ accessToken: newAccessToken });
  } catch {
    res.status(401).json({ message: '리프레시 실패' });
  }
});
```

핵심은 이중 검증이다. `jwt.verify`로 서명과 만료를 확인한 뒤, DB에서 해당 리프레시 토큰이 아직 살아있는지 한 번 더 본다. 액세스 토큰용 시크릿(`JWT_SECRET`)과 리프레시 토큰용 시크릿(`REFRESH_SECRET`)은 분리한다.

## 보호된 라우터의 검증 위치

보호된 라우터 앞에 미들웨어를 하나 두고, 들어오는 요청의 Authorization 헤더에서 액세스 토큰을 꺼내 검증한다. 검증 도중 만료가 감지되면 `jsonwebtoken`이 `TokenExpiredError`를 던지므로, 클라이언트가 이 신호를 받아 `/refresh`로 재발급을 요청하게 한다.

## 참조 지식

| 사용한 지식 | 참조 이유 |
|---|---|
| [jwt.md](./jwt.md) — JWT 개념·구조 | 페이로드에 어떤 정보를 넣을지(식별자만), 왜 비밀번호를 넣으면 안 되는지(페이로드 노출) 결정하기 위해 필요했다. |
| [jwt.md](./jwt.md) — 액세스/리프레시 토큰 구조 | 액세스 토큰을 15분으로 짧게 두고 `/refresh`로 재발급하는 두 토큰 구조 자체가 이 결과의 골격이다. |
| [jwt.md](./jwt.md) — 리프레시 토큰 회전과 이중 검증 | `/refresh`에서 `jwt.verify` 다음에 DB 확인을 한 번 더 두는 이유, 시크릿을 분리하는 이유는 회전·폐기·탈취 방어 모델에서 온다. |
