# PG사 기반 결제 승인

PG사는 카드사와의 결제 연결을 서비스가 직접 구현하지 않도록 중간에서 결제망을 연결하고, 카드 정보가 서비스 서버에 닿지 않게 해주는 결제 대행 계층이다.

대화에서는 국내 사이드 프로젝트 기준으로 토스페이먼츠가 추천됐고, 결제창은 클라이언트에서 띄우고 최종 승인은 서버에서 confirm API를 호출하는 흐름이 다뤄졌다.

## PG사가 하는 일

PG사는 여러 카드사와의 연결을 단일 API로 추상화한다. 직접 카드사와 연결하려면 카드사마다 계약하고 연동해야 한다.

또한 카드 번호가 PG사 서버로만 흐르고 서비스 서버에는 닿지 않게 해 PCI DSS 같은 카드 정보 보안 규제 부담을 줄여준다. 직접 카드 번호를 받는 방식은 비용과 책임이 커서 사실상 PG사를 끼는 방식이 표준이라고 설명됐다.

## 토스페이먼츠 confirm 흐름

클라이언트에서 PG사 SDK로 결제창을 띄우면 성공 시 `paymentKey`, `orderId`, `amount`를 success URL로 받는다. 서버는 이 값을 받아 서버 DB의 주문 금액과 대조한 뒤 토스페이먼츠 confirm API를 호출해야 결제가 확정된다.

```js
app.post('/payments/confirm', async (req, res) => {
  const { paymentKey, orderId, amount } = req.body;

  const order = await db.orders.findById(orderId);
  if (order.amount !== amount) return res.status(400).send('amount mismatch');

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

핵심은 클라이언트가 보낸 금액을 그대로 믿지 않고, 서버 DB의 주문 금액과 비교한 뒤 confirm을 호출하는 것이다.

## 인증과 주문 소유자 확인

결제 confirm 엔드포인트는 로그인된 사용자만 호출하게 해야 한다. 세션이 없거나 만료됐으면 confirm을 호출하지 않고 401로 끊어 재로그인을 유도한다.

```js
function requireAuth(req, res, next) {
  if (!req.session?.userId) return res.status(401).json({ error: 'login required' });
  next();
}

app.post('/payments/confirm', requireAuth, async (req, res) => {
  // confirm 로직, 그리고 order.userId === req.session.userId 도 같이 확인
});
```

주문이 누구 것인지도 세션의 `userId`와 대조해야 한다. 그렇지 않으면 다른 사람의 주문을 자기 세션으로 confirm하는 문제가 생길 수 있다고 설명됐다.
