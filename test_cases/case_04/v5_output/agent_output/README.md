# 외부 API 응답 캐싱 학습 정리

외부 API 응답을 Redis로 캐싱하는 방법을 학습하고 Express 라우트에 적용하는 결과로 정리했다.

## 폴더 구조

```
.
├── README.md
├── redis-caching.md          # 지식: Redis 캐싱 (동작, TTL, 키 네이밍, 무효화, 스탬피드)
└── external-api-cache.md     # 결과: 외부 API 응답을 Express에서 Redis로 캐싱
```

## 문서

- [redis-caching.md](./redis-caching.md) — Redis가 캐시로서 어떻게 동작하는지, TTL 설정과 만료 처리, 키 네이밍 컨벤션, 데이터 변경 시 무효화 전략, 캐시 스탬피드 대응
- [external-api-cache.md](./external-api-cache.md) — Express 라우트 핸들러에서 외부 API 응답을 Redis로 캐싱하는 구현 결과
