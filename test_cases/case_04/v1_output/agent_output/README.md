# 문서 배치 구조

이 저장소는 "Express 서비스에서 외부 API 응답을 Redis로 캐싱"하기 위해 학습한 지식과,
그 지식으로 만들려던 결과를 인과 관계에 따라 정리한 것이다.

## 인과 관계
지식(knowledge) → 그 지식을 활용한 결과(result) 흐름으로 구성된다.
`redis-basics`는 토대 지식, `cache-operations`는 캐시 운영 지식, `result`는 이를 종합한 산출물이다.

## 폴더 트리

```
.
├── README.md
├── knowledge/
│   ├── redis-basics/
│   │   └── redis-동작-원리.md
│   └── cache-operations/
│       ├── ttl-만료.md
│       ├── 키-네이밍-컨벤션.md
│       ├── 캐시-무효화.md
│       └── 캐시-스탬피드.md
└── result/
    └── 외부-API-응답-Redis-캐싱.md
```

## 문서 안내

### knowledge/redis-basics
- **redis-동작-원리.md** — Redis가 메모리 키-값 저장소로 동작하는 방식, 단일 스레드, 인스턴스 간 공유, Cache-Aside 기본 흐름

### knowledge/cache-operations
- **ttl-만료.md** — TTL 설정 기준과 Redis 만료(lazy/active) 내부 동작
- **키-네이밍-컨벤션.md** — 콜론 계층 키 네이밍과 그 이유
- **캐시-무효화.md** — 데이터 변경 시 `del` 기반 무효화와 race condition 회피
- **캐시-스탬피드.md** — 인기 키 만료 시 외부 API 폭주 방지 전략

### result
- **외부-API-응답-Redis-캐싱.md** — 만들려던 캐싱 기능, 사용된 지식과 그 사용 이유
