# 결제 요청의 인증 가드

## 핵심 요약
결제 confirm 엔드포인트도 결국 로그인된 사용자가 호출한다. **결제 라우트 앞에 인증 미들웨어를 두어**, 세션이 없거나 만료됐으면 confirm을 호출하지 않고 401로 끊는 것이 정석이다. 결제는 멱등성·금액 정합성이 중요해서 어중간한 자동 갱신은 이중 결제 위험을 만든다.

## 왜 자동 갱신이 위험한가
세션 만료 상태에서 결제가 들어왔을 때 자동으로 세션을 갱신해 진행하면, 어중간한 상태에서 **두 번 결제**되는 시나리오가 가장 위험하다. 그래서 만료 시 깔끔하게 끊고, 사용자에게 재로그인 후 다시 결제하도록 유도한다.

## 구현 예시
```js
function requireAuth(req, res, next) {
  if (!req.session?.userId) return res.status(401).json({ error: 'login required' });
  next();
}

app.post('/payments/confirm', requireAuth, async (req, res) => {
  // confirm 로직, 그리고 order.userId === req.session.userId 도 같이 확인
});
```

## 두 가지 검증을 반드시 함께
1. **세션 존재/유효성**: 없거나 만료면 401.
2. **주문 소유자 대조**: `order.userId === req.session.userId`. 안 그러면 남의 주문을 자기 세션으로 confirm 치는 시나리오가 열린다.
