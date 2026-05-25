# 사이드 프로젝트 로그인·결제 학습 정리

사이드 프로젝트에 구글 OAuth 로그인과 토스페이먼츠 결제를 붙이기 위해 학습한 개념과, 그 결과 만들기로 한 두 가지 기능을 정리한다.

## 구조

```
.
├── README.md
├── login-google-oauth.md        # 결과: 구글 OAuth 로그인 + 세션 발급
├── toss-payment-confirm.md      # 결과: 토스페이먼츠 결제 confirm 엔드포인트
└── knowledge/
    ├── oauth.md                 # OAuth 개념 + 구글 OAuth Express 구현
    ├── session.md               # 세션 관리와 만료 정책 (sliding/absolute)
    └── payment-gateway.md       # PG사 개념 + 토스페이먼츠 confirm 패턴
```

## 인과 관계

- 로그인·결제 두 흐름의 공통 기반은 "세션으로 사용자 상태를 유지한다"는 점이다.
- `oauth.md` → 사용자를 식별하고, `session.md` → 식별 결과를 이후 요청까지 들고 다닌다 → `login-google-oauth.md`로 합쳐진다.
- `payment-gateway.md` → 결제는 PG사에 confirm을 호출해 확정한다 → `toss-payment-confirm.md`로 합쳐지며, 이때 `session.md`가 인증 미들웨어로 다시 쓰인다.
