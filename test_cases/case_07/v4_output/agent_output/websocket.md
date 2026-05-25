# WebSocket

WebSocket은 클라이언트와 서버가 하나의 TCP 연결을 지속적으로 유지하면서 양방향으로 메시지를 주고받는 프로토콜이다. 서버가 원하는 시점에 클라이언트로 데이터를 즉시 보낼 수 있어 실시간 통신이 가능하다.

## HTTP와의 차이

HTTP는 요청-응답이 끝나면 연결이 끊기는 구조라 서버가 먼저 말을 걸 수 없다. WebSocket은 처음에 HTTP로 핸드셰이크(Upgrade 헤더)를 한 뒤, 같은 TCP 연결을 유지하면서 별도 프로토콜(`ws://`)로 전환된다. 연결이 계속 살아 있기 때문에 서버가 원할 때 즉시 클라이언트로 데이터를 보낼 수 있고, 이것이 "실시간"이 가능한 이유다.

## Express에서의 사용 (ws 라이브러리)

Express 앱과 같은 http 서버에 WebSocketServer를 붙이는 패턴이다. `connection` 핸들러 안에서 메시지 수신(`ws.on('message', ...)`)과 메시지 전송(`ws.send(...)`)을 함께 다룬다.

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

## 라이브러리 선택: ws vs socket.io

- **ws**: WebSocket 프로토콜만 얇게 구현한 라이브러리. 가볍고 표준에 충실하다.
- **socket.io**: ws 위에 룸(room), 네임스페이스, 자동 재연결, 이벤트 기반 API, WebSocket이 안 될 때 HTTP long-polling으로 자동 폴백하는 기능까지 얹은 프레임워크에 가깝다. 단, 자체 프로토콜이라 클라이언트도 `socket.io-client`를 써야 한다.

룸 단위 브로드캐스트나 재연결 처리가 자주 필요하면 socket.io가 편하고, 표준 그대로 가볍게 쓰고 싶으면 ws가 좋다.
