# React 앱용 Express API CORS 설정

React 앱에서 Express로 만든 API를 호출할 때 발생하는 CORS 에러를 해결하기 위해, Express 서버에 `cors` 미들웨어를 적용한다.

## 참조 지식

- `express-cors.md`: 브라우저가 CORS 허용 헤더를 검사하는 방식과 Express에서 `cors` 미들웨어로 이를 처리하는 방법을 설명한다.

## 적용 이유

React 앱과 Express API가 서로 다른 출처에 있을 때 브라우저가 응답 접근을 막을 수 있으므로, Express 응답에 허용 출처와 메서드, 인증 정보 관련 CORS 헤더를 붙여야 한다.

`POST` 요청 등에서 `OPTIONS` 프리플라이트가 먼저 발생할 수 있으므로, 이를 자동 처리하는 `cors` 미들웨어를 사용한다.
