# Express 회원가입/인증 학습 노트

Express 서버에 회원가입과 로그인 기능을 만들기 위해 학습한 내용을 정리한다.
가입 시점의 비밀번호 보관(bcrypt)과 로그인 이후의 상태 유지(JWT)를 두 축으로 다룬다.

## 폴더 구조

```
.
├── README.md
├── results/
│   └── express-signup-auth.md        # 결과 문서: Express 회원가입/로그인 구현
└── knowledge/
    ├── bcrypt.md                     # 비밀번호 해싱: bcrypt
    ├── jwt.md                        # 토큰 기반 인증: JWT
    └── jwt-storage.md                # JWT 클라이언트 저장 위치 선택
```

## 문서 인과 관계

- `results/express-signup-auth.md`가 만들고자 하는 대상이다.
- 그 안에서 `knowledge/bcrypt.md`(가입 시점)와 `knowledge/jwt.md`(로그인 이후 상태 유지)를 참조한다.
- `knowledge/jwt-storage.md`는 JWT를 어디에 둘지에 대한 독립 결정이라 별도 지식 문서로 둔다. 세션 방식은 검토 후 탈락한 대안이므로 별도 문서로 만들지 않고 `knowledge/jwt.md` 본문에 한 줄로만 남긴다.
