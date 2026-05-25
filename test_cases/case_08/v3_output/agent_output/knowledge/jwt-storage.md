# JWT 클라이언트 저장 위치 선택

발급된 JWT를 클라이언트 측 어디에 보관할지는 별도의 보안 결정이다. 두 후보가 있고 각각 다른 공격 표면을 갖는다.

## 로컬스토리지

- 다루기 쉽다.
- JavaScript로 접근 가능해서 **XSS 공격**에 토큰이 그대로 털릴 수 있다.

## 쿠키 (httpOnly)

- `httpOnly` 플래그를 주면 JavaScript에서 접근할 수 없어 XSS에는 안전하다.
- 대신 브라우저가 쿠키를 자동으로 첨부하기 때문에 **CSRF 공격**에 노출된다.

## 권장 구성

`httpOnly + Secure + SameSite=Strict`(또는 `Lax`) 쿠키에 저장한다. 이 조합으로 XSS를 막고, SameSite로 CSRF도 상당 부분 차단된다. 중요한 작업에는 CSRF 토큰을 별도로 두면 더 단단해진다.
