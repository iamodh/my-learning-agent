# CORS (Cross-Origin Resource Sharing)

## 핵심 요약

- CORS 에러를 내는 주체는 **서버가 아니라 브라우저**다.
- 브라우저는 다른 출처(origin)에서 받은 스크립트가 임의의 출처 API를 호출하지 못하도록, 응답에 `Access-Control-Allow-Origin` 헤더가 있는지 검사한다.
- 헤더가 없거나 내 출처와 맞지 않으면 응답이 도착했더라도 자바스크립트가 그 응답을 **읽지 못하게** 막는다.
- 해결책은 서버 응답에 적절한 허용 헤더를 붙이는 것이다.

## 왜 에러가 나는가

요청-응답 자체는 정상적으로 오간다. 문제는 보안이다.

> 브라우저는 "A 출처에서 받아온 스크립트가 마음대로 B 출처 API를 호출하면 위험하다"고 본다.

그래서 응답이 돌아오면 응답 헤더에 `Access-Control-Allow-Origin`이 있는지, 그 값이 내 출처와 일치하는지 확인한다. 조건이 안 맞으면 응답 본문을 스크립트에게 전달하지 않고 에러로 처리한다.

## 핵심 응답 헤더

| 헤더 | 역할 |
|------|------|
| `Access-Control-Allow-Origin` | 응답을 읽을 수 있는 출처를 지정 |
| `Access-Control-Allow-Methods` | 허용하는 HTTP 메서드 목록 |
| `Access-Control-Allow-Headers` | 허용하는 요청 헤더 목록 |
| `Access-Control-Allow-Credentials` | 쿠키/인증 정보 동반 요청 허용 여부 |

## 프리플라이트(Preflight)

단순하지 않은 요청(예: `Content-Type: application/json`, 커스텀 헤더, `PUT`/`DELETE` 등)에서는 브라우저가 **본 요청 전에** `OPTIONS` 메서드로 "이 요청 보내도 받아줄 거냐"를 먼저 묻는다.

- 이유: 서버가 거절할 요청이라면, 실제 `POST`/`DELETE`가 서버에 도달해 **부작용을 일으키기 전에** 차단하기 위함이다.
- 브라우저는 `OPTIONS` 응답의 `Access-Control-Allow-Methods` / `Allow-Headers`를 보고 본 요청을 보낼지 결정한다.

## 자격 증명(Credentials)

쿠키나 인증 정보를 함께 보내려면 양쪽 모두 설정이 필요하다.

- 서버: `Access-Control-Allow-Credentials: true`
- 프론트: `fetch`에 `credentials: 'include'`

이때 `Access-Control-Allow-Origin`은 `*`가 아닌 **구체적인 출처**여야 한다.
