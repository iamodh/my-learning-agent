# 문서 배치 구조

이 대화에서 학습한 것을 **지식**과 **결과**로 나눴다. 결과 문서가
지식 문서를 참조하는 인과 관계에 따라 폴더를 분리했다.

```
.
├── README.md                          # 이 파일 — 배치 구조
├── result/                            # 만들려던 것
│   └── react-express-api-연동.md      # React → Express API 호출 구조
└── knowledge/                         # 사용된 개념
    └── cors.md                        # CORS (프리플라이트·Express 적용 포함)
```

## 인과 관계

- `result/react-express-api-연동.md` 가 제작 의도이며,
  그것을 가능하게 한 `knowledge/cors.md` 를 *왜 필요했는지*와 함께
  참조한다.
- 지식은 단일 개념(CORS) 범위로 묶었고, 프리플라이트와 Express
  미들웨어 적용은 별도 문서로 쪼개지 않고 해당 개념의 섹션으로 통합했다.
