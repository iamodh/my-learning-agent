# Express 서버 JWT 인증

## 만들려고 한 것

라우터만 만들어둔 Express 서버에 JWT 기반 인증을 붙인다.
전체 흐름은 네 단계다:

```
발급 → 클라이언트가 저장 → 매 요청마다 같이 보냄 → 서버가 검증
```

구성 요소:
- 로그인 성공 시 액세스 토큰을 발급하는 `/login`
- 보호된 라우터 앞에서 `Authorization` 헤더의 토큰을 검증하는 미들웨어
- 액세스 토큰 만료 시 재발급하는 `/refresh`
- 시작 준비: `npm i jsonwebtoken`, 환경변수에 비밀키 설정

## 사용한 지식과 사용 이유

### [JWT 기초](../knowledge/jwt-basics.md)
- **왜 사용했나**: 인증을 세션이 아닌 JWT로 구현하기로 결정했기 때문.
  서버 확장 시 세션 동기화 부담을 줄이려는 목적이었고, 그 트레이드오프
  (토큰 무효화의 어려움)를 이해해야 후속 설계가 가능했다.
- **어디에 적용됐나**: `jwt.sign`으로 로그인 시 토큰을 발급하는 부분,
  그리고 페이로드에 `sub`/`role` 같은 식별자만 넣고 비밀번호 등 민감
  정보를 배제하는 결정. JWT가 암호화가 아닌 서명이어서 페이로드가
  노출된다는 사실이 이 결정의 직접적 근거다.

### [액세스 / 리프레시 토큰 전략](../knowledge/access-refresh-token-strategy.md)
- **왜 사용했나**: 액세스 토큰을 15분으로 짧게 잡았기 때문에 만료 후
  재로그인 없이 세션을 유지할 방법이 필요했다. 동시에 탈취 위험을
  관리해야 했다.
- **어디에 적용됐나**: `/refresh` 엔드포인트의 이중 검증
  (`jwt.verify` + DB 조회), 리프레시 토큰의 httpOnly 쿠키 저장과
  DB 추적, 토큰 회전 및 토큰별 시크릿 분리. 이 전략이 "짧은 액세스
  토큰"의 불편함을 메우면서 위험을 분산하는 핵심 근거였다.

## 구현 골격

```js
const jwt = require('jsonwebtoken');

// 1. 로그인 시 액세스 토큰 발급
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

// 2. 액세스 토큰 만료 시 재발급 (이중 검증)
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

보호 라우터 앞에는 `Authorization` 헤더에서 토큰을 꺼내
`jwt.verify`로 검증하는 미들웨어를 둔다. `TokenExpiredError`를
잡으면 클라이언트가 `/refresh`로 재발급을 요청하게 한다.

## 최종 설계 요약

액세스 토큰은 짧게 가져가 탈취 피해를 줄이고, 그 불편함은 긴 수명의
리프레시 토큰으로 메우되, 리프레시 토큰은 DB에 추적 가능하게 두고
회전시켜 위험을 분산한다. 페이로드는 노출되므로 식별자만 넣는다.
