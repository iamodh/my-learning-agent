# 실시간 알림 시스템 (기술 선택 결정)

## 만들려고 한 것
서버에서 클라이언트로 실시간 알림을 푸시하고, 클라이언트에서 '읽음' 표시를 서버로
다시 보내는 알림 시스템. 추후 채팅/입력 상태 같은 양방향 기능으로 확장 가능성 있음.

## 최종 결정
- **통신 방식: WebSocket**
- **라이브러리: socket.io**

## 결정 경로와 사용된 지식

1. **실시간 푸시 후보 검토** → [SSE](../knowledge/realtime-transport/sse.md), [WebSocket](../knowledge/realtime-transport/websocket.md)
   - SSE가 HTTP 친화적이고 구현이 가벼워 단방향 알림에는 매력적이었음.

2. **SSE 탈락 → WebSocket 선택** → [WebSocket vs SSE 선택 기준](../knowledge/realtime-transport/websocket-vs-sse.md)
   - 이유: 알림은 받기만 하는 게 아니라 클라이언트 → 서버 '읽음' 액션이 필요하다.
     SSE는 단방향이라 '읽음'을 위해 별도 HTTP 요청을 섞는 하이브리드 구조가 되고,
     향후 채팅/입력 상태 확장까지 고려하면 처음부터 양방향 채널 하나로 통일하는 게
     복잡도가 낮다. 그래서 양방향인 WebSocket을 선택.

3. **라이브러리: socket.io 선택** → [ws vs socket.io](../knowledge/realtime-transport/ws-vs-socketio.md)
   - 이유: 알림 시스템은 룸 단위 브로드캐스트와 재연결 처리가 자주 필요하다.
     socket.io는 룸/네임스페이스, 자동 재연결, WebSocket 불가 시 long-polling
     폴백을 내장해 운영 편의가 알림 시나리오와 잘 맞는다. ws는 가볍지만 이 기능을
     직접 구현해야 해서 제외.

## 요약
SSE는 단방향·HTTP 친화적이라 매력적이지만 '읽음' 같은 클라이언트 → 서버 액션 때문에
양방향인 WebSocket으로 결정. 라이브러리는 룸/폴백/재연결 기능 때문에 socket.io 채택.
