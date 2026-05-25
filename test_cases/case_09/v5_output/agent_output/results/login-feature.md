# 로그인 기능

사이드 프로젝트에 구글 OAuth 기반 로그인을 붙인다. 이메일/비밀번호를 직접 관리하지 않고 외부 제공자에게 사용자 식별을 위임한 뒤, 서버 세션으로 로그인 상태를 유지한다.

## 구현 구성

- `GET /auth/google` — 구글 인증 페이지로 리다이렉트하는 진입점.
- `GET /auth/google/callback` — 인가 코드를 받아 access token 교환 → 프로필 조회 → 세션 발급.
- 세션은 HttpOnly 쿠키로 브라우저에 심고, 서버는 세션 저장소에 `{ sessionId → userId }`를 들고 있는다.
- 만료 정책은 sliding 30분 + absolute 14일을 기본으로 두고, 결제 같은 민감 액션 직전에는 더 짧은 재인증을 추가로 요구한다.

## 참조 매핑

| 사용한 지식 | 참조 이유 |
|---|---|
| `knowledge/oauth-login.md` | 콜백에서 인가 코드 → access token → 프로필을 가져오는 3단계 흐름과 Express 구현을 그대로 따른다. |
| `knowledge/session-management.md` | 콜백 끝에서 세션을 발급해 쿠키로 심고, 이후 요청에서 세션 ID로 사용자를 식별하기 위해 만료 전략까지 같이 적용한다. |
