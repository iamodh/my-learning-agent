# 외부 API 응답 캐시 (Express + Redis)

## 무엇을 만들었는가

외부 API 호출이 잦은 Express 라우트에 Redis 기반 캐시 계층을 붙여, 응답이 거의 바뀌지 않는 데이터를 매번 외부로 다시 요청하지 않도록 했다. 라우트 핸들러에서 먼저 Redis를 조회하고, 히트면 그대로 반환하고 미스면 외부 API를 호출한 뒤 결과를 TTL과 함께 Redis에 저장한다.

구현은 ioredis 클라이언트를 사용한 cache-aside 패턴이다.

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

데이터가 갱신되는 경로에서는 동일 키에 대해 `redis.del(key)`로 명시적 무효화를 수행해서, 다음 조회 시 새 값으로 자연스럽게 다시 채워지도록 한다.

## 참조한 지식

- [redis-caching.md](./redis-caching.md)

## 참조 이유

- **Redis 캐싱 기본 흐름**: 라우트에서 "조회 → 히트 시 반환 / 미스 시 원본 호출 후 저장" 구조 자체가 이 지식의 cache-aside 패턴 그대로다.
- **TTL**: 외부 API 응답이 "거의 안 바뀌는" 성격이라 일정 시간 후 자동 만료가 필요했다. `set(..., 'EX', 300)`으로 키 단위 TTL을 지정해 영구 캐싱을 막는다.
- **키 네이밍 컨벤션**: `product:${id}` 형태로 콜론 계층 키를 사용해 다른 리소스와 충돌하지 않게 하고, 같은 리소스의 다른 키들을 패턴으로 식별할 수 있도록 했다.
- **캐시 무효화**: DB가 갱신될 때 덮어쓰기 대신 `del`로 키를 지우는 쪽을 택했다. 동시 요청에서 옛 값을 다시 쓰는 race condition을 피하기 위함이다.
- **캐시 스탬피드 인지**: 인기 키가 만료된 순간 외부 API로 트래픽이 몰릴 수 있다는 점을 확인했다. 현재 구현은 기본 cache-aside까지이며, 트래픽이 커지면 락 기반 대응을 추가할 여지를 남겨둔다.
