# Express 회원가입/로그인 기능

## 만든 것

Express 서버에 회원가입과 로그인 엔드포인트를 구성한다. 가입 시점에 비밀번호를 안전하게 보관하고, 로그인 이후엔 stateless 토큰으로 사용자 상태를 증명하는 구조다.

- `POST /signup` — 이메일/비밀번호를 받아 해시한 뒤 사용자 레코드 생성.
- `POST /login` — 입력 비밀번호를 저장된 해시와 비교하고, 성공 시 토큰 발급.
- `authenticate` 미들웨어 — 이후 요청의 `Authorization` 헤더에서 토큰을 꺼내 검증하고 `req.user`에 사용자 정보를 채워준다.

## 인증 흐름

1. 클라이언트가 이메일/비밀번호 전송.
2. 서버가 이메일로 DB에서 사용자 조회.
3. `bcrypt.compare`로 입력 비밀번호와 저장된 해시 비교.
4. 일치하면 `jwt.sign`으로 토큰 발급해 응답.
5. 클라이언트는 이후 요청마다 `Authorization` 헤더에 토큰을 실어 보냄.
6. 서버는 `verify` 미들웨어로 검증하고 `req.user`에 사용자 정보를 꽂아준다.

가입 시점엔 `bcrypt.hash`, 로그인 시점엔 `bcrypt.compare + jwt.sign`, 그 이후 요청들엔 `jwt.verify`가 핵심이다.

## 사용한 지식과 참조 이유

- [bcrypt](../knowledge/bcrypt.md) — 가입 시점에 비밀번호를 단방향 해시로 저장하고, 로그인 시 `compare`로 검증하기 위해 사용한다. DB 유출 시 평문 비밀번호 노출을 막는 것이 목적이며, `saltRounds`로 해싱 강도를 조율한다.
- [JWT](../knowledge/jwt.md) — 로그인 이후 사용자 상태를 stateless하게 유지하기 위해 사용한다. 서버가 토큰 저장소를 운영할 필요가 없어 확장에 유리하며, 만료 시간으로 토큰 탈취 피해를 제한한다.
- [JWT 클라이언트 저장 위치 선택](../knowledge/jwt-storage.md) — 발급한 토큰을 클라이언트 어디에 둘지를 결정하기 위해 사용한다. XSS/CSRF 트레이드오프를 고려해 `httpOnly + Secure + SameSite` 쿠키 방식을 택한다.

## 비고

로그인 상태 유지 방식은 초기에 세션 방식도 검토했으나 stateless 확장성을 이유로 JWT로 결정했다. 리프레시 토큰 분리는 필요해지면 액세스 토큰 구조 위에 얹는다.
