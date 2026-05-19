# Express JWT 인증 학습 정리

대화에서 학습한 지식과 그것으로 만들려던 결과를 인과 관계에 따라 분류했다.
`knowledge/`(원인: 학습한 개념)가 `result/`(결과: 만들려던 것)로 이어진다.

## 폴더 구조

```
.
├── README.md                            # 문서 배치 안내 (이 파일)
├── knowledge/                           # 학습한 지식 (개념별 1문서)
│   ├── jwt-basics.md                    # JWT란 무엇인가 / 세션과의 차이 / 구조 / 페이로드
│   └── access-refresh-token-strategy.md # 액세스·리프레시 이중 토큰 전략 / 회전 / 저장
└── result/                              # 지식으로 만들려던 결과물
    └── express-jwt-인증.md              # Express 서버 JWT 인증 구현 (knowledge 참조)
```

최대 폴더 깊이: 2

## 인과 관계

- **knowledge/jwt-basics.md** → result에서 토큰 발급과 페이로드 설계 결정의 근거
- **knowledge/access-refresh-token-strategy.md** → result에서 재발급 흐름과
  탈취 방어 설계의 근거
- **result/express-jwt-인증.md** → 위 두 지식을 사용해 만든 최종 결과물이며,
  각 지식을 왜 사용했는지 이유와 함께 참조한다.
