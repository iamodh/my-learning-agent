# 결제 기능

토스페이먼츠로 결제 confirm을 처리한다. 결제창은 클라이언트가 띄우고, 최종 승인은 로그인된 사용자의 세션을 검증한 뒤 서버에서 호출한다.

## 구현 구성

- `POST /payments/confirm` — 토스 success URL에서 받은 `paymentKey`, `orderId`, `amount`를 받아 토스 confirm API를 호출.
- 라우트 앞에 `requireAuth` 미들웨어를 두어 세션이 없거나 만료됐으면 401로 끊는다 (자동 갱신은 두 번 결제 위험이 있어 피한다).
- confirm 전에 두 가지를 대조한다:
  - **금액 정합성**: 클라이언트가 보낸 `amount`와 DB의 주문 금액 일치 여부.
  - **주문 소유자**: `order.userId === req.session.userId` — 남의 주문을 자기 세션으로 confirm 치는 시나리오를 막는다.
- 토스 confirm이 성공하면 주문을 `paid` 상태로 마크한다.

```js
app.post('/payments/confirm', requireAuth, async (req, res) => {
  // 위의 confirm 로직, 그리고 order.userId === req.session.userId 도 같이 확인
});
```

## 참조 매핑

| 사용한 지식 | 참조 이유 |
|---|---|
| `knowledge/pg-payment-flow.md` | PG사가 왜 필요한지(카드사 연결 추상화, PCI DSS 부담 이전)와 토스페이먼츠의 confirm 패턴·금액 위변조 방지 로직을 그대로 적용한다. |
| `knowledge/session-management.md` | 결제 라우트 보호를 위해 `requireAuth` 미들웨어를 깔고, 세션의 `userId`로 주문 소유자까지 함께 검증한다. |
