# Redis 기반 캐싱

Redis 캐싱은 자주 바뀌지 않는 데이터를 Redis에 키-값 형태로 저장해두고, 같은 요청이 다시 들어왔을 때 외부 API를 매번 호출하지 않고 Redis에서 먼저 조회하는 방식이다. Redis는 데이터를 메모리(RAM)에 보관하므로 외부 API 호출이나 디스크 조회보다 빠르며, 별도 프로세스로 동작해 여러 서버 인스턴스가 같은 캐시를 공유할 수 있다.

## 기본 흐름

1. 요청이 들어오면 Redis에서 캐시 키를 조회한다.
2. 값이 있으면 캐시 히트로 보고 저장된 값을 반환한다.
3. 값이 없으면 캐시 미스로 보고 외부 API를 호출한다.
4. 외부 API 응답을 Redis에 저장한 뒤 반환한다.

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

`EX 300`은 TTL을 300초로 설정한다는 뜻이다.

## TTL과 만료

TTL은 캐시가 유지되는 시간이다. 캐시가 영원히 남아 있으면 실제 데이터 변경을 반영하지 못할 수 있으므로, 데이터 변경 빈도에 맞춰 만료 시간을 둔다.

대화에서는 거의 바뀌지 않는 메타데이터는 1시간, 자주 바뀌는 데이터는 30초에서 5분 정도가 예로 언급됐다.

Redis는 키마다 만료 시간을 기록한다. 만료 처리는 두 방식으로 이뤄진다.

- 접근 시점에 만료 여부를 확인해 삭제하는 lazy 방식
- 백그라운드에서 주기적으로 일부 키를 확인해 만료된 키를 정리하는 active expiration 방식

TTL이 지난 키가 잠깐 메모리에 남아 있을 수는 있지만, 조회 시에는 없는 값처럼 처리된다.

## 키 네이밍

캐시 키는 콜론(`:`)으로 계층을 구분하는 방식이 언급됐다.

예시는 다음과 같다.

- `product:123`
- `user:42:profile`
- `search:keyword:laptop:page:2`
- `prod:product:123`

키에는 리소스 종류, 식별자, 파라미터나 속성 같은 변형 정보를 순서대로 넣는다. 이렇게 하면 충돌을 줄이고, 다른 개발자가 의미를 파악하기 쉬우며, 패턴 기반 일괄 처리에도 유리하다.

## 데이터 변경 시 캐시 무효화

데이터가 바뀌면 TTL 만료만 기다리지 않고 캐시를 명시적으로 무효화할 수 있다. 대화에서는 DB 업데이트 직후 `redis.del(key)`로 해당 키를 삭제하는 방식이 더 안전한 선택으로 정리됐다.

캐시를 새 값으로 덮어쓰는 방법도 있지만, 동시에 다른 요청이 들어오면 오래된 값이 다시 쓰이는 race condition이 생길 수 있다. 삭제 방식은 다음 조회 때 캐시 미스가 발생하고, 그 시점에 최신 데이터를 다시 받아와 캐시를 채우게 한다.

## 캐시 스탬피드

인기 키가 만료된 순간 많은 요청이 동시에 들어오면 모두 캐시 미스가 되어 외부 API 호출이 폭주할 수 있다. 이 문제를 캐시 스탬피드 또는 thundering herd라고 한다.

대화에서 언급된 대응 방식은 다음 세 가지다.

- Redis의 `SETNX` 같은 방식으로 짧은 락을 걸어 한 요청만 외부 API를 호출하게 한다.
- TTL이 끝나기 전에 백그라운드에서 미리 갱신한다.
- 만료 직후 잠깐 오래된 값을 사용하면서 뒤에서 새 값을 채운다.

트래픽이 큰 키에는 락 방식이 가장 확실한 대응으로 언급됐다.
