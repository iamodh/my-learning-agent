# Redis 캐싱

Redis는 데이터를 디스크가 아닌 메모리(RAM)에 키-값 형태로 보관하는 별도 프로세스의 저장소이며, 외부 API 호출이나 디스크 조회보다 훨씬 빠른 응답(보통 마이크로초 단위)을 낸다. 이 특성 때문에 자주 바뀌지 않는 데이터를 일시적으로 들고 있다가 다음 요청에 빠르게 돌려주는 캐시 용도로 자주 쓰인다.

Redis는 명령을 단일 스레드로 순차 처리해서 동시성 문제가 단순한 편이고, 별도 프로세스이기 때문에 여러 서버 인스턴스가 같은 캐시를 공유할 수 있다는 점에서 단순한 인메모리 변수와 구분된다.

## 기본 흐름 (cache-aside)

요청이 들어오면 먼저 Redis에서 키로 조회한다.

- 있으면 그대로 반환한다 (캐시 히트).
- 없으면 원본 소스(예: 외부 API)를 호출한 뒤 결과를 Redis에 저장하고 반환한다 (캐시 미스).

Express 라우트 핸들러 안에서 직접 처리하거나 미들웨어로 빼는 형태가 흔하다. ioredis로 작성한 예시는 다음과 같다.

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

`EX 300`은 TTL을 300초로 지정한다는 뜻이다.

## TTL (만료 시간)

캐시가 영원히 살아있으면 안 되기 때문에 키마다 만료 시간을 둔다. TTL 값은 데이터가 얼마나 자주 바뀌는지에 맞춰서 정한다 — 거의 안 바뀌는 메타데이터는 1시간, 자주 바뀌는 건 30초~5분 정도가 흔하다.

만료 처리 방식은 두 가지가 함께 동작한다.

- **Lazy expiration**: 클라이언트가 그 키에 접근하는 시점에 만료됐는지 체크해서 삭제한다.
- **Active expiration**: 백그라운드에서 주기적으로 키를 랜덤 샘플링해서 만료된 것들을 청소한다.

그래서 TTL이 지난 키가 잠깐 메모리에 남아 있을 수는 있지만, 조회하면 무조건 없는 것으로 처리된다.

## 키 네이밍 컨벤션

키 이름을 `data1`, `userdata` 같이 평탄하게 짓지 말고, 콜론(`:`)으로 계층을 구분하는 컨벤션을 쓴다. 리소스 종류·식별자·변형(파라미터)을 순서대로 넣는다.

- `product:123`
- `user:42:profile`
- `search:keyword:laptop:page:2`

이 방식의 장점은 두 가지다. 첫째, 나중에 `KEYS product:*`나 `SCAN`으로 패턴 매칭해서 일괄 처리할 수 있다. 둘째, 다른 개발자가 봐도 무엇을 가리키는 키인지 즉시 알 수 있다. 환경별로 prefix를 붙이는 것도 흔하다 — 예: `prod:product:123`.

## 캐시 무효화

데이터가 바뀌는 시점엔 TTL이 자연 만료되기를 기다리지 말고 명시적으로 무효화한다. 두 가지 전략이 있다.

- **삭제(`redis.del(key)`)**: DB 업데이트 직후 해당 키를 지운다. 다음 조회 때 자연스럽게 캐시 미스가 나서 새 데이터로 다시 채워진다.
- **덮어쓰기**: 업데이트하면서 캐시도 새 값으로 같이 갱신한다.

보통 삭제 쪽이 더 안전하다. 덮어쓰기는 갱신하는 도중 다른 요청이 동시에 옛날 값을 다시 쓸 수 있어서 race condition을 만든다. 삭제는 "다음에 누가 읽을 때 최신으로 다시 받아와"라는 단순한 의도이기 때문에 꼬일 일이 적다.

## 캐시 스탬피드 (cache stampede / thundering herd)

인기 키 하나가 만료된 직후 요청 수백 개가 동시에 와서 모두 캐시 미스가 나면 외부 API로 트래픽이 폭주하는 문제가 생긴다. 이를 캐시 스탬피드 또는 thundering herd라고 부른다.

대표적인 대응 세 가지가 있다.

- **락(lock)**: 캐시 미스 시 한 요청만 외부 API를 호출하게 하고 나머지는 그 결과를 기다리게 한다. 예를 들어 Redis의 `SETNX`로 짧은 락을 잡는다.
- **Probabilistic early expiration**: TTL이 끝나기 직전에 백그라운드에서 미리 갱신한다.
- **Stale-while-revalidate**: 만료된 직후 잠깐 옛날 값을 그대로 쓰면서 뒤에서 새 값으로 채운다.

트래픽이 큰 키일수록 첫 번째 락 방식이 가장 확실하다.
