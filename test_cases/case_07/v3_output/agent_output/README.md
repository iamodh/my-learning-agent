# 실시간 알림 시스템 학습 문서

서버에서 클라이언트로 실시간 알림을 보내는 시스템을 만들기 위해 통신 방식과 라이브러리를 학습한 결과를 정리한다.

## 구조

```
.
├── README.md
├── knowledge/
│   ├── websocket.md
│   ├── sse.md
│   └── ws-vs-socketio.md
└── result/
    └── realtime-notification-system.md
```

- `knowledge/`: 학습한 개념 지식
  - `websocket.md`: WebSocket 프로토콜의 동작 원리
  - `sse.md`: Server-Sent Events (검토 후 탈락한 대안)
  - `ws-vs-socketio.md`: ws와 socket.io 라이브러리 비교
- `result/`: 만들기로 한 결과물
  - `realtime-notification-system.md`: Express 기반 실시간 알림 시스템
