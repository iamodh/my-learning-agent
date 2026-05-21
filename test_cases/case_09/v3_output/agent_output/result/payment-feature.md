# 사이드 프로젝트 결제 기능

## 무엇을 만들었는가

토스페이먼츠로 결제창을 띄우고, 서버가 confirm API로 최종 승인을 호출하는 결제 기능. 로그인 세션이 있는 사용자만 호출할 수 있고, 주문 소유자와 금액을 서버에서 모두 검증한다.

## 구성

1. 클라이언트가 토스페이먼츠 SDK로 결제창을 띄우고, 성공 시 `paymentKey`, `orderId`, `amount`를 서버로 전달.
2. `/payments/confirm` 엔드포인트 앞에 인증 미들웨어 `requireAuth`를 두어 세션이 없거나 만료됐으면 401로 끊는다.
3. confirm 핸들러에서 두 가지를 검증한 뒤 토스 confirm API 호출.
   - 클라이언트가 보낸 금액과 DB의 주문 금액이 일치하는지.
   - 주문의 소유자(`order.userId`)가 현재 세션의 `userId`와 일치하는지.

```js
function requireAuth(req, res, next) {
  if (!req.session?.userId) return res.status(401).json({ error: 'login required' });
  next();
}

app.post('/payments/confirm', requireAuth, async (req, res) => {
  // confirm 로직, 그리고 order.userId === req.session.userId 도 같이 확인
});
```

세션이 만료된 채로 결제 요청이 들어오면 자동 갱신하지 않고 401로 끊어 재로그인을 유도한다. 결제는 멱등성·금액 정합성이 중요해서 어중간하게 갱신하다 이중 결제가 나는 게 가장 위험하기 때문이다.

## 참조 매핑

- `knowledge/payment-gateway.md`
- `knowledge/toss-payments-confirm.md`
- `knowledge/session-management.md`

## 참조 이유

- **`payment-gateway.md`**: 카드사와 직접 연동할지 PG사를 끼울지 결정해야 했다. PCI DSS 부담을 PG사로 넘길 수 있다는 점이 결정적이라 PG사 사용을 채택했고, 국내 사이드 프로젝트에 적합하다고 판단해 토스페이먼츠를 골랐다.
- **`toss-payments-confirm.md`**: 토스페이먼츠는 클라이언트 결제창 결과만으로 결제가 확정되지 않고 서버 confirm이 필요하다. 또한 클라이언트가 보낸 금액을 그대로 믿으면 위변조에 취약하므로, DB의 주문 금액과 대조한 뒤 confirm을 호출하는 패턴을 그대로 구현했다.
- **`session-management.md`**: confirm 엔드포인트는 로그인된 사용자만 호출해야 하고, 주문 소유자 검증에도 세션의 `userId`가 필요했다. 그래서 결제 라우트 앞에 세션 검사를 강제하는 미들웨어를 두고, 세션 만료 시 자동 갱신 없이 401로 끊어 이중 결제 위험을 차단했다.
