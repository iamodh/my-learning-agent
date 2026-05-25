# 외부 API 응답 Redis 캐시 레이어

## 무엇을 만들었는가

Express 서비스에서 거의 바뀌지 않는 외부 API 응답을 Redis로 캐싱하는 레이어. 라우트 핸들러 안에서 직접 Redis를 조회하고, 캐시 미스일 때만 외부 API를 호출한 뒤 결과를 저장한다. 클라이언트는 `ioredis`를 사용한다.

## 구현 패턴

```javascript
import Redis from 'ioredis';
const redis = new Redis();

app.get('/api/products/:id', async (req, res) => {
  const key = `product:${req.params.id}`;
  const cached = await redis.get(key);
  if (cached) return res.json(JSON.parse(cached));

  const data = await fetchFromExternalAPI(req.params.id);
  await redis.set(key, JSON.stringify(data), 'EX', 300);
  res.json(data);
});
```

`EX 300`은 TTL 300초(5분) 설정이다.

## 사용된 지식과 참조 이유

- [Redis 동작 원리](../knowledge/redis-basics.md) — 외부 API보다 훨씬 빠른 응답을 얻기 위해 인메모리 키-값 저장소를 캐시 레이어로 두었고, 여러 서버 인스턴스가 같은 캐시를 공유할 수 있는 별도 프로세스 특성이 필요했다.
- [TTL과 만료 처리](../knowledge/ttl.md) — 외부 API 응답이 "거의 안 바뀐다"는 특성에 맞춰 캐시가 영원히 살아 있지 않도록 TTL을 둬야 했다. 예시로는 5분(`EX 300`)을 적용했다.
- [키 네이밍 컨벤션](../knowledge/key-naming.md) — `product:${id}` 처럼 `리소스:식별자` 형태를 써서 충돌을 피하고 가독성을 확보했다.
- [캐시 무효화 전략](../knowledge/cache-invalidation.md) — 원본 데이터가 바뀌는 시점에 TTL만 믿고 기다릴 수 없어서, 변경 직후 `redis.del`로 키를 지워 다음 조회에서 새 값을 채우는 방식을 채택했다(race condition 회피).
- [캐시 스탬피드](../knowledge/cache-stampede.md) — 인기 키 만료 순간 외부 API로 트래픽이 폭주하는 상황을 막기 위해 락 기반 대응을 우선 고려한다.
