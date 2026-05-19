# 액세스 / 리프레시 토큰 전략

## 핵심 요약

- 본질은 **토큰마다 서로 다른 신뢰 수준을 부여**하는 것이다.
- 액세스 토큰: 자주 쓰여 노출 위험이 높음 → 짧고 가볍게.
- 리프레시 토큰: 드물게 쓰임 → 길고 DB 추적 가능하게.
- 위험은 "짧은 수명 + DB 추적 + 토큰 회전"으로 분산시킨다.

## 왜 두 종류를 쓰는가

액세스 토큰을 짧게(예: 15분) 두면 탈취돼도 피해 시간이 짧다.
하지만 만료될 때마다 재로그인은 사용자 경험이 나쁘다.
그래서 긴 수명(예: 2주)의 리프레시 토큰을 백업으로 두고,
액세스 토큰이 만료되면 리프레시 토큰으로 새 액세스 토큰을 받는다.

검증 시점에 jsonwebtoken이 `exp`를 보고 `TokenExpiredError`를
자동으로 던진다. 그 에러를 잡아 클라이언트가 재발급 요청을 보내게 한다.

## 리프레시 토큰 저장

- 형식은 JWT여도 되고 랜덤한 긴 문자열이어도 된다.
- 중요한 건 "서버가 토큰의 생존 여부를 추적할 수 있어야" 한다는 점.
- 보통 **DB에 저장**해 발급/폐기를 관리한다 → 로그아웃·탈취 시 무효화 가능.
- 클라이언트에는 **httpOnly 쿠키**로 보낸다. JS에서 접근 불가 → XSS로 탈취 어려움.

## 재발급 흐름 (이중 검증)

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
1. `jwt.verify`로 서명·만료 확인
2. DB에 그 토큰이 아직 살아있는지 재확인

액세스용 시크릿과 리프레시용 시크릿은 분리한다.

## 리프레시 토큰 탈취 방어

긴 수명 때문에 탈취 시 더 위험하다. 대응은 두 가지다.

1. **리프레시 토큰 회전(rotation)**: 리프레시 요청마다 새 토큰을 발급하고
   이전 것은 즉시 폐기한다. 공격자가 훔친 토큰을 한 번이라도 쓰면 정상
   사용자의 토큰은 이미 폐기 상태가 되어 다음 정상 요청에서 인증 실패가
   나고, 이를 감지해 해당 사용자의 모든 리프레시 토큰을 일괄 무효화한다.
2. **저장·모니터링**: httpOnly + Secure + SameSite 쿠키로 저장하고,
   DB에 사용자ID·발급시각·기기정보를 함께 두어 의심 패턴 시 강제 로그아웃.

액세스 토큰을 짧게 유지하는 것 역시 이 방어선의 일부다.

## 실무 필수 포인트

- 페이로드는 식별자만
- 서명 검증 + DB 검증의 이중 안전장치
- 토큰별 시크릿 분리
