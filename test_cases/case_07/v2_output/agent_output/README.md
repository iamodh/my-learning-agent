# 실시간 알림 시스템 학습 노트

Express 기반 실시간 알림 시스템을 만들기 위해 학습한 지식과 그 결과 결정을 정리한 문서 모음이다.

## 문서 배치

```
.
├── README.md
├── realtime-notification-system.md   # 결과 문서: 만들고자 한 시스템과 참조 매핑
└── knowledge/
    ├── websocket.md   # 채택된 통신 방식 (HTTP 차이, Express+ws 예시, ws vs socket.io 포함)
    └── sse.md         # 검토 후 탈락한 대안
```

## 인과 관계

- `realtime-notification-system.md`가 만들고자 하는 산출물이다.
- 이 산출물은 `knowledge/websocket.md`의 지식을 참조한다.
- `knowledge/sse.md`는 같은 결정 과정에서 검토되었으나 탈락한 대안이라 결과 문서의 참조 매핑에는 포함되지 않는다.
