# 문서 배치 구조

이 디렉토리는 GraphQL 리졸버/에러 처리 학습 대화를 지식 문서와 결과 문서로 정리한 것이다.
폴더는 인과 관계(지식 → 결과)에 따라 구분했다.

## 구조

```
agent_output/
├── README.md
├── knowledge/                       # 지식 문서 (개념 단위)
│   ├── javascript/
│   │   └── async-await-promise.md   # async/await와 Promise 반환 동작
│   └── graphql/
│       ├── resolver-return-value.md # GraphQL 리졸버의 반환값
│       └── error-handling.md        # Apollo Server 에러 처리
└── result/                          # 결과 문서 (만들려던 것)
    └── safe-graphql-resolver-pattern.md  # 안전한 GraphQL 리졸버 + 에러 처리 표준 패턴
```

## 인과 관계

`knowledge/`의 세 개념 문서가 원인(학습한 지식),
`result/`의 표준 패턴 문서가 결과(그 지식으로 만든 것)다.

결과 문서는 세 지식 문서를 참조하며, 각 지식이 왜 사용되었는지를 표로 명시한다.

- `knowledge/graphql/resolver-return-value.md` → 패턴의 출발점(non-null 에러)
- `knowledge/javascript/async-await-promise.md` → undefined 원인 및 코드 간소화 근거
- `knowledge/graphql/error-handling.md` → 4단계 안전 패턴의 핵심 근거
- `result/safe-graphql-resolver-pattern.md` → 위 셋을 종합한 최종 산출물

## 최대 폴더 깊이

`agent_output/` 기준 3 (예: `agent_output/knowledge/graphql/error-handling.md`).
