# Redis 캐싱

Redis는 데이터를 메모리(RAM)에 키-값 형태로 보관하는 별도 프로세스로, 디스크 조회나 외부 API 호출보다 훨씬 빠른 응답(보통 마이크로초 단위)을 낸다. 단일 스레드로 명령을 순차 처리해 동시성 모델이 단순하고, 메모리 변수와 달리 별도 프로세스라서 여러 서버 인스턴스가 같은 캐시를 공유할 수 있다. 이런 성질 덕분에 자주 안 바뀌는 외부 API 응답처럼 반복 조회되는 데이터를 캐시하는 용도로 쓰인다.

기본 흐름은 "캐시 먼저 조회 → 있으면 반환(히트), 없으면 원본 호출 후 캐시에 저장하고 반환(미스)"이다. Express에서 ioredis 같은 클라이언트로 붙여 라우트 핸들러 안에서 직접 처리하거나 미들웨어로 분리한다.

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

## TTL (만료 시간)

`SET` 명령의 `EX` 옵션으로 키마다 TTL을 초 단위로 지정한다. 위 예시의 `EX 300`은 5분 뒤 자동 삭제된다는 의미다.

TTL은 데이터가 얼마나 자주 바뀌는지에 맞춰 정한다. 거의 안 바뀌는 메타데이터는 1시간, 자주 바뀌는 데이터는 30초~5분 정도가 흔하다.

내부적으로 Redis는 키마다 만료 시간을 기록해두고 두 방식으로 정리한다.

- **lazy expiration**: 클라이언트가 그 키에 접근할 때 만료됐는지 체크해서 삭제한다.
- **active expiration**: 백그라운드에서 주기적으로 랜덤 샘플링해 만료된 키들을 청소한다.

이 때문에 TTL이 지난 키가 잠깐 메모리에 남아있을 수는 있지만, 조회 시점에는 항상 없는 것으로 처리된다.

## 키 네이밍 컨벤션

콜론(`:`)으로 계층을 구분해 `리소스:식별자:속성/변형` 순으로 짓는다. 예:

- `product:123`
- `user:42:profile`
- `search:keyword:laptop:page:2`

이 컨벤션의 이점은 두 가지다. 첫째, `KEYS product:*`나 `SCAN`으로 패턴 매칭해 일괄 처리가 가능하다. 둘째, 다른 개발자가 키만 봐도 무엇을 가리키는지 즉시 알 수 있다. 환경별로 prefix를 붙이는 것도 흔하다 (`prod:product:123`).

## 데이터 변경 시 무효화

데이터가 바뀌면 TTL 만료를 기다리지 말고 명시적으로 캐시를 무효화한다. 두 가지 전략이 있다.

- **삭제(`redis.del(key)`)**: DB 업데이트 직후에 키를 지운다. 다음 조회 때 자연스럽게 미스가 나서 새 데이터로 다시 채워진다.
- **덮어쓰기**: 업데이트하면서 캐시도 새 값으로 set한다.

보통 **삭제가 더 안전하다**. 덮어쓰는 도중 동시에 다른 요청이 옛 값을 또 쓰면 race condition으로 옛 값이 캐시에 박힐 수 있다. 삭제는 "다음에 누가 읽을 때 최신으로 다시 받아와"라는 단순한 의도라 꼬일 일이 적다.

## 캐시 스탬피드 (thundering herd)

인기 키 하나가 만료된 직후 요청 수백 개가 동시에 들어오면 전부 캐시 미스가 나서 외부 API로 폭주하는 문제를 캐시 스탬피드(또는 thundering herd)라고 한다. 대표적인 대응은 세 가지다.

- **락(lock)**: 캐시 미스 시 한 요청만 외부 API를 호출하고 나머지는 그 결과를 기다리게 한다. Redis의 `SETNX`로 짧은 락을 잡는 방식이 흔하다.
- **probabilistic early expiration**: TTL이 끝나기 직전에 백그라운드에서 미리 갱신한다.
- **stale-while-revalidate**: 만료된 직후 잠깐 옛 값을 그대로 반환하면서 뒤에서 새 값을 채운다.

트래픽이 큰 키일수록 첫 번째 락 방식이 가장 확실하다.
