# 학습 정리: 안전한 GraphQL 리졸버 에러 처리

이 대화에서는 Apollo Server 기반 GraphQL 리졸버의 반환 규칙과 에러 처리 패턴을 학습했고, 그 결과 네 레이어로 책임이 분리된 리졸버 구성을 정리했다.

## 폴더 구조

```
.
├── README.md
├── knowledge/
│   ├── graphql-resolver-return.md      # 리졸버 반환 규칙 + return await 불필요
│   ├── async-await-behavior.md         # async 함수에서 await 생략 시 동작
│   ├── graphql-error-throw.md          # GraphQLError + extensions.code로 의도한 에러 throw
│   ├── apollo-format-error.md          # formatError로 외부 노출 정책 (safe code 분기)
│   └── apollo-plugin-error-logging.md  # 플러그인 훅으로 요청 컨텍스트 포함 로깅
└── result/
    └── safe-graphql-resolver.md        # 네 레이어로 분리한 리졸버 구성 결과물
```
