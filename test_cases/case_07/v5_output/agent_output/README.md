# 실시간 알림 학습 정리

Express 서버에서 클라이언트로 실시간 알림을 보내는 방법을 학습하고, 양방향 통신이 필요하다고 판단하여 WebSocket(socket.io) 기반으로 구현 방향을 잡았다.

## 문서 배치

```
.
├── README.md
├── knowledge/
│   ├── websocket.md       # WebSocket 개념 + ws/socket.io 비교 + 코드 예시
│   └── sse.md             # SSE — 검토 후 탈락한 대안
└── result/
    └── realtime-notification-server.md
```

## 흐름

- `knowledge/websocket.md` — 결과 문서가 직접 참조하는 핵심 지식.
- `knowledge/sse.md` — 검토 단계에서 비교 대상이었으나 양방향 요구로 탈락. 결과 문서의 참조 매핑에는 포함되지 않는다.
- `result/realtime-notification-server.md` — 위 지식을 바탕으로 구성한 Express + socket.io 알림 서버.
