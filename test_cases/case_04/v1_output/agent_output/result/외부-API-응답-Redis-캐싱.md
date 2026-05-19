# 결과 문서: 외부 API 응답 Redis 캐싱

## 만들려고 했던 것
Express 서비스에서 **응답이 거의 바뀌지 않는 외부 API 호출을 Redis로 캐싱**하려고 했다. 매 요청마다 외부 API를 호출해 비용이 늘고 응답이 느린 문제를 해결하는 것이 목표다.

## 구현 형태
라우트 핸들러 안에서 Cache-Aside 패턴으로 처리한다.

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

로컬에 Redis를 띄우고 Express에 `ioredis`(또는 `node-redis`) 클라이언트를 붙여 시작한다.

## 사용된 지식과 사용 이유

| 사용된 지식 | 왜 사용되었는가 |
|---|---|
| [Redis 동작 원리](../knowledge/redis-basics/redis-동작-원리.md) | 메모리 기반 키-값 저장소라서 외부 API보다 빠르고, 별도 프로세스라 여러 인스턴스가 캐시를 공유할 수 있어 캐싱 대상으로 적합하기 때문 |
| [TTL과 캐시 만료](../knowledge/cache-operations/ttl-만료.md) | 캐시를 영원히 두면 오래된 데이터가 반환되므로, 데이터 변경 주기에 맞춰 자동 만료(`EX 300`)를 설정하기 위해 |
| [캐시 키 네이밍 컨벤션](../knowledge/cache-operations/키-네이밍-컨벤션.md) | `product:${id}` 같이 콜론 계층 키를 써서 키 충돌을 막고 가독성·일괄 관리를 확보하기 위해 |
| [캐시 무효화](../knowledge/cache-operations/캐시-무효화.md) | 데이터가 바뀌는 시점에 TTL만 기다리지 않고 `del`로 즉시 무효화해 최신 데이터를 보장하기 위해 (race condition 회피) |
| [캐시 스탬피드](../knowledge/cache-operations/캐시-스탬피드.md) | 인기 키 만료 순간 외부 API 폭주를 막기 위해 락/조기 갱신/stale-while-revalidate 전략을 적용하기 위해 |

## 운영 시 정리
- Redis는 메모리에 키-값으로 저장해서 빠르다.
- 만료는 TTL로 자동 처리하되, 데이터가 바뀌는 시점엔 명시적으로 `del`로 무효화한다.
- 키는 `리소스:id:속성` 식 콜론 계층으로 잡아 충돌과 가독성을 동시에 해결한다.
- 인기 키 만료 시 캐시 스탬피드를 막기 위한 락 등의 보호 장치를 둔다.
