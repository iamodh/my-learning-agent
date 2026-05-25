# 세션 관리와 만료 정책

세션은 "사용자가 누구인지"를 한 번 식별한 결과를 이후 요청까지 서버가 들고 있기 위한 메커니즘이다. OAuth 콜백에서 사용자 정보를 받자마자 서버가 세션 ID를 발급해 HttpOnly 쿠키로 브라우저에 심고, 서버는 세션 저장소(Redis나 DB)에 `{ sessionId → userId }`를 들고 있다가 이후 요청의 쿠키로 사용자를 식별한다.

토큰(JWT) 방식도 있으나, 만료와 강제 로그아웃 제어가 더 직관적이라 이 케이스에서는 세션 방식을 선택했다.

## 세션 발급 (express-session 기준)

```js
req.session.userId = profile.email;
```

## 만료 정책 — sliding + absolute 병행

만료가 너무 짧으면 자꾸 로그아웃돼서 UX가 나빠지고, 너무 길면 보안 위험이 커진다. 보통 두 전략을 섞어 쓴다.

- **sliding expiration**: 요청이 올 때마다 만료 시각을 갱신해서, 활동 중인 사용자는 끊기지 않게 한다.
- **absolute expiration**: 활동 여부와 무관하게 "최초 로그인 후 N시간"으로 상한을 둔다.

일반 서비스라면 sliding 30분 + absolute 14일 정도가 무난하고, 결제 같은 민감 액션 직전에는 더 짧은 재인증을 따로 요구하기도 한다.

Redis 같은 저장소라면 키에 TTL을 걸어두면 만료가 자동으로 처리돼서 구현이 깔끔하다.

## 인증 미들웨어로 라우트 보호

민감 라우트(예: 결제 confirm) 앞에는 세션 존재 여부를 검사하는 미들웨어를 둔다. 만료된 세션은 자동 갱신하지 말고 401로 끊어, 사용자에게 재로그인 후 재시도하도록 유도한다 — 결제처럼 금액 정합성·멱등성이 중요한 흐름에서는 어중간한 자동 갱신이 중복 결제 위험을 만든다.

```js
function requireAuth(req, res, next) {
  if (!req.session?.userId) return res.status(401).json({ error: 'login required' });
  next();
}
```
