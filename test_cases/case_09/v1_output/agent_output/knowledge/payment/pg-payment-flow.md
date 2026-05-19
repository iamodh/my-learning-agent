# PG사 결제 연동

## 핵심 요약
결제는 PG사(Payment Gateway)를 하나 정하는 것이 출발점이다. PG사는 (1) 수십 곳의 카드사 연결을 단일 API로 추상화하고, (2) **카드 번호가 PG사 서버로만 흐르게 해 PCI DSS 보안 규제 부담을 떠넘겨** 준다. 그래서 카드사와 직접 연결하지 않고 PG사를 끼는 것이 사실상 표준이다.

## PG사가 해주는 것
- **추상화**: 비자/마스터/국내 카드사 수십 곳과의 연결을 단일 API로 묶어준다. 직접 하면 카드사마다 따로 계약·연동해야 한다.
- **보안 책임 이전**: 카드 번호가 우리 서버에 닿지 않게 한다. 직접 카드 번호를 받으면 PCI DSS 인증을 우리가 통과해야 해서 비용·책임이 커진다.

대표 PG사: 국내는 토스페이먼츠 / 아임포트(포트원) / KG이니시스, 글로벌은 Stripe. 국내 사이드 프로젝트라면 문서가 깔끔한 토스페이먼츠가 무난하다.

## 결제 흐름: "결제창은 클라이언트, 최종 승인은 서버"
1. 클라이언트에서 PG사 SDK로 결제창을 띄운다.
2. 사용자가 카드 정보를 입력하면 PG사가 결제 결과(`paymentKey`, `orderId`, `amount`)와 함께 success URL로 돌려보낸다.
3. 서버가 그 키로 PG사에 **confirm API를 한 번 더 호출**해야 진짜 결제가 확정된다.

## 서버 confirm 구현 예시 (토스페이먼츠)
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

## 핵심 포인트
**클라이언트가 보낸 금액을 그대로 믿지 말고, 서버 DB의 주문 금액과 대조한 후 confirm**한다. 안 그러면 결제창에서 금액이 조작될 수 있다.
