# PG사(Payment Gateway)와 토스페이먼츠 confirm 패턴

PG사는 카드사 수십 곳과의 연결을 단일 API로 추상화해 주고, 더 중요하게는 카드 번호가 PG사 서버로만 흐르고 우리 서버에는 닿지 않도록 해주는 중개자다. 이를 통해 PCI DSS(카드 정보 보안 규제) 부담을 거의 다 PG사로 넘긴다. 기술적으로 카드사와 직접 연결도 가능하지만, 카드사별 계약과 PCI DSS 인증 부담이 커서 사실상 PG사를 끼는 것이 표준이다.

## 결제 흐름

클라이언트에서 PG사 SDK로 결제창을 띄우면, 사용자가 카드 정보를 PG사에 직접 입력한다. PG사는 결제 결과(성공/실패 + 결제 키)를 우리 서버에 알리고, 우리 서버는 그 키로 PG사에 "이거 진짜 승인해줘"라는 최종 confirm을 호출한다.

국내 사이드 프로젝트에서는 문서가 깔끔한 토스페이먼츠를 선택했다. 후보로는 아임포트(포트원), KG이니시스도 거론되며, 글로벌이면 Stripe가 표준이다.

## 토스페이먼츠 — "결제창은 클라이언트, 최종 승인은 서버" 패턴

클라이언트가 SDK로 결제창을 띄우고 성공 시 `paymentKey`, `orderId`, `amount`를 들고 success URL로 돌아온다. 서버는 그것을 받아 confirm API를 호출해야 결제가 확정된다.

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

핵심은 "클라이언트가 보낸 금액을 그대로 믿지 말고, 서버 DB의 주문 금액과 대조한 뒤에 confirm을 호출한다"는 점이다. 그렇지 않으면 결제창에서 금액을 조작당할 수 있다.
