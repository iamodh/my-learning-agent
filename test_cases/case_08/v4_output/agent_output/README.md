# Express 회원가입/로그인 인증 시스템

대화에서 학습한 내용을 정리한 문서 모음이다. Express 서버에 회원가입과 로그인 기능을 붙이기 위해 bcrypt(비밀번호 안전 저장)와 JWT(stateless 인증)를 학습했고, 두 기술을 묶어 인증 시스템을 구성했다.

## 문서 배치

```
.
├── README.md
├── knowledge/
│   ├── bcrypt.md           # 비밀번호 단방향 해싱
│   └── jwt.md              # 서명 기반 stateless 토큰 인증
└── result/
    └── express-signup-login-auth.md   # 회원가입 + 로그인 인증 시스템
```

## 인과 관계

- `knowledge/bcrypt.md` — 가입 시점에 비밀번호를 안전하게 저장하기 위해 학습.
- `knowledge/jwt.md` — 로그인 이후 사용자 상태를 stateless하게 유지하기 위해 학습.
- `result/express-signup-login-auth.md` — 위 두 지식을 가입 → 로그인 → 인증 요청의 시간 축에 배치해 만든 결과.
