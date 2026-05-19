# Express 회원가입·인증 학습 정리

대화에서 학습한 내용을 **지식 문서**와 **결과 문서**로 분류하고, 인과 관계(지식 → 결과)에 따라 폴더로 배치했다.

## 폴더 구조

```
.
├── README.md
├── knowledge/                       # 학습한 개념 (각 문서 = 단일 개념 범위)
│   ├── bcrypt-비밀번호-해싱.md
│   └── jwt-stateless-인증.md
└── result/                          # 만들려던 것 (지식을 참조)
    └── express-회원가입-인증-시스템.md
```

폴더 깊이: 최대 2 (루트 기준 `knowledge/파일`, `result/파일`).

## 인과 관계

`knowledge/`의 두 개념(bcrypt, JWT)이 `result/`의 회원가입·인증 시스템을 구성하는 재료가 된다. 결과 문서는 각 지식 문서를 참조하며, 왜 그 지식이 사용되었는지 이유를 함께 기술한다.

## 문서 목록

| 분류 | 문서 | 내용 |
|------|------|------|
| 지식 | `knowledge/bcrypt-비밀번호-해싱.md` | 비밀번호 단방향 해싱, salt, work factor(saltRounds) |
| 지식 | `knowledge/jwt-stateless-인증.md` | JWT 구조·원리, 세션 vs 토큰, 만료 전략, 토큰 저장 위치 |
| 결과 | `result/express-회원가입-인증-시스템.md` | 가입~인증 전체 플로우와 두 지식의 사용 이유 |
