# WebSocket

WebSocket은 클라이언트와 서버가 하나의 TCP 연결을 지속적으로 유지하면서 양방향으로 메시지를 주고받을 수 있게 해주는 프로토콜이다. 연결이 계속 살아 있기 때문에 서버가 원할 때 즉시 클라이언트로 데이터를 보낼 수 있고, 이 특성이 실시간 통신을 가능하게 한다.

## HTTP와의 차이

HTTP는 요청-응답이 끝나면 연결이 끊기는 구조라 서버가 먼저 클라이언트에 말을 걸 수 없다. WebSocket은 처음에 HTTP로 핸드셰이크(Upgrade 헤더)를 한 뒤, 그 TCP 연결을 유지한 채 별도 프로토콜(`ws://`)로 전환된다. 이후에는 양쪽이 자유롭게 메시지를 주고받을 수 있다.

## Express + ws 설정 예시

`ws` 라이브러리로 Express 서버에 WebSocket을 붙이는 가장 단순한 형태는 다음과 같다. 같은 http 서버에 `WebSocketServer`를 붙이고, `connection` 핸들러 안에서 송수신을 처리한다.

```js
const express = require('express');
const http = require('http');
const { WebSocketServer } = require('ws');

const app = express();
const server = http.createServer(app);
const wss = new WebSocketServer({ server });

wss.on('connection', (ws) => {
  console.log('client connected');

  ws.on('message', (data) => {
    // 클라이언트 → 서버 (예: '읽음' 이벤트)
    console.log('received:', data.toString());
  });

  // 서버 → 클라이언트 (예: 알림 푸시)
  ws.send(JSON.stringify({ type: 'notification', text: '새 알림' }));
});

server.listen(3000);
```

## ws vs socket.io

- `ws`: WebSocket 프로토콜만 얇게 구현한 라이브러리. 가볍고 표준에 충실하다.
- `socket.io`: 그 위에 룸(room), 네임스페이스, 자동 재연결, 이벤트 기반 API, 그리고 WebSocket이 안 될 때 HTTP long-polling으로 자동 폴백하는 기능까지 얹은 프레임워크. 단, 자체 프로토콜이라 클라이언트도 `socket.io-client`를 써야 한다.

룸 단위 브로드캐스트나 재연결 처리가 자주 필요한 시나리오(예: 알림 시스템)에는 `socket.io`가 편하고, 표준 그대로 가볍게 가고 싶으면 `ws`가 적합하다.
