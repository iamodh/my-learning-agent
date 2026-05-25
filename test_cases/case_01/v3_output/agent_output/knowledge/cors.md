# CORS (Cross-Origin Resource Sharing)

CORS는 브라우저가 다른 출처(origin)에서 받아온 스크립트가 임의로 다른 출처의 API를 호출하지 못하게 막는 보안 정책이다. 에러를 일으키는 주체는 서버가 아니라 브라우저다.

## 동작 방식

브라우저는 응답에 `Access-Control-Allow-Origin` 헤더가 붙어 있는지 검사한다. 그 헤더가 없거나 내 출처와 맞지 않으면, 응답 자체는 도착했더라도 자바스크립트가 응답 본문을 읽지 못하도록 차단한다.

따라서 서버 쪽에서 해야 할 일은 응답에 적절한 허용 헤더들을 붙여 주는 것이다.

## 프리플라이트(preflight)

단순하지 않은 요청(예: JSON content-type, 커스텀 헤더가 들어간 요청)에서는 브라우저가 본 요청을 보내기 전에 OPTIONS 메서드로 "이 요청을 서버가 받아 줄 것인지"를 먼저 물어본다. 이 절차를 프리플라이트라고 한다.

미리 물어보는 이유는, 서버가 거절할 요청이라면 실제 DELETE나 POST가 서버에 도달해 부작용을 일으키기 전에 차단하기 위함이다.

프리플라이트 응답에는 `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` 같은 헤더가 포함된다.
