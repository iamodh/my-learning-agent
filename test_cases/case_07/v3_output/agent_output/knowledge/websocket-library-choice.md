# WebSocket 라이브러리 선택

Node 환경에서 WebSocket을 사용할 때 대화에서는 `ws`와 `socket.io`를 비교했다. 최종적으로는 알림 시스템에 필요한 운영 편의 기능 때문에 `socket.io` 방향으로 정했다.

## `ws`

`ws`는 WebSocket 프로토콜을 얇게 구현한 라이브러리다. 가볍고 표준 WebSocket에 충실하다.

## `socket.io`

`socket.io`는 WebSocket 위에 여러 기능을 얹은 프레임워크에 가깝다. 대화에서 언급된 기능은 다음과 같다.

- 룸
- 네임스페이스
- 자동 재연결
- 이벤트 기반 API
- WebSocket이 안 될 때 HTTP long-polling 폴백

단, `socket.io`는 자체 프로토콜을 쓰므로 클라이언트도 `socket.io-client`를 사용해야 한다.

## 최종 선택

알림 시스템에서는 룸 단위 브로드캐스트, 재연결 처리, 폴백 기능이 유용하다고 판단했다. 그래서 가볍게 표준 WebSocket만 쓰는 `ws`보다 `socket.io`를 쓰는 방향으로 정했다.
