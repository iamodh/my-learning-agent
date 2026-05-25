# 학습 정리: 외부 API 응답 Redis 캐싱

외부 API 호출 비용·지연을 줄이기 위해 Express 서비스에 Redis 캐시를 도입하면서 학습한 내용과 만든 결과를 정리한다.

## 문서 배치

```
agent_output/
├── README.md
├── redis-caching.md                 (지식) Redis 캐싱의 동작 원리·TTL·키 네이밍·무효화·스탬피드
└── external-api-response-cache.md   (결과) Express 라우트에서 외부 API 응답을 Redis로 캐싱한 구현
```

- 지식 문서는 Redis 캐싱 한 주제로 통합되어 있다. TTL·키 네이밍·무효화·캐시 스탬피드는 모두 Redis 캐싱의 하위 동작·옵션·사용 패턴이라 같은 문서의 섹션으로 묶었다.
- 결과 문서는 외부 API 응답 캐싱이라는 한 가지 제작 대상에 대한 구현이다.
