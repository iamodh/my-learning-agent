# 실시간 알림 시스템 학습 정리

서버 → 클라이언트 실시간 알림 시스템을 만들기 위해 통신 방식과 라이브러리를
학습하고 결정한 내용을 정리한 문서 모음.

## 인과 구조

학습한 지식(`knowledge/`)을 바탕으로 기술 선택 결정(`result/`)에 도달했다.
선택 경로: SSE 검토 → 양방향 필요로 WebSocket 결정 → 운영 편의로 socket.io 결정.

## 폴더 구조

```
.
├── README.md
├── knowledge/
│   └── realtime-transport/
│       ├── websocket.md          # WebSocket 프로토콜 (양방향 실시간)
│       ├── sse.md                # SSE (단방향 HTTP 스트림)
│       ├── websocket-vs-sse.md   # 둘 중 무엇을 쓸지 선택 기준
│       └── ws-vs-socketio.md     # WebSocket 라이브러리 선택
└── result/
    └── realtime-notification-system.md  # 만든 것 + 결정 경로 + 사용 지식
```

## 읽는 순서

1. `knowledge/realtime-transport/websocket.md`, `sse.md` — 두 후보 기술 이해
2. `knowledge/realtime-transport/websocket-vs-sse.md` — 선택 기준
3. `knowledge/realtime-transport/ws-vs-socketio.md` — 라이브러리 비교
4. `result/realtime-notification-system.md` — 위 지식이 어떻게 결정에 쓰였는지
