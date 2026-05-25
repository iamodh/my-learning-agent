# Express 회원가입/인증 학습 정리

Express 서버에 회원가입과 로그인 후 인증 유지 기능을 붙이기 위해 학습한 내용과, 그 결과 만들기로 한 구현을 정리한 문서들이다.

## 폴더 구조

```
.
├── README.md
├── knowledge/
│   ├── bcrypt-비밀번호-해싱.md
│   └── jwt-인증.md
└── results/
    ├── 회원가입-엔드포인트.md
    └── 로그인-인증-미들웨어.md
```

## 문서 안내

- `knowledge/bcrypt-비밀번호-해싱.md` — 비밀번호를 단방향 해시로 저장하는 이유, bcrypt의 salt와 work factor, saltRounds 값 선택 기준.
- `knowledge/jwt-인증.md` — JWT의 구성과 stateless 인증 원리, 발급/검증, 만료 정책, 클라이언트 저장 위치 트레이드오프.
- `results/회원가입-엔드포인트.md` — 가입 시 bcrypt.hash로 비밀번호를 저장하는 `/signup` 엔드포인트.
- `results/로그인-인증-미들웨어.md` — `/login`에서 bcrypt.compare + jwt.sign으로 토큰을 발급하고, 이후 요청을 jwt.verify 미들웨어로 인증하는 흐름.
