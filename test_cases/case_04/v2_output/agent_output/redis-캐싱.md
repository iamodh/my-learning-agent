# Redis 캐싱

Redis 캐싱은 자주 바뀌지 않는 데이터(예: 외부 API 응답)를 메모리 기반 키-값 저장소에 보관해 두고, 동일한 요청이 오면 외부 호출 없이 빠르게 반환하는 기법이다. 기본 흐름은 "요청 → Redis 조회 → 있으면 반환(캐시 히트) / 없으면 외부 호출 후 Redis에 저장하고 반환(캐시 미스)"이다.

## Redis가 캐시로 적합한 이유

- 데이터를 디스크가 아니라 메모리(RAM)에 키-값 형태로 보관해서, 응답이 보통 마이크로초 단위로 빠르다.
- 명령을 단일 스레드로 순차 처리해서 동시성 문제가 단순하다.
- 메모리 변수와 달리 별도 프로세스라, 여러 서버 인스턴스가 같은 캐시를 공유할 수 있다.

## 기본 적용 패턴

라우트 핸들러 안에서 직접 처리하거나 미들웨어로 빼는 방식이 가장 흔하다.

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

`EX 300`은 TTL 300초(5분)라는 뜻으로, 그 시간이 지나면 키가 자동 삭제된다.

## TTL과 만료 동작

TTL은 데이터가 얼마나 자주 바뀌는지에 맞춰 정한다. 거의 안 바뀌는 메타데이터는 1시간, 자주 바뀌는 것은 30초~5분 정도가 흔하다.

Redis 내부에서는 키마다 만료 시간을 기록해두고 두 가지 방식으로 정리한다.

- **Lazy expiration**: 클라이언트가 해당 키에 접근할 때 만료 여부를 체크해서 삭제.
- **Active expiration**: 백그라운드에서 주기적으로 키들을 랜덤 샘플링해 만료된 것을 청소.

그래서 TTL이 지난 키가 잠깐 메모리에 남아있을 수 있지만, 조회 시점에는 무조건 없는 것으로 처리된다.

## 키 네이밍 컨벤션

콜론(`:`)으로 계층을 구분하는 컨벤션을 쓴다. 리소스 종류, 식별자, 변형(파라미터) 순서로 넣는다.

- `product:123`
- `user:42:profile`
- `search:keyword:laptop:page:2`

장점:

- `KEYS product:*` 또는 `SCAN`으로 패턴 매칭해서 일괄 처리할 수 있다.
- 다른 개발자가 봐도 무엇을 가리키는지 바로 알 수 있다.
- 환경별 prefix를 붙이는 것도 흔하다 (`prod:product:123`).

## 캐시 무효화 (데이터가 바뀔 때)

TTL만 믿고 기다리면 그동안 옛 데이터가 나가므로, 변경 시점에 명시적으로 무효화한다. 두 가지 전략이 있다.

1. DB 업데이트 직후 `redis.del(key)`로 해당 키를 지운다. 다음 조회 때 자연스럽게 캐시 미스가 나서 새 데이터로 다시 채워진다.
2. 업데이트하면서 캐시도 새 값으로 덮어쓴다.

보통 첫 번째(삭제)가 더 안전하다. 덮어쓰는 도중에 다른 요청이 옛 값을 또 쓰면 race condition이 생길 수 있는데, 삭제는 "다음에 읽을 때 최신으로 다시 받아와"라는 단순한 의도라 꼬일 일이 적다.

## 캐시 스탬피드(Cache Stampede)

인기 키 하나가 만료된 순간 요청 수백 개가 동시에 들어와 모두 캐시 미스가 나면, 외부 API로 폭주하는 문제다. *thundering herd*라고도 부른다.

대표 대응 세 가지:

- **락(lock)**: 캐시 미스 시 한 요청만 외부 API를 호출하게 하고 나머지는 결과를 기다리게 한다. 예: Redis `SETNX`로 짧은 락. 트래픽이 큰 키일수록 가장 확실하다.
- **Probabilistic early expiration**: TTL이 끝나기 직전에 백그라운드에서 미리 갱신.
- **Stale-while-revalidate**: 만료 직후 잠깐 옛 값을 그대로 쓰면서, 뒤에서 새 값으로 채운다.
