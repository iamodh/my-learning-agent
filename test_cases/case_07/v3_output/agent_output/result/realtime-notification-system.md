# 실시간 알림 시스템

Express 서버에서 클라이언트로 실시간 알림을 푸시하고, 클라이언트의 '읽음' 표시 같은 액션도 같은 채널로 받는 양방향 알림 시스템을 만든다.

## 무엇을 만들었는가

- Express HTTP 서버에 WebSocket 서버를 같은 HTTP 서버 인스턴스에 붙인 구조.
- 서버 → 클라이언트: 알림 메시지 푸시 (`ws.send`).
- 클라이언트 → 서버: '읽음' 같은 액션 수신 (`ws.on('message')`).
- 라이브러리는 룸, 자동 재연결, long-polling 폴백을 위해 socket.io 방향으로 진행.

## 기본 구현 형태 (ws 라이브러리 기준)

대화에서 받은 가장 간단한 형태의 예시:

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

Express 서버에 `WebSocketServer`를 같은 http 서버로 붙이는 패턴이고, `connection` 핸들러 안에서 송수신을 처리한다.

## 참조한 지식

| 지식 | 참조 이유 |
|---|---|
| [WebSocket](../knowledge/websocket.md) | 알림 수신 + '읽음' 처리처럼 양방향 통신이 필요한 시나리오에서, 서버가 먼저 메시지를 보낼 수 있고 클라이언트도 같은 연결로 응답할 수 있는 프로토콜이 필요했다. |
| [ws vs socket.io](../knowledge/ws-vs-socketio.md) | 룸 단위 브로드캐스트, 자동 재연결, WebSocket 불가 환경의 polling 폴백이 알림 시스템 운영에 유리해 socket.io를 선택했다. |

초기에는 SSE를 검토했으나, 클라이언트 → 서버 '읽음' 액션 때문에 양방향이 가능한 WebSocket으로 최종 결정했다.
