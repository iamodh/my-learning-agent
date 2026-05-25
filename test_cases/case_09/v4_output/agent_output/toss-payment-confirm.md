# 토스페이먼츠 결제 confirm 엔드포인트

## 무엇을 만드는가

사이드 프로젝트의 결제 확정 엔드포인트 `POST /payments/confirm`. 클라이언트가 토스 SDK로 결제창을 띄워 성공한 뒤 `paymentKey`, `orderId`, `amount`를 들고 호출하면, 서버가 다음을 수행한다.

1. 로그인 세션이 있는지 인증 미들웨어로 검사한다. 없거나 만료됐으면 401로 끊어 재로그인을 유도한다 (자동 갱신하지 않는다).
2. DB의 주문 금액과 요청 본문의 `amount`가 일치하는지 확인한다. 일치하지 않으면 400.
3. 주문의 소유자(`order.userId`)가 세션의 `userId`와 일치하는지 확인한다. 남의 주문을 자기 세션으로 confirm 치지 못하도록 막는다.
4. 토스 confirm API를 호출해 최종 승인을 받고, 성공 시 주문 상태를 결제 완료로 마킹한다.

```js
app.post('/payments/confirm', requireAuth, async (req, res) => {
  const { paymentKey, orderId, amount } = req.body;

  const order = await db.orders.findById(orderId);
  if (order.amount !== amount) return res.status(400).send('amount mismatch');
  if (order.userId !== req.session.userId) return res.status(403).send('forbidden');

  const secret = Buffer.from(`${process.env.TOSS_SECRET_KEY}:`).toString('base64');
  const tossRes = await fetch('https://api.tosspayments.com/v1/payments/confirm', {
    method: 'POST',
    headers: {
      Authorization: `Basic ${secret}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ paymentKey, orderId, amount }),
  });

  if (!tossRes.ok) return res.status(400).json(await tossRes.json());

  const payment = await tossRes.json();
  await db.orders.markPaid(orderId, payment);
  res.json({ ok: true });
});
```

## 어떤 지식을 썼는가 (참조 매핑)

- [`knowledge/payment-gateway.md`](knowledge/payment-gateway.md) — PG사 역할과 토스페이먼츠 confirm 패턴
- [`knowledge/session.md`](knowledge/session.md) — 인증 미들웨어로 세션 검사

## 왜 그 지식을 썼는가 (참조 이유)

- **PG사·토스 confirm 패턴**: 결제는 클라이언트가 보낸 금액을 그대로 믿을 수 없고, 서버 DB 주문 금액과 대조한 뒤 PG사에 confirm을 호출해야 확정된다. 위변조 방지와 결제 확정의 단일 진실원천이 PG사 confirm 응답이라는 점이 그대로 이 엔드포인트의 골격이 된다.
- **세션**: 결제는 멱등성·금액 정합성이 민감해서 만료된 세션을 자동 갱신해서는 안 된다. 인증 미들웨어로 세션을 선검사해, 인증이 없는 요청은 토스 호출 이전에 차단한다. 또한 세션의 `userId`로 주문 소유권을 검증한다.
