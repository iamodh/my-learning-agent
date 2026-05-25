# 학습 산출물

이번 학습은 GraphQL 리졸버에서 자주 만나는 함정(non-nullable 에러, Promise 미반환, await 누락)을 해결하고, Apollo Server에서 에러를 분기·로깅·마스킹하는 안전한 리졸버 패턴을 만드는 과정이었다.

## 구조

```
.
├── README.md
├── graphql-resolver-async.md      # 리졸버 반환·async/await 동작
├── apollo-error-handling.md       # GraphQLError, formatError, 로깅 플러그인
└── safe-graphql-resolver-pattern.md  # 최종 조립한 안전 패턴 (결과)
```

- `graphql-resolver-async.md`: 리졸버가 값을 어떻게 반환해야 하는지, async 함수가 await 없이 동작할 때 무슨 일이 일어나는지.
- `apollo-error-handling.md`: Apollo Server에서 의도된 에러와 예상 못한 에러를 구분해 처리하는 도구들 (`GraphQLError`, `formatError`, 플러그인 훅).
- `safe-graphql-resolver-pattern.md`: 위 지식을 조립해서 만든 4단계 표준 리졸버 패턴.
