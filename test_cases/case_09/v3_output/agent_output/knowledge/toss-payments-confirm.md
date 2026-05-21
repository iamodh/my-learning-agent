# 토스페이먼츠 결제창 + 서버 confirm 패턴

토스페이먼츠는 "결제창은 클라이언트, 최종 승인은 서버" 패턴을 따른다. 클라이언트의 결제창 결과만으로는 결제가 확정되지 않고, 서버가 별도로 confirm API를 호출해야 진짜 승인이 떨어진다.

## 흐름

1. 클라이언트에서 SDK로 결제창을 띄운다.
2. 성공 시 `paymentKey`, `orderId`, `amount`를 들고 success URL로 돌아온다.
3. 서버가 이 값을 받아 토스의 confirm API를 호출한다.
4. confirm이 성공해야 결제가 확정된다.

## 서버 confirm 구현

```js
app.post('/payments/confirm', async (req, res) => {
  const { paymentKey, orderId, amount } = req.body;

  // 1) 우리 DB에 저장된 주문 금액과 클라이언트가 보낸 amount 일치 확인 (위변조 방지)
  const order = await db.orders.findById(orderId);
  if (order.amount !== amount) return res.status(400).send('amount mismatch');

  // 2) 토스에 최종 승인 요청
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

## 금액 위변조 방지

클라이언트가 보낸 금액을 그대로 믿고 confirm을 호출하면, 결제창에서 금액이 조작당할 수 있다. 반드시 서버 DB의 주문 금액과 대조한 뒤 confirm을 호출해야 한다.
