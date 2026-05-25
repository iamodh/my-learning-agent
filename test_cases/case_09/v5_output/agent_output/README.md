# 사이드 프로젝트 로그인·결제 기능 학습 정리

대화에서 다룬 두 가지 기능(OAuth 로그인, PG 결제)을 구현하기 위해 필요한 지식과 실제 산출물 결정을 정리한다. 두 흐름은 세션 관리를 공통 연결고리로 공유한다.

## 폴더 구조

```
.
├── README.md
├── knowledge/
│   ├── oauth-login.md
│   ├── session-management.md
│   └── pg-payment-flow.md
└── results/
    ├── login-feature.md
    └── payment-feature.md
```

## 문서 배치 의도

- `knowledge/` — 두 기능을 만들기 위해 학습한 개념 단위 지식
  - `oauth-login.md` — OAuth 3단계 흐름과 Express 구현 (추상 개념 + 구체 구현 묶음)
  - `session-management.md` — 세션 발급·쿠키·만료 전략·인증 미들웨어
  - `pg-payment-flow.md` — PG사 역할과 토스페이먼츠 confirm 패턴 (추상 + 구체 묶음)
- `results/` — 실제로 만들기로 결정한 두 기능의 산출물
  - `login-feature.md` — OAuth 로그인 + 세션 발급
  - `payment-feature.md` — 인증 미들웨어 위에 얹은 토스페이먼츠 결제 confirm
