# 문서 배치 구조

이 학습 기록은 **지식(knowledge) → 결과(result)** 의 인과 관계로 구성된다.
CORS 개념을 학습했고, 그 지식을 사용해 React + Express API 연동에서 CORS 에러를 해결했다.

## 폴더 구조

```
agent_output/
├── README.md                          # 본 문서 (배치 구조 안내)
├── knowledge/                         # 학습한 개념 (원인)
│   └── cors.md                        # CORS: 에러 원인, 헤더, 프리플라이트, 자격 증명
└── result/                            # 지식으로 만든 것 (결과)
    └── react-express-api-cors-연동.md  # React+Express 연동 시 CORS 에러 해결
```

## 인과 관계

| 구분 | 문서 | 설명 |
|------|------|------|
| 지식 (원인) | `knowledge/cors.md` | 브라우저가 왜 CORS 에러를 내는지, 어떤 응답 헤더가 필요한지, 프리플라이트는 무엇인지 |
| 결과 | `result/react-express-api-cors-연동.md` | 위 지식을 적용해 `cors` 미들웨어로 React-Express 연동 문제를 해결 |

결과 문서는 `knowledge/cors.md`를 참조하며, 각 지식이 왜 필요했는지 이유를 함께 기록한다.
