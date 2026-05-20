# 사이드 프로젝트 결제 confirm 기능

## 만들려는 것

사이드 프로젝트에 PG사 기반 결제 기능을 붙인다. 대화에서는 국내 사이드 프로젝트 기준으로 토스페이먼츠를 추천받았고, 클라이언트 결제창 이후 서버에서 최종 confirm을 호출하는 방향으로 정리됐다.

## 참조 지식

- `knowledge/payment-gateway-confirm.md`
- `knowledge/session-management.md`

## 참조 이유

PG사 기반 결제 승인 지식은 카드사 직접 연동 대신 PG사를 사용하고, `paymentKey`, `orderId`, `amount`를 서버에서 검증한 뒤 confirm API를 호출해 결제를 확정하기 위해 필요하다.

세션 관리 지식은 결제 confirm 엔드포인트가 로그인된 사용자만 호출되도록 하고, 세션의 `userId`와 주문 소유자를 대조하기 위해 필요하다.
