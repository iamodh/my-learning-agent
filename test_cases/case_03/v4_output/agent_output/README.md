# 학습 결과: Express JWT 인증

대화에서 Express 서버에 JWT 기반 인증을 붙이기 위해 학습한 내용과 그 결과물을 정리한다.

## 문서 배치

```
agent_output/
├── README.md
├── jwt.md                  # 지식 문서: JWT 개념과 액세스/리프레시 토큰 구조
└── express-jwt-auth.md     # 결과 문서: Express에 JWT 인증 적용
```

- `jwt.md` — JWT가 무엇인지, 세션과의 차이, 페이로드/서명 구조, 액세스 토큰과 리프레시 토큰을 함께 쓰는 이유, 리프레시 토큰 회전까지 한 인증 방식의 측면들을 모은 문서.
- `express-jwt-auth.md` — 위 지식을 사용해 Express 서버에 로그인 발급과 리프레시 재발급을 구현한 결과.
