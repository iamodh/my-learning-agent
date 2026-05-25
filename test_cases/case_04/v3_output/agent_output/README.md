# 외부 API Redis 캐싱 학습 정리

Express 서비스의 외부 API 응답을 Redis로 캐싱하기 위해 학습한 내용과 그 결과 산출물을 정리했다.

## 구조

```
agent_output/
├── README.md
├── knowledge/
│   ├── redis-basics.md       # Redis 동작 원리 (인메모리, 단일 스레드, 별도 프로세스)
│   ├── ttl.md                # TTL 기준과 lazy/active expiration
│   ├── key-naming.md         # 콜론 계층 키 네이밍 컨벤션
│   ├── cache-invalidation.md # del 기반 무효화와 race condition
│   └── cache-stampede.md     # 스탬피드 현상과 락·조기 갱신·stale-while-revalidate
└── result/
    └── api-response-cache.md # Express 라우트 핸들러 캐시 레이어
```
