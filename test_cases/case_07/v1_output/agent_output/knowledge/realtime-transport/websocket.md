# WebSocket

## 핵심
- 서버가 클라이언트로 먼저 데이터를 보낼 수 있는 **양방향** 실시간 통신 프로토콜.
- 처음 HTTP 핸드셰이크(`Upgrade` 헤더) 후, 같은 TCP 연결을 유지한 채 별도 프로토콜(`ws://`)로 전환된다.
- 연결이 계속 살아 있으므로 서버가 원할 때 즉시 클라이언트로 푸시 가능 → "실시간".

## HTTP와의 차이
- HTTP: 요청-응답이 끝나면 연결 종료. 서버가 먼저 말을 걸 수 없음.
- WebSocket: 핸드셰이크 후 연결 유지. 양쪽이 자유롭게 메시지 송수신.

## 특징
- 양방향이므로 "알림 수신 + 읽음 처리 + 채팅/입력 상태" 같은 기능을 한 연결로 통일 가능.
- 메시지 순서, 세션 관리를 단일 연결 안에서 일관되게 처리.

## 최소 예시 (Express + ws)
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
Express 서버에 `WebSocketServer`를 같은 `http` 서버로 붙이고, `connection` 핸들러 안에서 송수신을 처리한다.
