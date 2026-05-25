# Express JWT 인증

기존에 라우터만 만들어둔 Express 서버에 JWT 기반 인증을 붙인다. 로그인 시 액세스 토큰을 발급하고, 별도 `/refresh` 엔드포인트로 액세스 토큰을 재발급한다. 리프레시 토큰은 httpOnly 쿠키로 주고받으며 DB에 저장해 폐기 가능하게 관리한다.

## 흐름

발급 → 클라이언트가 저장 → 매 요청마다 같이 보냄 → 서버가 검증.

## 로그인 — 액세스 토큰 발급

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

페이로드에는 `sub`(사용자 ID)와 `role`만 담는다. 페이로드는 base64로 노출되므로 민감 정보는 넣지 않는다.

## 리프레시 — 액세스 토큰 재발급

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

핵심은 두 단계 검증이다.
- `jwt.verify`로 서명과 만료 확인
- DB에서 해당 토큰이 폐기되지 않았는지 확인

액세스 토큰용 시크릿(`JWT_SECRET`)과 리프레시 토큰용 시크릿(`REFRESH_SECRET`)은 분리해서 쓴다.

## 참조 매핑

| 사용 지식 | 참조 이유 |
|---|---|
| [JWT 인증](../knowledge/jwt-auth.md) | 토큰 발급(`jwt.sign`)·검증(`jwt.verify`) 호출 방식, 페이로드에 식별자만 담는 원칙, 액세스/리프레시 이중 토큰 구조와 회전 패턴, DB 이중 검증의 근거가 모두 이 문서의 개념을 그대로 적용한 것이다. |
