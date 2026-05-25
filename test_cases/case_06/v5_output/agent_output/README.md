# 산출물 구조

대화에서 다룬 내용을 Apollo GraphQL 리졸버 패턴 학습과 그 결과로 정리한 "안전한 리졸버 패턴"으로 분류했다.

```
agent_output/
├── README.md
├── knowledge/
│   └── apollo-resolver-patterns.md
└── result/
    └── safe-resolver-pattern.md
```

- `knowledge/apollo-resolver-patterns.md`: 리졸버에서의 async/await 동작, GraphQLError로 의도한 에러 던지기, formatError로 외부 노출 마스킹, Apollo 플러그인으로 에러 로깅 — 모두 같은 Apollo Server 설정·리졸버 호출 자리에서 함께 작동하는 측면이라 한 문서의 섹션으로 묶음.
- `result/safe-resolver-pattern.md`: 위 지식을 조합해 사용자가 도달한 4단계 표준 패턴(리졸버 async 통일 → GraphQLError throw → 플러그인 로깅 → formatError 마스킹).
