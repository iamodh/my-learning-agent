# 리프레시 토큰

리프레시 토큰은 짧은 수명의 액세스 토큰이 만료됐을 때 새 액세스 토큰을 받기 위해 사용하는 토큰이다. 액세스 토큰은 탈취 피해 시간을 줄이기 위해 짧게 두고, 사용자가 매번 다시 로그인하지 않도록 리프레시 토큰을 함께 둔다.

## 기본 흐름

대화에서 다룬 구조는 짧은 수명의 액세스 토큰과 긴 수명의 리프레시 토큰을 함께 쓰는 방식이다.

- 액세스 토큰: 평소 API 호출에 사용하며 짧게 유지한다.
- 리프레시 토큰: 액세스 토큰이 만료되면 새 액세스 토큰을 받는 데 사용한다.
- `jsonwebtoken`은 검증 시 `exp` 클레임을 보고 만료된 토큰에 대해 `TokenExpiredError`를 던진다.

## 저장과 검증

리프레시 토큰은 JWT로 만들 수도 있고 랜덤한 긴 문자열이어도 된다. 형식보다 중요한 점은 서버가 이 토큰이 살아 있는지 추적할 수 있어야 한다는 것이다.

대화에서는 리프레시 토큰을 DB에 저장해 발급과 폐기를 관리하는 방식이 다뤄졌다. 그래야 로그아웃이나 탈취 시 무효화할 수 있다. 클라이언트 쪽에는 보안상 `httpOnly` 쿠키에 담아 보내는 방식이 언급됐다.

## 재발급 예시

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

핵심은 `jwt.verify`로 서명과 만료를 확인하고, DB에서 해당 리프레시 토큰이 아직 살아 있는지 다시 확인하는 것이다. 대화에서는 액세스 토큰용 시크릿과 리프레시 토큰용 시크릿을 분리하는 것이 좋다고 다뤄졌다.

## 회전

리프레시 토큰은 수명이 길기 때문에 탈취되면 위험하다. 대화에서 다룬 대응은 리프레시 토큰 회전이다.

리프레시 요청이 들어올 때마다 새 리프레시 토큰을 발급하고 이전 토큰은 즉시 폐기한다. 탈취된 토큰이 사용되면 정상 사용자의 다음 요청에서 인증 실패가 발생할 수 있고, 이를 감지해 해당 사용자의 모든 리프레시 토큰을 무효화할 수 있다.

대화에서는 `httpOnly`, `Secure`, `SameSite` 쿠키 저장과 DB에 사용자 ID, 발급 시각, 기기 정보를 저장해 의심스러운 패턴을 볼 수 있게 하는 방식도 함께 언급됐다.
