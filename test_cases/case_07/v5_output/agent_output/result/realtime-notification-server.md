# Express 기반 실시간 알림 서버

Express 서버에서 클라이언트로 알림을 실시간 푸시하고, 클라이언트의 "읽음" 같은 액션도 같은 채널로 받기 위해 WebSocket 기반 양방향 통신 서버를 구성한다. 라이브러리는 룸/폴백/자동 재연결 기능을 고려하여 socket.io로 잡았다.

## 무엇을 만드는가

- 같은 Express http 서버에 WebSocket 서버를 함께 붙인 단일 프로세스 구성.
- 서버 → 클라이언트: 새 알림 푸시.
- 클라이언트 → 서버: "읽음" 등 사용자 액션 수신.
- 추후 채팅·입력 상태 등 양방향 기능으로 확장 가능한 단일 채널.

## 결정 요약

초기에는 SSE를 검토했으나, 클라이언트 → 서버 액션("읽음")이 필요해 양방향 통신이 가능한 WebSocket으로 결정했다. 라이브러리는 룸 단위 브로드캐스트, 자동 재연결, WebSocket 미지원 환경에서의 HTTP long-polling 폴백을 이유로 socket.io 방향으로 잡았다.

## 구현 기반 — ws 패턴 (Express 결합 형태)

ws 라이브러리를 사용한 가장 단순한 결합 형태는 다음과 같다. Express의 http 서버를 만들고 거기에 WebSocket 서버를 같이 붙이는 패턴이며, socket.io를 채택하더라도 "같은 http 서버에 붙인다"는 구성은 동일하다.

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

`connection` 핸들러 안에서 `ws.send`로 서버 → 클라이언트 알림을 보내고, `ws.on('message')`로 클라이언트 → 서버의 "읽음" 같은 이벤트를 받는 구조다.

## 참조 매핑

| 참조 지식 | 참조 이유 |
|---|---|
| `knowledge/websocket.md` | 양방향 실시간 채널의 기반 프로토콜이며, ws/socket.io 선택과 Express http 서버에 결합하는 코드 패턴이 이 문서에 정리되어 있다. 라이브러리 선택(socket.io) 근거(룸·폴백·재연결)도 이 문서에서 가져왔다. |
