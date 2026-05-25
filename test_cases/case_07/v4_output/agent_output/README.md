# 학습 결과물

실시간 알림 시스템을 서버 → 클라이언트로 구현하기 위해 통신 방식을 학습하고, Express + socket.io 기반 양방향 알림 채널을 만들기로 결정한 과정의 산출물.

## 배치 구조

```
agent_output/
├── README.md
├── websocket.md            # 지식: WebSocket 개념 + ws/socket.io 라이브러리 선택
├── sse.md                  # 지식: SSE (검토 후 탈락한 대안)
└── notification-system.md  # 결과: Express + socket.io 알림 시스템
```

## 문서 분류

- **지식 문서**
  - `websocket.md` — 실시간 양방향 통신 프로토콜. ws/socket.io 라이브러리 비교 및 Express 연결 코드 예시 포함.
  - `sse.md` — 서버 → 클라이언트 단방향 스트림. 본 시나리오에서는 양방향 요구로 탈락.

- **결과 문서**
  - `notification-system.md` — Express 기반 실시간 알림 시스템. socket.io로 구현 방향 결정.
