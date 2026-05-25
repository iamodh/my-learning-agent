# 산출물 구조

대화에서 학습한 내용을 지식 문서와 결과 문서로 정리했다.

```
.
├── README.md
├── cors.md                          # 지식 문서: CORS 개념 + cors 미들웨어 + 프리플라이트
└── express-react-cors-setup.md      # 결과 문서: 리액트 → 익스프레스 API CORS 설정
```

## 문서 안내

- `cors.md` — CORS가 무엇이고 왜 에러가 발생하는지, 익스프레스에서 `cors` 미들웨어로 어떻게 처리하는지, 프리플라이트(OPTIONS) 요청은 어떻게 자동 처리되는지를 한 문서에서 다룬다.
- `express-react-cors-setup.md` — 리액트 앱에서 익스프레스 API를 호출할 때 발생하던 CORS 에러를 해결하기 위해 익스프레스에 `cors` 미들웨어를 적용한 결과를 정리한다.
