# Express JWT 인증 시스템

Express 서버의 보호된 라우터에 JWT 기반 인증을 붙인다. 라우터는 이미 만들어져 있는 상태에서 시작한다.

## 전체 흐름

1. 로그인 성공 시점에 서버가 토큰을 발급해 클라이언트에 내려준다.
2. 클라이언트는 토큰을 저장하고, 매 요청마다 Authorization 헤더에 담아 보낸다.
3. 보호된 라우터 앞 미들웨어가 헤더에서 토큰을 꺼내 검증한다.
4. 액세스 토큰이 만료되면 리프레시 토큰으로 새 액세스 토큰을 받아온다.

발급용 라이브러리는 `jsonwebtoken`이며, 서명용 비밀키는 환경변수에 둔다.

## 구성

### 로그인 엔드포인트 — 토큰 발급

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

페이로드에는 식별자(`sub`)와 권한(`role`)만 담는다.

### 리프레시 엔드포인트 — 액세스 토큰 재발급

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

서명 검증 + DB 검증의 두 단계를 모두 통과해야 새 액세스 토큰을 발급한다.

## 설계 결정과 참조 지식

| 결정 | 참조 지식 | 참조 이유 |
|------|-----------|-----------|
| 페이로드에 `sub`, `role`만 담는다 | [JWT](../knowledge/jwt.md) | 페이로드는 base64 인코딩일 뿐 누구나 읽을 수 있으므로 식별 최소 정보만 담아야 한다. 비밀번호·민감 정보는 금지. |
| 액세스 토큰 수명 15분 + 리프레시 토큰을 함께 사용 | [액세스 토큰 / 리프레시 토큰 이중 구조](../knowledge/access-refresh-token.md) | 짧은 액세스 토큰으로 탈취 피해 시간을 줄이고, 그로 인한 재로그인 불편은 긴 수명의 리프레시 토큰으로 메우기 위함. |
| 액세스 토큰과 리프레시 토큰의 시크릿을 분리 | [액세스 토큰 / 리프레시 토큰 이중 구조](../knowledge/access-refresh-token.md) | 두 토큰이 서로 다른 신뢰 수준을 가지므로 서명 키도 분리한다. |
| 리프레시 토큰을 DB에 저장하고 httpOnly 쿠키로 전달 | [리프레시 토큰 저장과 회전](../knowledge/refresh-token-storage.md) | 로그아웃·탈취 시 무효화하려면 서버가 생존 여부를 추적해야 하고, XSS로부터 보호하려면 JS가 접근할 수 없는 저장소가 필요하다. |
| `/refresh`에서 서명 검증 + DB 조회를 모두 수행 | [리프레시 토큰 저장과 회전](../knowledge/refresh-token-storage.md) | 서명만으로는 폐기 여부를 알 수 없어 DB 조회가 필요하다. |
| 리프레시 토큰 회전 적용 | [리프레시 토큰 저장과 회전](../knowledge/refresh-token-storage.md) | 탈취된 토큰이 한 번이라도 사용되면 정상 사용자 측에서 인증 실패가 발생해 탈취를 감지하고 일괄 무효화할 수 있다. |
