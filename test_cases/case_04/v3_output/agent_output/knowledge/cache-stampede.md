# 캐시 스탬피드

인기 키가 만료되는 순간 요청 수백 개가 동시에 캐시 미스를 내면, 그 부하가 그대로 원본(외부 API·DB)으로 몰린다. 이 현상을 캐시 스탬피드(cache stampede) 또는 thundering herd라고 부른다.

## 대응 전략

1. **락(lock)**: 캐시 미스 시 한 요청만 원본을 호출하고 나머지는 그 결과를 기다리게 한다. Redis에서는 `SETNX`로 짧은 락을 거는 방식이 흔하다. 트래픽이 큰 키일수록 가장 확실한 방법이다.
2. **Probabilistic early expiration**: TTL이 끝나기 직전에 백그라운드에서 미리 캐시를 갱신한다.
3. **Stale-while-revalidate**: 만료 직후 잠깐 옛날 값을 그대로 쓰면서 뒤에서 새 값을 채운다.
