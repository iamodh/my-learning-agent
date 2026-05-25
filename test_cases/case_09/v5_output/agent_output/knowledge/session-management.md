# 세션 관리

OAuth로 사용자 정보를 받은 뒤, 매 요청마다 외부 제공자에게 다시 묻지 않기 위해 서버가 자체적으로 사용자 식별 상태를 유지하는 메커니즘이다. 토큰(JWT) 방식도 있지만 만료·강제 로그아웃 컨트롤이 직관적이라는 이유로 세션 방식을 선택했다.

## 세션 발급과 쿠키

콜백에서 사용자 정보를 받자마자 서버가 세션 ID를 발급하고, 그것을 HttpOnly 쿠키로 브라우저에 심는다. 서버는 세션 저장소(Redis나 DB)에 `{ sessionId → userId }`를 들고 있다가, 이후 요청마다 쿠키의 세션 ID로 사용자를 식별한다.

```js
req.session.userId = profile.email; // express-session 기준
```

## 만료 전략 — sliding + absolute

보통 두 가지를 섞어 쓴다.

- **Sliding expiration**: 요청이 올 때마다 만료 시각을 갱신해서 활동 중인 사용자는 끊기지 않게 한다.
- **Absolute expiration**: 활동 여부와 상관없이 "최초 로그인 후 N시간"으로 상한을 둔다.

일반 서비스라면 sliding 30분 + absolute 14일 정도가 무난하다. 결제 같은 민감 액션 직전에는 더 짧은 재인증을 따로 요구하기도 한다. Redis 같은 저장소면 키에 TTL을 걸어두면 알아서 만료되어 구현이 깔끔해진다.

## 인증 미들웨어로 보호 라우트 만들기

세션 검증은 보호가 필요한 라우트 앞에 미들웨어로 깔아 둔다. 세션이 없거나 만료됐으면 본 로직을 호출하지 않고 401로 끊는다.

```js
function requireAuth(req, res, next) {
  if (!req.session?.userId) return res.status(401).json({ error: 'login required' });
  next();
}
```

결제처럼 멱등성·정합성이 중요한 액션에서는 자동 갱신으로 어중간하게 통과시키지 말고 끊고 재로그인을 유도해야 한다 — 두 번 결제되는 사고가 더 위험하다.
