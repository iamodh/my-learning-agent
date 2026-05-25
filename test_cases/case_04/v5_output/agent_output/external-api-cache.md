# 외부 API 응답 캐싱 (Express + Redis)

## 무엇을 만들었는가

외부 API 호출 결과가 거의 바뀌지 않는데도 매번 호출해 비용과 지연이 늘던 문제를 해결하기 위해, Express 라우트 핸들러에서 Redis를 캐시로 끼워 넣는 구조를 만들었다. 요청이 들어오면 먼저 Redis에서 키로 조회해 히트면 그대로 반환하고, 미스면 외부 API를 호출한 뒤 결과를 TTL과 함께 Redis에 저장한다. 데이터가 바뀌는 시점에는 해당 키를 `del`로 무효화해 다음 조회에서 새 값으로 다시 채워지게 한다.

## 구현

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

구현에 포함된 설계 선택:

- **키 형식**: `product:<id>` 콜론 계층 컨벤션을 따라, 리소스 종류와 식별자를 결합한다.
- **TTL**: `EX 300`으로 5분. 외부 API 응답이 자주 바뀌지 않는다는 전제에서 잡은 값.
- **무효화**: 데이터 변경 시점에는 `redis.del(key)`로 지워서, 다음 조회의 자연스러운 미스 흐름으로 새 값이 채워지게 한다(덮어쓰기 대신 삭제를 택한 이유는 race condition 회피).
- **스탬피드 대비**: 인기 키 만료 직후의 동시 미스 폭주를 막기 위한 락 등의 대응이 추가 보호로 필요하다는 점을 인식하고 있다.

## 참조 매핑

| 사용한 지식 | 참조 이유 |
| --- | --- |
| [redis-caching.md](./redis-caching.md) | 캐시-우선 조회 흐름, `SET ... EX`로 TTL을 다는 방식, `product:<id>` 형태의 키 네이밍, `del`로 무효화하는 전략과 그 안전성 근거, 인기 키 만료 시 스탬피드 위험과 대응 방향까지 — 이 구현 전체가 한 지식 문서의 적용이다. |
