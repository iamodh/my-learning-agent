# 사이드 프로젝트 로그인 기능

## 무엇을 만들었는가

이메일/비번을 직접 관리하지 않고, 구글 OAuth로 사용자를 식별한 뒤 서버 세션으로 로그인 상태를 유지하는 로그인 기능. Express 기반.

## 구성

1. `/auth/google` — 사용자를 구글 인증 페이지로 리다이렉트.
2. `/auth/google/callback` — 인가 코드를 access token으로 교환하고 사용자 프로필을 받아 DB에 upsert.
3. 콜백 처리 마지막에 세션 ID를 발급하고 HttpOnly 쿠키로 내려준다. 이후 요청은 쿠키의 세션 ID로 사용자를 식별한다.
4. 세션 만료는 sliding 30분 + absolute 14일을 기준으로 둔다.

## 참조 매핑

- `knowledge/oauth-google.md`
- `knowledge/session-management.md`

## 참조 이유

- **`oauth-google.md`**: 비밀번호를 직접 받지 않으려는 요구사항이 출발점이었다. OAuth가 비밀번호를 우리 서버에 흘리지 않고 구글에 식별을 위임하는 메커니즘이라 채택했고, 콜백에서 인가 코드를 access token으로 교환해 프로필을 받는 흐름을 그대로 구현했다.
- **`session-management.md`**: OAuth로 사용자 식별을 마친 뒤에도 매 요청마다 구글에 다시 묻을 수는 없기 때문에, 서버가 로그인 상태를 직접 유지하는 메커니즘이 필요했다. 만료 컨트롤이 직관적이라는 점에서 JWT 대신 세션을 선택했고, sliding + absolute 혼합 만료 정책으로 UX와 보안을 절충했다.
