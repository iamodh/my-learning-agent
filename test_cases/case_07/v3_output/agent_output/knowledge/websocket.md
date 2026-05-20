# WebSocket

WebSocket은 처음에 HTTP로 핸드셰이크를 한 뒤, 기존 TCP 연결을 유지하면서 양쪽이 자유롭게 메시지를 주고받는 통신 방식이다. 일반 HTTP 요청-응답과 달리 연결이 계속 살아 있으므로 서버가 원할 때 클라이언트로 즉시 데이터를 보낼 수 있다.

## 일반 HTTP와의 차이

일반 HTTP는 클라이언트가 요청하고 서버가 응답하면 연결이 끝나는 구조다. 이 구조에서는 서버가 먼저 클라이언트에게 말을 걸 수 없다.

WebSocket은 `Upgrade` 헤더를 사용해 HTTP 연결을 WebSocket 프로토콜로 전환하고, 이후에는 같은 연결을 유지한다. 그래서 서버 알림 푸시처럼 실시간성이 필요한 상황에 적합하다.

## Express에서 `ws`를 붙이는 예시

대화에서는 Express 서버에 `ws`의 `WebSocketServer`를 같은 HTTP 서버로 붙이는 예시를 다뤘다.

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

`connection` 핸들러 안에서 서버가 `send`로 알림을 보내고, 클라이언트에서 온 메시지는 `message` 이벤트로 받을 수 있다.
