# JWT (JSON Web Token)

JWT는 서버가 서명해서 발급한, 그 자체로 정보를 담고 있는 문자열이다. 세션 방식이 세션 ID만 주고 실제 정보는 서버에 보관하는 것과 달리, JWT는 사용자 정보를 토큰 안에 직접 담아 보내고 서버는 별도 저장 없이 서명만 검증한다. 그래서 서버를 여러 대로 늘려도 세션 동기화 고민이 줄어드는 대신, 한번 발급된 토큰을 중간에 무효화하기는 어렵다는 트레이드오프가 있다.

구조는 `헤더.페이로드.서명` 세 부분이 점으로 이어진 형태이며, 점 단위로 base64로 인코딩되어 있다.

큰 흐름은 네 단계다: **발급 → 클라이언트가 저장 → 매 요청마다 같이 보냄 → 서버가 검증**.

## 서명 vs 암호화

JWT는 *암호화*가 아니라 *서명*이다. 페이로드는 base64 인코딩일 뿐이라 jwt.io 같은 곳에 붙여넣으면 누구나 내용을 읽을 수 있다. 서명이 보장하는 것은 "이 내용이 발급 이후로 변조되지 않았다"는 것이지 "내용이 비밀이다"가 아니다.

따라서 비밀번호나 민감 정보는 절대 페이로드에 넣으면 안 된다. 정말 숨겨야 하는 데이터라면 JWE라는 별도 방식을 쓰거나, 서버에 두고 ID로만 조회해야 한다.

## 페이로드에 넣을 것

페이로드에는 "이 사용자가 누군지 식별할 수 있는 최소 정보"만 넣는 것이 원칙이다.

- 보통 `sub`(사용자 ID), `role` 정도면 충분하다.
- 표준 클레임: `iss`(발급자), `exp`(만료), `iat`(발급시각).
- 나머지는 본인이 정의하는 커스텀 클레임.
- 너무 많이 넣으면 토큰이 커져서 매 요청마다 전송 비용이 생기므로, 자주 쓸 식별 정보만 담는다.

## 토큰 발급 코드

핵심은 `jwt.sign(payload, secret, options)` 한 줄이다.

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

첫 번째 인자가 페이로드, 두 번째가 서명용 비밀키, 세 번째가 옵션이다. `expiresIn`은 토큰에 만료시각(`exp` 클레임)을 자동으로 박아준다.

## 액세스 토큰 + 리프레시 토큰

만료시간이 짧으면 사용자가 매번 재로그인해야 하므로 보통 두 종류 토큰을 같이 쓴다.

- **액세스 토큰**: 짧은 수명(예: 15분). 평소 API 호출용. 탈취 피해 시간을 최소화하기 위해 짧게 유지한다.
- **리프레시 토큰**: 긴 수명(예: 2주). 액세스 토큰이 만료되면 이걸로 새 액세스 토큰을 받아온다.

검증 시점에 jsonwebtoken이 `exp` 클레임을 보고 자동으로 `TokenExpiredError`를 던지므로, 그 에러를 잡아 클라이언트가 재발급 요청을 보내게 하면 된다.

### 리프레시 토큰 저장 위치와 형식

- JWT로 만들어도 되고, 그냥 랜덤한 긴 문자열이어도 된다. 형식보다 중요한 것은 "서버가 이 토큰이 살아있는지 추적할 수 있어야 한다"는 점이다.
- 보통 DB에 저장해 발급/폐기를 관리한다. 그래야 로그아웃·탈취 시 무효화할 수 있다.
- 클라이언트 쪽에는 httpOnly 쿠키에 담아 보내는 것이 일반적이다. JavaScript에서 접근 못 하니 XSS 공격으로 훔쳐가기 어렵다.
- 액세스 토큰용 시크릿과 리프레시 토큰용 시크릿은 분리한다.

### 리프레시 엔드포인트 코드

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

핵심은 **두 단계 검증**이다. `jwt.verify`로 서명·만료를 보고, 그 다음 DB에 그 토큰이 아직 살아있는지 한 번 더 확인한다.

## 리프레시 토큰 탈취 방어

리프레시 토큰은 수명이 길어 탈취되면 위험이 크다. 주요 대응 두 가지:

1. **리프레시 토큰 회전(rotation)**: 리프레시 요청이 들어올 때마다 새 리프레시 토큰을 발급하고 이전 것은 즉시 폐기한다. 공격자가 훔친 토큰을 한 번이라도 쓰면 정상 사용자의 토큰은 이미 폐기 상태가 되어, 다음에 정상 사용자가 요청할 때 인증 실패가 발생한다. 이를 감지해 해당 사용자의 모든 리프레시 토큰을 한꺼번에 무효화할 수 있다.
2. **저장·전송 보호**: httpOnly + Secure + SameSite 쿠키로 저장하고, DB에 사용자ID·발급시각·기기정보를 함께 저장해 의심 패턴이 보이면 강제 로그아웃시킨다.

액세스 토큰을 짧게 유지하는 것도 결국 이 방어선의 일부다.

## 요약: 실무 포인트

- 페이로드는 식별자만.
- 서명 검증 + DB 검증의 이중 안전장치(리프레시 토큰).
- 토큰별로 시크릿 분리.
- 각 토큰이 서로 다른 신뢰 수준을 갖는다: 액세스는 자주 쓰여 노출 위험이 높으므로 짧고 가볍게, 리프레시는 드물게 쓰이므로 길고 DB 추적이 가능한 형태로.
