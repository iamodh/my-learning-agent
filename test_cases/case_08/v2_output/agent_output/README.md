# case_08 산출물

Express 서버에 회원가입·로그인 기능을 만들면서 학습한 지식과 결과를 정리한 문서들이다.

## 구조

```
agent_output/
├── README.md
├── express-auth-feature.md   # 결과 문서: Express 회원가입·로그인 기능
├── bcrypt.md                 # 지식 문서: 비밀번호 해싱 (saltRounds 포함)
└── jwt-auth.md               # 지식 문서: JWT 인증 (만료/저장 위치 포함)
```

## 인과 관계

- `express-auth-feature.md`가 최종 결과 문서이고, 그 안에서 두 지식 문서(`bcrypt.md`, `jwt-auth.md`)를 참조한다.
- 가입 시점의 비밀번호 보관 책임은 `bcrypt`, 로그인 이후의 상태 증명 책임은 `JWT`가 맡는다.
