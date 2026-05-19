# 학습 정리: 로그인 + 결제 기능

이 대화에서는 사이드 프로젝트에 로그인과 결제를 함께 붙이기 위한 지식을 학습했다.
문서는 **인과 관계**에 따라 배치했다. 지식 문서(`knowledge/`)가 토대이고,
그 지식들이 모여 만들려던 결과(`result/`)로 이어진다.

## 폴더 구조

```
.
├── README.md
├── knowledge/
│   ├── auth/
│   │   ├── oauth.md                  # OAuth 외부 인증 (사용자 식별)
│   │   └── session-management.md     # 세션 관리·만료 정책 (상태 유지, 두 흐름의 연결고리)
│   └── payment/
│       ├── pg-payment-flow.md        # PG사 결제 연동 (금전 흐름·보안 책임 이전)
│       └── payment-auth-guard.md     # 결제 요청 인증 가드 (세션을 결제에 적용)
└── result/
    └── login-and-payment-design.md   # 만든 것: 로그인+결제 설계 (위 지식 참조)
```

## 인과 흐름

OAuth(식별) → 세션(상태 유지) → 결제 인증 가드(세션을 결제에 적용) → PG 결제(금전 흐름)

`knowledge/auth`는 로그인 흐름, `knowledge/payment`는 결제 흐름이며,
세션 관리가 두 흐름을 잇는 핵심 연결 지점이다.
결과 문서는 각 지식이 **왜** 사용되었는지를 함께 설명한다.
