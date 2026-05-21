# 사이드 프로젝트 인증·결제 학습 자료

대화에서 다룬 두 기능(로그인, 결제)을 만들기 위해 학습한 지식과, 그 지식을 실제 적용한 결과 문서를 정리한다.

## 폴더 구조

```
.
├── README.md
├── knowledge/
│   ├── oauth-google.md
│   ├── session-management.md
│   ├── payment-gateway.md
│   └── toss-payments-confirm.md
└── result/
    ├── login-feature.md
    └── payment-feature.md
```

## 문서 목록

### 지식 문서 (`knowledge/`)
- `oauth-google.md` — 구글 OAuth 3단계 인증 흐름
- `session-management.md` — 세션 기반 로그인 상태 유지와 만료 정책
- `payment-gateway.md` — PG사가 결제 흐름에서 맡는 역할
- `toss-payments-confirm.md` — 토스페이먼츠 결제창 + 서버 confirm 패턴

### 결과 문서 (`result/`)
- `login-feature.md` — 사이드 프로젝트의 로그인 기능
- `payment-feature.md` — 사이드 프로젝트의 결제 기능
