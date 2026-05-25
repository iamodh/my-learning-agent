# PG 결제 흐름

PG사(Payment Gateway)는 우리 서버를 카드사 망과 직접 연결하지 않고도 결제를 처리할 수 있게 해주는 중개자다. 결제 기능을 붙일 때 사실상 표준으로 끼는 구성요소다.

## PG사가 해주는 두 가지

1. 비자/마스터/국내 카드사 수십 곳과의 연결을 **단일 API로 추상화**한다. 직접 하려면 카드사마다 따로 계약·연동해야 한다.
2. **카드 번호가 PG사 서버로만 흐르고 우리 서버에는 닿지 않게** 해준다. 이로써 PCI DSS(카드 정보 보안 규제) 부담을 거의 다 PG사로 넘긴다 — 직접 카드 번호를 받으면 우리가 그 인증을 통과해야 하므로 비용·책임이 커진다.

## 공통 흐름

- 클라이언트에서 PG사 SDK로 결제창을 띄운다.
- 사용자가 카드 정보를 입력하면 PG사가 결제 결과(성공/실패 + 결제 키)를 우리 서버로 알려준다.
- 우리 서버는 그 키로 PG사에 "이거 진짜 승인해줘"라고 최종 confirm을 호출한다.

국내에서는 토스페이먼츠, 아임포트(포트원), KG이니시스가 많이 쓰이고 글로벌에서는 Stripe가 표준이다. 이번 사이드 프로젝트는 토스페이먼츠로 진행한다.

## 토스페이먼츠 — "결제창은 클라이언트, 최종 승인은 서버"

클라이언트에서 SDK로 결제창을 띄우면 성공 시 `paymentKey`, `orderId`, `amount`를 들고 success URL로 돌아온다. 서버는 그것을 받아 confirm API를 한 번 더 호출해야 결제가 확정된다.

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

핵심 포인트는 **클라이언트가 보낸 금액을 그대로 믿지 말고 서버 DB의 주문 금액과 대조한 뒤 confirm**한다는 것. 안 그러면 결제창에서 금액을 조작당할 여지가 생긴다.
